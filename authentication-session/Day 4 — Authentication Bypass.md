# Day 4 — Authentication Bypass: The Front Door That Isn't Actually Locked

**Severity:** Critical | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Authentication bypass is any technique that lets you reach an authenticated area or
action **without** ever presenting valid credentials — not by guessing a password, but
by finding a path the login check simply doesn't cover.

## 🌱 Why It Exists

In almost every case I've seen, this isn't one big design flaw — it's an inconsistency.
The login form itself is solid. But somewhere else in the application, the same
protected resource is reachable through a different route that the developer didn't
realize needed the same check:

- A REST endpoint that checks auth on `GET` but not on the equivalent `POST`/`PUT`
- A mobile API that duplicates web functionality but was built by a different team on a
  different sprint, with the auth middleware wired up slightly differently
- Client-side route guards (React/Angular) that hide a page in the UI, while the
  underlying API behind that page has no server-side check at all — the classic
  "security through UI hiding" mistake
- Forced browsing directly to an internal path that predates the current login flow
  (leftover from a previous version of the app)

## ⚔️ How To Find It

**Method/route inconsistency testing:**
```bash
# If GET requires auth, always test whether other methods do too
curl -i https://target.com/api/admin/settings          # expect 401
curl -i -X POST https://target.com/api/admin/settings  # sometimes... 200
```

**Direct object/path access without a token:**
Strip the `Authorization` header or session cookie entirely and hit endpoints you'd
expect to be protected — especially ones only ever reached client-side via a JS
redirect rather than a hard server-side check.

```bash
curl -i https://target.com/api/v1/reports/export --header "Authorization:"
```

**Parameter-based bypass:**
Some legacy apps (more common than you'd think in long-lived enterprise systems) have
debug or internal parameters left over from development:
```
POST /login
username=admin&password=x&debug=true
POST /api/user/profile?internal_bypass=1
```

**HTTP request smuggling / path normalization tricks:**
```
/admin/ vs /admin/. vs /admin%2f vs /ADMIN/  → some reverse-proxy + backend
combinations treat these differently, and the auth check sits at the wrong layer
```

## 💥 How To Exploit It

Once a bypass path is confirmed, chain it to something concrete — reading another user's
data, reaching an admin action, or triggering a state-changing operation. A bypass that
only "proves" you reached a page with no sensitive content is a weak finding; a bypass
that lets you export another tenant's report or modify a configuration value is what
gets prioritized for an emergency patch.

## 🌍 Where This Actually Bites

- **Income Tax / Banking** — an auth-bypass path that reaches a document export or
  statement-generation endpoint is about as bad as it gets: it turns what should be a
  fully authenticated flow into an open data pipe, and in regulated sectors this
  typically triggers mandatory incident reporting the moment it's confirmed in
  production.
- **Retail / E-commerce** — bypassing auth on order-management or admin discount
  endpoints can enable direct financial fraud (unauthorized discounts, order
  manipulation) rather than just data exposure.
- **Freight Logistics** — partner portals often have a "public tracking" flow alongside
  an "authenticated partner" flow sharing the same backend; a bypass here can expose
  commercial shipment details across companies that are supposed to be isolated from
  each other.
- **Education** — bypasses into faculty/admin areas of an LMS typically expose grade
  records and personally identifiable student data, and are a common finding in
  platforms that were extended over many years by different vendors.

## 📋 How To Report It

- Document the exact route/method/parameter that bypasses the check, and the *specific*
  inconsistency (method mismatch, missing server-side check behind a UI guard, stale
  debug parameter, path-normalization gap).
- Show the authenticated resource reached, redacted appropriately.
- Recommend centralizing authorization at a single enforcement layer (middleware/gateway)
  rather than per-route checks scattered across the codebase — this is almost always the
  actual root-cause fix, not just patching the one path that was found.
- Flag it for a broader review: if one method/route inconsistency exists, there are
  usually siblings elsewhere in the same API that were built the same way.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
