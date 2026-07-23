# Day 12 — JS Endpoint Extraction: Reading the API Map the Frontend Ships to Every Browser

**Severity:** Medium → Critical (depends on what's exposed) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Modern single-page applications (React, Angular, Vue) bundle their entire client-side logic — including every API route the app calls — into JavaScript files shipped to every visitor's browser. Since that code runs client-side, it's fully readable by anyone who opens dev tools or downloads the bundle directly. The API surface a company thinks is only visible through their documented endpoints is, in practice, sitting in plain text in the JS bundle.

## 🌱 Why It Exists

From the developer side, this isn't negligence — it's an unavoidable consequence of how SPA frameworks work. The frontend needs to know every endpoint it might call, and that code has to execute in the browser, so it has to be present in a form the browser can read. What actually causes the finding is a second-order mistake: developers building against a documented, "safe" set of endpoints often leave admin, debug, or internal routes in the *same* bundle because they were built by the same team using the same API client — with no separate build step to strip internal-only calls out of the public bundle.

## ⚔️ How To Find It

```bash
# Collect JS files (live crawl + historical via wayback)
gau target.com --ft js -o js_urls.txt

# Extract endpoints with LinkFinder
python3 linkfinder.py -i https://target.com/static/js/main.bundle.js -o cli

# Scan the same files for hardcoded secrets
python3 SecretFinder.py -i https://target.com/static/js/main.bundle.js -o cli
```

Run both extraction and secret-scanning against every JS bundle you find — they answer different questions (what endpoints exist vs. what credentials leaked) and both routinely turn up findings independently.

## 💥 How To Exploit It

Once you have the extracted endpoint list, the actual testing is straightforward: hit each undocumented route with your existing authenticated session and see what responds. The recurring pattern worth specifically watching for is an endpoint that exists in the bundle but was never wired into the visible UI — often an admin or export function the frontend team built defensively "in case we need it later" and simply left reachable.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, JS bundle analysis on a customer-facing storefront revealed an `/api/v1/internal/admin/export` route with no corresponding UI element anywhere in the app — reachable with a regular authenticated customer session, returning bulk order data across accounts. In a **freight logistics** partner portal, extraction from an older cached bundle (pulled via the Wayback Machine rather than the live site) surfaced a deprecated API version still live in production, missing the auth checks the current version had — the frontend had moved on, but the backend route it once called was never retired.

## 📋 How To Report It

For a hardcoded secret (API key, JWT signing secret, cloud credential) found in a bundle, treat it as Critical regardless of what it unlocks specifically — client-side secrets are permanently compromised the moment they ship, since there's no way to control who's already downloaded that JS file. For an undocumented endpoint, grade by what it actually returns when tested, not by the fact that it's undocumented — document the exact bundle file and line/context where you found the reference, and the authenticated request that confirmed it's live, so the dev team can trace it back to the specific frontend code path that references it.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
