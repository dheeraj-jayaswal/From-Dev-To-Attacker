# Day 26 — X-Powered-By Disclosure: One Header, Full Backend Stack Revealed

**Severity:** Low | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Several backend frameworks — PHP, Express.js, ASP.NET — set an `X-Powered-By` header identifying the language and often the exact version, entirely by default, with no application code doing it deliberately. A single request reveals the backend runtime, which narrows the entire vulnerability research process down from "what could this be" to "what's the known CVE list for exactly this version."

## 🌱 Why It Exists

This is a framework default rather than a developer choice — nobody writes code to add `X-Powered-By: Express`, the framework does it automatically on every response unless a team explicitly disables it. It sits in the same category as server banners: a genuinely useful signal during early ecosystem development, never revisited as a hardening step in most production deployments.

## ⚔️ How To Find It

```bash
curl -sI https://target.com | grep -i x-powered
```

Running this across every enumerated subdomain in one pass (`httpx` with header inspection) is far more efficient than checking hosts individually, and it's worth doing specifically to spot version outliers — a fleet running PHP 8.x consistently except one subdomain still on PHP 5.6 is the finding worth chasing.

## 💥 How To Exploit It

Version-specific chains follow directly: an old PHP version narrows the search to type-juggling and deserialization bugs; Express specifically raises the question of whether the app does unsafe object merging anywhere reachable (prototype pollution); an ASP.NET version points toward checking whether ViewState MAC validation is disabled, which opens a deserialization path.

## 🌍 Where This Actually Bites

In a **banking**-adjacent fintech client's estate, `X-Powered-By: PHP/5.6` turned up on one internal-facing tool while every customer-facing service ran current PHP — a legacy internal admin tool that had clearly been deprioritized for patching because "it's not customer-facing," despite still processing sensitive account data. This is the recurring pattern worth calling out specifically: internal tooling is disproportionately where old runtime versions survive, precisely because it doesn't get the same security attention as customer-facing surfaces.

## 📋 How To Report It

Grade Low on its own; escalate only when paired with a version-specific CVE, exactly as with server banners. The remediation is genuinely trivial — one config line per stack (`expose_php = Off`, `app.disable('x-powered-by')`) — so frame this as a quick, low-effort hardening fix rather than a major finding, while still flagging the underlying pattern (why is this one host running an outdated version) if that's what the disclosure actually revealed.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
