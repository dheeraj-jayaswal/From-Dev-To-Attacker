# Day 51 — GraphQL Security Testing: One Endpoint, Infinite Attack Surface

**Severity:** High → Critical | **Category:** API Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

GraphQL exposes its entire schema through introspection by design, and a single flexible endpoint replaces dozens of REST routes — which also means REST's implicit per-route authorization checks don't automatically carry over, and GraphQL-specific abuse patterns (deeply nested queries, batched requests) create denial-of-service and authorization-bypass paths that don't have a direct REST equivalent.

## 🌱 Why It Exists

Introspection is a genuinely useful development and tooling feature — GraphQL clients rely on it to auto-generate types and documentation — and disabling it in production is a deliberate step a team has to remember to take that's easy to skip, since nothing about a working GraphQL deployment requires it to be off. Nested-query and batching abuse exist because GraphQL's flexibility is the entire selling point of the technology — restricting query depth and complexity is an opt-in safeguard, not a default.

## ⚔️ How To Find It

```bash
curl -X POST https://target.com/graphql -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name fields{name}}}}"}'
```

Once the schema is recovered, look specifically for object-level authorization gaps at the field level — a query that returns a list of orders might correctly filter by the authenticated user at the top level, but a nested field resolving a related object (the customer record attached to that order) may not independently re-check ownership, since it's a different resolver function entirely.

## 💥 How To Exploit It

Introspection left enabled hands over the complete schema — type and mutation names alone (`adminOverrideStatus`, `deleteAllRecords`) often tell you exactly what to test next before you've sent a single crafted query. Batching abuse (sending many operations in a single request) can bypass request-level rate limiting that only counts HTTP requests, not the number of operations bundled inside one; deeply nested queries against a schema with circular relationships can cause severe server-side resource exhaustion with a single, small request.

## 🌍 Where This Actually Bites

In a **banking**-adjacent platform's GraphQL API, introspection left enabled in production revealed mutation names describing internal administrative actions that weren't documented anywhere else — the schema itself was the recon that told me exactly which authorization boundaries to test next. In an **income tax** platform, a nested query traversing from a filing record to its associated preparer to that preparer's other clients bypassed the top-level ownership check entirely — the nested resolver never independently verified the requesting user should see that data, since it was written by a different part of the team than the top-level query.

## 📋 How To Report It

Grade introspection-enabled-in-production as its own Medium finding regardless of what follow-up testing reveals, since it's a real information-disclosure issue on its own merit. Grade nested-field authorization gaps and batching-based rate-limit bypasses by their actual data-exposure or DoS impact — often High to Critical. Recommend disabling introspection outside development, enforcing query depth/complexity limits, and — the fix that actually matters most — auditing authorization at every resolver independently rather than assuming a top-level check covers nested fields underneath it.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
