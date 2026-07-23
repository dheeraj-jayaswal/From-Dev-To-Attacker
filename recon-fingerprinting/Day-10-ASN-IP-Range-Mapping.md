# Day 10 — ASN & IP Range Mapping: Finding the Servers That Never Had a Domain

**Severity:** Low (recon technique, high downstream value) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Every organization that manages its own IP routing has an Autonomous System Number (ASN) — a registration identifier tied to the entire block of IP addresses they've been allocated. Mapping a target's ASN gives you every IP range they own, including servers that were never given a public DNS name and would never show up in subfinder, amass, or certificate transparency logs no matter how thorough those tools are.

## 🌱 Why It Exists

This isn't a misconfiguration — it's just how internet routing works, and most testers simply never look at this layer because domain-based recon feels complete after a few tools return results. The gap is a blind spot in methodology, not a flaw in the target's setup: a company's attack surface is bigger than its DNS zone, and IP-only infrastructure is exactly where things get deployed without going through the same change-management process as anything with a public hostname.

## ⚔️ How To Find It

```bash
# Find the ASN from a known IP
TARGET_IP=$(dig A target.com +short | head -1)
whois "$TARGET_IP" | grep -iE "OriginAS|OrgName|NetRange"

# Or via bgpview.io
curl -s "https://api.bgpview.io/ip/$TARGET_IP" | jq '.data.prefixes'

# Pull every prefix owned by that ASN
curl -s "https://api.bgpview.io/asn/12345/prefixes" | jq -r '.data.ipv4_prefixes[].prefix'
```

From there, sweep for live hosts (`nmap -sn` for small ranges, `masscan` at scale), then reverse-DNS and port-scan whatever responds.

## 💥 How To Exploit It

The technique surfaces targets, not vulnerabilities directly — but the targets it surfaces are disproportionately valuable. Servers that exist only as bare IPs, with no DNS record, frequently sit outside the change-management process that governs anything with a public hostname: no WAF in front of them, older unpatched software, admin interfaces that were "internal only" back when the IP was first assigned and never revisited since.

## 🌍 Where This Actually Bites

I've used ASN mapping in **freight logistics** engagements to identify partner-integration servers that were provisioned years earlier for a specific carrier relationship and never decommissioned when that relationship ended — running on IPs with no DNS record, no monitoring, and credentials nobody had rotated. In **banking**, I've found admin tooling (an internal dashboard, a monitoring stack) exposed on a raw IP within the client's ASN that was clearly meant to be internal-only but had no network-level restriction — the WAF and CDN in front of the main banking portal meant nothing to a server one hop away on the same IP block.

## 📋 How To Report It

ASN mapping itself isn't a finding — frame it in the report as your methodology, then report what you actually discovered on the exposed IPs. When you do find something on an IP-only host, be explicit in the write-up about *how* you found it (ASN → prefix → live host sweep → this specific IP), since that context tells the client's infra team exactly where their asset-inventory process has a gap: they're tracking domains, not IP allocations, and that's the process fix this finding should drive — not just patching the one server you happened to find.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
