# Day 8 — CNAME to an Unclaimed Service: Grading the Severity Correctly

**Severity:** Low → High (depends entirely on the destination service) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

This is the close cousin of subdomain takeover, and the two get conflated often enough that it's worth separating them clearly. A CNAME pointing to an unclaimed service is the *condition* — the DNS record still points somewhere that no longer has an active account behind it. Whether that condition is a full takeover, a temporary dead-end, or a non-issue depends entirely on what the destination service allows.

## 🌱 Why It Exists

Same root cause as takeover — a decommissioned third-party account, an un-cleaned DNS record — but I'm treating it separately because the reporting mistake I see most often is testers grading every dangling CNAME as automatically High/Critical. That's not accurate, and inflating severity on findings that don't deserve it erodes trust with client engineering teams who then have to push back on your report.

## ⚔️ How To Find It

```bash
cat subs.txt | while read sub; do
  cname=$(dig CNAME "$sub" +short)
  [ -n "$cname" ] && echo "$sub -> $cname"
done > cname_map.txt

# For each destination, check the actual response
curl -s -o /dev/null -w "%{http_code}\n" "https://$cname_clean"
```

The severity grading step is the part that actually takes judgment:

- **High** — the destination service allows free, unverified registration of that exact name (Netlify, Heroku, S3, old Shopify setups). This is a full takeover.
- **Medium** — the account exists but is suspended or inactive, not immediately claimable, but worth monitoring since it could become claimable later.
- **Low** — the platform itself is defunct or requires verified domain ownership to claim, so there's no practical exploitation path, but it's still DNS hygiene worth flagging.

## 💥 How To Exploit It

Confirm the service-specific fingerprint for "this name is available" versus "this name exists but isn't active" — the two look different on every provider, and mixing them up is exactly how over-graded findings happen. Don't complete registration unless scope explicitly allows it; the fingerprint match plus a screenshot of the provider's signup page showing the name as available is sufficient proof.

## 🌍 Where This Actually Bites

In a **freight logistics** engagement, I've seen a partner-facing status page CNAME'd to a third-party uptime-monitoring service that had lapsed — Medium severity, since the monitoring platform required account verification to reclaim the subdomain, closing off the easy takeover path, but still worth fixing before that changed. Contrast that with a **banking** client's old customer-facing microsite CNAME'd to a static site host with zero verification on registration — that one graded straight to High, because anyone could claim it in under five minutes and immediately look like an official bank subdomain.

## 📋 How To Report It

Grade honestly based on the destination service's actual claim mechanism — don't default to High just because the pattern looks similar to a takeover. Include the specific evidence for whichever tier applies: for High, the registration-availability screenshot; for Medium, the account-status fingerprint and a note to re-check periodically; for Low, simply the DNS hygiene recommendation. This distinction matters more for your credibility as a senior tester than any individual finding — a report that correctly separates "exploitable today" from "worth cleaning up eventually" is what makes clients trust your severity ratings on everything else in the same report.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
