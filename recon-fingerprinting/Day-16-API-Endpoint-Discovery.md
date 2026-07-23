# Day 16 — API Endpoint Discovery: Reading the Map the Company Already Published

**Severity:** Medium | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

API endpoint discovery is the systematic process of building a complete map of an application's API surface from every source that describes it — Swagger/OpenAPI specifications, GraphQL introspection, the JavaScript bundle shipped to browsers, and sometimes `robots.txt` itself. Most of an API's attack surface isn't hidden; it's documented somewhere the company just didn't intend for an outside tester to be reading it as a target list.

## 🌱 Why It Exists

None of these sources exist as a mistake — Swagger specs are meant for frontend/partner integration teams, GraphQL introspection is a development convenience, and the JS bundle has to contain every route the app calls by definition. The gap is that these are development-and-integration artifacts left reachable in production with the same access as everything else, rather than gated behind the internal network or an authenticated developer portal the way the company probably assumes they are.

## ⚔️ How To Find It

```bash
# Swagger/OpenAPI — check the common path variants
for p in /swagger.json /api-docs /v2/api-docs /openapi.json; do
  curl -s -o /dev/null -w "%{http_code} $p\n" https://target.com$p
done

# GraphQL introspection
curl -s -X POST https://target.com/graphql -H "Content-Type: application/json" \
  -d '{"query":"{__schema{queryType{fields{name}}mutationType{fields{name}}}}"}'
```

Parse whatever spec you find programmatically rather than reading it manually — extracting every path/method/auth-requirement combination into a flat list is what actually makes the discovery usable for systematic testing afterward.

## 💥 How To Exploit It

The map itself isn't the finding — it's the target list for everything else in your methodology. The one pattern worth calling out specifically: cross-reference which endpoints in the spec are marked as requiring authentication against which ones you can actually reach without a token. A spec that documents an endpoint as protected, where the live server doesn't actually enforce that, is a very fast route to an authorization bypass finding.

## 🌍 Where This Actually Bites

In a **freight logistics** partner API, a publicly reachable Swagger spec documented internal-only endpoints (`/api/v1/internal/carrier-rates`) alongside the public partner-facing ones, in the same file, with no separation — meaning any partner with API access to the documented surface could see the full internal route map too. In an **income tax** platform, GraphQL introspection left enabled in production surfaced mutation names like `adminOverrideFilingStatus` — the mutation existed, was reachable, and its very name told you exactly what to test for authorization gaps before you'd sent a single crafted query.

## 📋 How To Report It

The discovery itself is typically Informational — report what testing the discovered endpoints actually reveals. The one standalone finding worth its own write-up is GraphQL introspection left enabled in production: even without a follow-up authorization bug, handing any anonymous caller the complete schema (including internal-sounding mutation and type names) is a real information-disclosure finding on its own, and the fix (disable introspection outside development) is a one-line config change worth flagging clearly.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
