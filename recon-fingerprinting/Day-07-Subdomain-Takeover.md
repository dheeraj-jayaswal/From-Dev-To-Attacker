# Day 7 — Subdomain Takeover: Claiming a Door the Company Forgot to Lock

**Severity:** High | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Subdomain takeover happens when a DNS CNAME record still points to a third-party service (Netlify, Heroku, AWS S3, Shopify, GitHub Pages, and dozens of others) that the company has since deleted or deprovisioned. The DNS record is orphaned, but still live — and because most of these services let anyone register a resource name for free, an attacker can claim the exact name the abandoned record points to, and instantly control content served from the company's own subdomain.

## 🌱 Why It Exists

This is a process gap, not a code gap. Decommissioning a cloud resource and deleting its DNS record are two separate manual steps performed by two different teams (infra/DevOps deletes the app; the DNS owner is supposed to clean up the record). When a project is deprioritized, migrated, or its owning team changes, one of those two steps quietly gets skipped — and DNS records, unlike cloud resources, don't expire on their own.

## ⚔️ How To Find It

```bash
# Map CNAMEs across your subdomain list
cat subs.txt | while read sub; do
  cname=$(dig CNAME "$sub" +short)
  [ -n "$cname" ] && echo "$sub -> $cname"
done > cname_map.txt

# Check whether each destination is still claimed
```

Then confirm claimability with a fingerprint-aware tool — `nuclei -t takeovers/`, `subzy`, or `subjack` — which match known "not found" pages across 50+ services rather than requiring you to memorize each provider's error text.

## 💥 How To Exploit It

Confirming the fingerprint is enough for a report — you don't need to actually register the resource unless the engagement's scope explicitly permits it. The proof is the chain: dangling CNAME → destination returns a provider-specific "unclaimed" page → the resource name is available on that provider's signup flow (checked without completing registration).

## 🌍 Where This Actually Bites

I've seen this most often on marketing or campaign microsites in **retail/ecommerce** — a seasonal promotion subdomain built on a static-site host, torn down after the campaign ends, DNS never cleaned up. The company's brand trust is exactly what makes it dangerous: an attacker serving a phishing page from `promo.bigretailer.com` gets instant credibility no fake domain could match. In **education**, I've found this on event or admissions-portal subdomains built on third-party form platforms and abandoned after the enrollment cycle. And critically — if the parent domain scopes session cookies broadly (`domain=.company.com`), a takeover on even a minor, forgotten subdomain can enable cookie theft against the *main* application's logged-in users, turning a marketing-site cleanup miss into an account-takeover vector for the core product.

## 📋 How To Report It

Document the CNAME record, the provider's unclaimed-resource fingerprint, and confirmation that the name is available — without actually claiming it unless authorized. Score around CVSS 8.1, and be explicit in the impact section about cookie scope: state clearly whether the affected subdomain shares a parent domain with session cookies, since that single detail is usually what moves this from "defaced marketing page" severity to "account takeover" severity in the client's eyes. Recommend a standing DNS-cleanup checklist tied to any service decommission, not just a one-time fix for this record.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
