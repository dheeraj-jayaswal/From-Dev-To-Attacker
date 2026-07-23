# Day 34 — Security Header Analysis: Missing Headers Are Missing Defenses

**Severity:** Medium (individually) → High (in combination) | **Category:** Security Misconfiguration | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

HTTP security headers instruct the browser how to defend itself while rendering a page — whether to allow framing (`X-Frame-Options`), which script sources to trust (`Content-Security-Policy`), whether to enforce HTTPS (`Strict-Transport-Security`). Missing headers don't create a vulnerability by themselves; they remove a layer of browser-side mitigation that would otherwise contain the impact of a vulnerability found elsewhere.

## 🌱 Why It Exists

These headers require deliberate server configuration — none of them are set by a framework automatically in most default setups — and because their absence doesn't break anything visibly, they rarely surface as a bug during normal QA. A missing CSP header doesn't cause a support ticket the way a missing feature would; it just quietly removes a safety net that nobody notices is gone until it's needed.

## ⚔️ How To Find It

```bash
curl -sI https://target.com | grep -iE "content-security-policy|strict-transport-security|x-frame-options|x-content-type-options"
```

CORS deserves a distinct, separate check from the standard header sweep — reflecting an arbitrary `Origin` back in `Access-Control-Allow-Origin` combined with `Access-Control-Allow-Credentials: true` is a materially more serious finding than a missing CSP header, since it directly enables cross-origin authenticated requests rather than just weakening a mitigation layer.

## 💥 How To Exploit It

The value of this category is almost entirely in combination with another finding: a missing CSP header is what turns a found XSS into full, unmitigated script execution rather than one partially contained by a script-source allowlist; a reflected-origin CORS misconfiguration turns a valid session into something a malicious third-party site can silently ride along with to hit authenticated API endpoints on the victim's behalf.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, a reflected-origin CORS misconfiguration combined with credentials enabled meant any site the logged-in customer visited could silently query their order history and saved address data via a background request — no XSS or other vulnerability required, the misconfiguration alone was sufficient. In an **education** platform, a missing CSP header turned what would have otherwise been a contained, low-impact stored XSS finding into full session-cookie theft, since there was no script-source restriction stopping the injected payload from exfiltrating data to an external domain.

## 📋 How To Report It

Report missing standard headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options) together as one Medium finding with a clear per-header remediation table, rather than as separate low-value individual findings — clients respond better to "here are the six headers to add" than to six disconnected tickets. Grade CORS misconfiguration with reflected origin and credentials enabled as its own, more serious finding — High at minimum — since it's directly exploitable on its own rather than being a mitigation gap that only matters alongside another bug.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
