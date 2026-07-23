# Day 20 — Source Map Exposure: The Debugging Tool That Ships Your Whole Codebase

**Severity:** High | **Category:** Sensitive Data Exposure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

JavaScript source maps translate minified, bundled production code back into its original, readable source — a genuinely useful developer tool for reading real stack traces in browser DevTools. When the `.map` file itself is deployed to a production server without access restriction, anyone can download it and reconstruct the entire original frontend codebase, not just a stack trace — every file, every comment, every hardcoded value.

## 🌱 Why It Exists

Build tooling generates source maps by default in most frontend frameworks, and the natural workflow is "same deployment pipeline as the JS bundle it maps to" — nobody makes a deliberate decision to expose it, it just rides along with everything else the build step outputs, unless someone explicitly configures the production build to exclude it (`devtool: false` in webpack, for instance).

## ⚔️ How To Find It

```bash
# Check the last line of any bundled JS file for the reference
curl -s https://target.com/static/js/main.chunk.js | tail -3
# //# sourceMappingURL=main.chunk.js.map

# Then check if the map itself is reachable
curl -s -o /dev/null -w "%{http_code}\n" https://target.com/static/js/main.chunk.js.map
```

A purpose-built tool like `sourcemapper` or `unwebpack-sourcemap` reconstructs the full original directory structure from the map's `sourcesContent` field in one step, which is far faster than manually parsing the JSON for anything beyond a quick spot-check.

## 💥 How To Exploit It

Once reconstructed, treat the recovered source exactly like a leaked source-code repository: grep systematically for hardcoded API keys and JWT signing secrets, undocumented admin route definitions, and — the pattern I check for specifically — developer comments flagging a debug bypass ("TODO: remove before production", a hardcoded header check that skips authentication). These bypass comments show up in source maps more often than you'd expect, because they were written for the developer's own convenience and never meant to survive to a production deployment.

## 🌍 Where This Actually Bites

In a **banking** engagement, a recovered source map revealed a hardcoded JWT signing secret in a config file that never made it into the visible, minified bundle in any readable form — without the map, that secret was effectively invisible; with it, forging arbitrary authenticated tokens became straightforward. In **retail/ecommerce**, source map recovery on a customer portal surfaced a complete internal admin route list (`adminRoutes.js`) that had never been documented anywhere else — turning a single exposed `.map` file into a full map of every admin function worth testing for authorization gaps.

## 📋 How To Report It

Grade this High by default given how consistently source maps expose route structure and configuration values, and escalate to Critical the moment a live credential or signing secret is confirmed in the recovered source. In the report, list the specific sensitive files recovered (not the full source dump) and quote the exact line containing a bypass comment or hardcoded secret as evidence, since that specificity is what turns "source maps were exposed" into a finding an engineering team can act on immediately. Recommend disabling source map generation for production builds entirely, rather than just removing the currently-found file — the next deployment will regenerate it otherwise.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
