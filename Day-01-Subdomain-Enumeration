# Day 1 — Subdomain Enumeration: The Forgotten Door Nobody Locked

**Severity:** Informational → Critical (depends on what you find) | **Category:** Recon & Fingerprinting | **Series:** #150DaysInTheTrenches

---

## 🔍 What It Is

Subdomain enumeration is the process of discovering every subdomain tied to a target's
main domain — `api.company.com`, `uat.company.com`, `partner-portal.company.com`, and so
on. On paper it's just information gathering. In practice, it's usually where an
engagement actually starts finding bugs, because every subdomain is a separate
application, often built by a different team, on a different stack, with a different
security posture than the main site.

## 🌱 Why It Exists

This isn't a vulnerability in the traditional sense — it exists because of how
enterprises actually build software. A tax-filing platform doesn't get built once and
frozen; it accumulates a UAT environment, a staging environment, a partner integration
API, a legacy portal that was "supposed to be decommissioned," and a marketing
microsite — all under the same root domain, all provisioned at different times by
different teams under different deadlines. Nobody centrally tracks all of it. DNS
records, though, are public. So the "forgotten door" is never actually hidden — it's
just never been looked for.

## ⚔️ How To Find It

**Passive recon first** — no packets hit the target directly, which matters a lot when
you're working under a strict rules-of-engagement window (very common in banking and
income-tax engagements, where scope windows are tight and any active scanning outside
approved hours can trigger a client's SOC).

```bash
# Certificate Transparency logs — SSL certs are logged publicly, subdomains leak here
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u

# Passive DNS aggregation
subfinder -d target.com -silent

# Combine and dedupe multiple sources
amass enum -passive -d target.com -o amass_passive.txt
```

**Active validation** — once you have a candidate list, confirm which subdomains are
actually alive and worth digging into:

```bash
cat subdomains.txt | httpx -silent -title -status-code -tech-detect -o live_hosts.txt
```

Sort the output by technology stack and title — the interesting targets jump out fast:
anything with "UAT," "staging," "internal," "partner," or an obviously outdated
CMS/framework version in the tech-detect column.

## 💥 How To Exploit It

Subdomain enumeration by itself isn't the finding — it's the map. What you do with the
map is where impact comes from. In real engagements, this step has directly led to:

- A UAT environment left in debug mode, leaking stack traces with database connection
  strings
- A partner-integration subdomain with no authentication in front of an internal API
- A legacy portal running a framework version with a known public CVE, still reachable
  because "it's not on the main site so nobody worried about it"

## 🌍 Where This Actually Bites

In a **banking or income-tax context**, a forgotten UAT subdomain is rarely just a
cosmetic risk — these environments are frequently seeded with copies (or thin masks) of
production-like data for testing purposes, and access control there is treated as an
afterthought because "it's not production." I've seen this pattern repeat across
different domains: a **freight logistics** partner API subdomain exposed without proper
authentication that could reveal shipment and customer data across multiple client
accounts, or a **retail/e-commerce** staging environment still pointed at a live payment
gateway sandbox with real order data flowing through it. The vulnerability class is
"just recon," but the blast radius is entirely dictated by what got parked on that
subdomain and forgotten.

## 📋 How To Report It

Subdomain discovery on its own is usually **Informational** — report it as a footprint
observation, not a standalone finding, unless the client specifically wants an attack
surface inventory. The severity is driven entirely by what you find *on* the subdomain:

- **Critical** — unauthenticated access to production-equivalent data, RCE via an
  outdated stack, exposed admin panels with default credentials
- **High** — sensitive information disclosure (stack traces, internal API docs, source
  maps revealing business logic)
- **Low/Informational** — the enumeration itself, or dead/parked subdomains with no
  live service behind them

In the report, always pair the discovered subdomain list with a short "why this matters"
narrative — a raw list of hostnames means nothing to a client's engineering lead; a list
that says "these 6 subdomains are running end-of-life software" gets prioritized.

---

*Part of the #From-Dev-To-Attacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
