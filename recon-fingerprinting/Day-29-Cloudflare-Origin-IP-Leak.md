# Day 29 — Cloudflare Origin IP Leak: Finding the Server Behind the Shield

**Severity:** High | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

When a site sits behind Cloudflare (or any CDN/WAF), every request through the public domain is filtered and rate-limited by that layer. If the origin server's real IP address ever leaks — through DNS history, a mismatched SSL certificate, or a subdomain that was never proxied — an attacker can connect directly to it, bypassing every protection the CDN was providing entirely.

## 🌱 Why It Exists

This is almost always a historical DNS artifact rather than a live misconfiguration: the origin IP was public before Cloudflare was added in front of it, and old DNS records pointing to that IP persist in passive-DNS history services indefinitely. It also recurs operationally — a new subdomain (a staging environment, an internal tool, an email server's MX record) gets added to DNS without being proxied through Cloudflare, quietly exposing the same origin the main site is trying to protect.

## ⚔️ How To Find It

```bash
# DNS history is the most reliable source
curl -s "https://api.securitytrails.com/v1/history/target.com/dns/a" -H "APIKEY: $KEY"

# SSL certificate matching across non-CDN IP ranges
shodan search "ssl:target.com" --fields ip_str,port
```

Non-proxied subdomains are worth checking systematically too — resolve every enumerated subdomain and filter out anything landing in Cloudflare's published IP ranges; whatever remains is a candidate origin.

## 💥 How To Exploit It

Confirm the candidate IP is genuinely the origin by sending a request with the target's hostname forced in the `Host` header directly to that IP — matching content confirms it. Once confirmed, every WAF-blocked payload becomes testable again: XSS and SQLi attempts that Cloudflare's managed rules were filtering at the edge now land unfiltered directly at the application, and any rate limiting Cloudflare enforced no longer applies either.

## 🌍 Where This Actually Bites

In a **banking**-adjacent engagement, DNS history revealed the origin IP predated the client's Cloudflare migration by several years — the origin server hadn't moved, only the DNS record in front of it had changed, meaning the "protected" application was reachable directly the entire time via that legacy IP. In **freight logistics**, an MX record for the client's mail server resolved to the same host serving the web application — email infrastructure is routinely left unproxied since CDNs don't front SMTP, and in this case that unrelated record was the thread that unraveled the whole origin-IP protection.

## 📋 How To Report It

Frame the finding around what the leak actually enables, not just the fact of the leak itself — demonstrate at least one payload or behavior that's blocked through the CDN but succeeds directly against the origin, since that's the concrete evidence that turns "we found your real IP" into a finding a client will prioritize. Recommend firewall rules restricting the origin server to accept connections only from Cloudflare's published IP ranges, since rotating the IP alone doesn't fix the underlying gap — the next DNS artifact or unproxied subdomain will leak the new one just as easily without that network-level restriction in place.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
