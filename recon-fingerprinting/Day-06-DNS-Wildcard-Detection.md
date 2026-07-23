# Day 6 — DNS Wildcard Detection: When Every Subdomain "Exists"

**Severity:** Low (as a standalone issue) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

A DNS wildcard record (`*.target.com`) resolves *any* subdomain that doesn't have its own explicit record to a single IP. It's a legitimate feature — used for multi-tenant SaaS setups, CDN catch-alls, and generic error routing — but if you don't detect it before running subdomain brute-force, every fabricated name in your wordlist comes back as a "hit," and your entire recon dataset becomes noise.

## 🌱 Why It Exists

This isn't a developer mistake in the usual sense — it's usually an intentional infrastructure decision (route every unrecognized subdomain to a landing page, or let a multi-tenant platform assign customer subdomains dynamically without provisioning DNS per customer). The issue only becomes *your* problem as a tester when you don't check for it first and burn hours chasing thousands of identical false positives.

## ⚔️ How To Find It

```bash
# Query a subdomain that could never legitimately exist
RAND="wildcard-check-$(openssl rand -hex 8)"
dig A "$RAND.target.com" +short
```

If that returns an IP, a wildcard is active — every unknown subdomain resolves there. Tools built for scale (`dnsx -wd`, `puredns`) detect this automatically before brute-forcing and filter matching results, which is the far more efficient path than hand-rolling the check every time.

## 💥 How To Exploit It

Detecting the wildcard isn't the exploit — it's the filter that makes the *rest* of your recon trustworthy. Once you know the wildcard IP (or IPs, if a load balancer round-robins responses), everything resolving elsewhere is a genuine, individually-provisioned subdomain worth investigating.

## 🌍 Where This Actually bites

I've run into this most often in **retail/ecommerce** platforms that use wildcard subdomains for per-merchant storefronts (`merchantname.platform.com`) — a legitimate and common architecture. The risk shows up when that same wildcard is layered with a **subdomain takeover** condition (covered separately): if the catch-all target itself becomes unclaimed, every merchant subdomain riding on that wildcard is affected simultaneously, not just one. In **education** platforms running multi-tenant campus portals, the same pattern applies — one wildcard misconfiguration can expose or redirect traffic for every institution on the platform at once, which changes a single-tenant bug into a multi-tenant incident.

## 📋 How To Report It

A wildcard alone is Informational — it's a configuration detail, not a vulnerability, and should be reported that way if reported at all. Escalate it only when it's chained: if the wildcard target itself is claimable (a dangling CNAME behind the wildcard), or if it enables cookie-scope issues across every subdomain riding on it. When you do write it up, be explicit that the severity comes from the chain, not the wildcard itself, so the client doesn't spend engineering time "fixing" a non-issue instead of the actual takeover risk behind it.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
