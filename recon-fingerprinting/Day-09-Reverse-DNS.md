# Day 9 — Reverse DNS: Turning a Bare IP Into an Attack Surface

**Severity:** Low (informational, but high recon value) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Reverse DNS asks the opposite question of a normal lookup: instead of "what IP does this domain resolve to," it's "what hostname is registered against this IP" — via the PTR record. It's an underused technique because most testers stop at forward enumeration (subfinder, amass, crt.sh) and never think to work backward from an IP they've already found.

## 🌱 Why It Exists

PTR records exist because plenty of internal tooling, logging systems, and legacy network conventions expect hostnames to be resolvable from IPs (mail server reputation checks are the most common legitimate reason). Companies rarely think about PTR records as something that needs to stay private, because forward DNS gets all the security attention — nobody audits what their reverse zone reveals.

## ⚔️ How To Find It

```bash
# Single IP
dig -x 93.184.216.34 +short

# At scale, across a discovered IP range
for i in $(seq 1 254); do echo "185.220.101.$i"; done | dnsx -ptr -resp -silent
```

The highest-value use isn't a single lookup — it's running reverse DNS across an entire IP block a company owns (found via ASN mapping) to surface hostnames that never appeared in any certificate transparency log or subdomain wordlist, because they were never meant to be public-facing at all.

## 💥 How To Exploit It

The technique itself doesn't exploit anything — it expands your target list. Three payoffs I rely on regularly: discovering internal-only hostnames like `db-replica` or `internal-api-v2` that passive recon completely misses; finding an origin server IP that bypasses a Cloudflare/CDN front, letting you test directly against the real backend; and mapping shared-hosting situations where one IP serves many tenants, meaning a misconfiguration found on one domain likely affects the others sharing that box.

## 🌍 Where This Actually Bites

In a **banking** engagement, reverse DNS across a client's ASN range surfaced a hostname (`vpn-legacy`) with no corresponding certificate or public documentation — a VPN gateway nobody on the current infra team even remembered existed, still accepting connections. In **income tax** platform testing, I've used origin-IP discovery via reverse DNS specifically to test whether the WAF/CDN in front of the public portal was actually enforcing controls consistently, versus a direct-to-origin path that skipped them entirely — a gap that, if real, undermines every other security control the client believes is protecting that portal.

## 📋 How To Report It

Reverse DNS itself is Informational — report what you find *through* it, not the technique. The one exception worth calling out on its own: if PTR records reveal internal naming conventions for sensitive infrastructure (database, VPN, CI/CD hostnames) that give attackers a targeting roadmap, note it as a Low finding with a simple remediation — generic hostnames for anything internet-adjacent, rather than descriptive names that hand out architecture details for free.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
