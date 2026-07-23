# Day 11 — Certificate Transparency Logs: A Public Diary of Every Subdomain a Company Has Ever Had

**Severity:** Medium (as a discovery vector) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Since 2013, every publicly-trusted SSL/TLS certificate has been required to log itself in a Certificate Transparency (CT) log before browsers will trust it. That requirement, meant to catch fraudulently-issued certificates, has a side effect that's enormously useful for recon: every subdomain a company has ever requested a certificate for — active, expired, or long since decommissioned — is permanently and publicly searchable.

## 🌱 Why It Exists

CT logging isn't a mistake at all — it's a deliberate, industry-wide security improvement. The reason it's valuable for recon is a mismatch in mental models: developers requesting certificates think about "is this cert valid" and rarely connect that to "this domain name is now permanently public record," even for short-lived staging environments that existed for a single sprint.

## ⚔️ How To Find It

```bash
# crt.sh JSON API — the fastest starting point
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | tr '\n' '\n' | sort -u

# Cross-check with a second aggregator for coverage
curl -s "https://api.certspotter.com/v1/issuances?domain=target.com&include_subdomains=true&expand=dns_names" | \
  jq -r '.[].dns_names[]'
```

The detail most testers skip: filtering by *expiry date*. A certificate that expired two years ago tells you exactly which subdomain existed at that time — often long enough ago that the team that built it has moved on, and the security posture on that forgotten host reflects standards from years earlier.

## 💥 How To Exploit It

CT mining doesn't exploit anything directly — it's the highest-signal, zero-noise way to build your subdomain target list, because it's a passive read against a public log rather than active queries against the target's own infrastructure. The wildcard certificate entries are worth a specific look too: a `*.internal.target.com` certificate tells you a naming convention exists for an internal subdomain pattern, even before you've found a single specific hostname matching it.

## 🌍 Where This Actually Bites

In an **education** sector engagement, CT logs surfaced a certificate for an admissions-portal subdomain from a prior academic year's enrollment cycle — expired, unlinked from current navigation, but still resolving and running the previous year's (unpatched) application code. In **retail/ecommerce**, I've used expired-certificate mining specifically to find seasonal-campaign subdomains (Black Friday microsites, limited-time promotional apps) that were built fast, secured adequately for their two-week lifespan, and then left running indefinitely after the campaign ended — because nobody's job was to go back and decommission them.

## 📋 How To Report It

CT mining is a technique, report the findings it leads to rather than the technique itself — with one exception: if the logs reveal subdomain naming conventions for genuinely sensitive internal systems (`vault.internal`, `db-primary`), that's worth a standalone Low/Informational note, since it's a permanent, un-removable public record that the client should factor into how they name anything internal going forward. For a decommissioned-but-still-running host found via an expired cert, treat the finding at the severity of whatever's actually running there — the expired certificate is just how you found the door, not the vulnerability itself.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
