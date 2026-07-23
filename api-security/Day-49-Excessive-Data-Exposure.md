# Day 49 — Excessive Data Exposure: The API Returns More Than the UI Shows

**Severity:** Medium → Critical (depends on the extra fields) | **Category:** API Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Excessive data exposure happens when an API endpoint returns a full backend object — every field the database record has — and relies entirely on the frontend to selectively display only some of them, rather than the API itself filtering the response down to what's actually needed. Anyone inspecting the raw HTTP response (not just using the rendered UI) sees everything, filtered fields included.

## 🌱 Why It Exists

Returning the full object from a database query is the path of least resistance during development — building a dedicated response schema that strips sensitive fields is genuinely extra work that has to be done deliberately, separate from just wiring up the query and serializing whatever it returns. It's a shortcut that works perfectly during development and never gets revisited once the frontend correctly hides the extra fields from view.

## ⚔️ How To Find It

The check is simple but frequently skipped: open the browser's network tab (or intercept with Burp) and compare the raw JSON response body against what's actually rendered on screen — any field present in the response but absent from the UI is a candidate finding, regardless of whether it seems sensitive at first glance.

```bash
curl -H "Authorization: Bearer $TOKEN" https://target.com/api/users/42 | jq .
# Compare every key in the response against what the UI actually displays
```

## 💥 How To Exploit It

No active exploitation is needed — the data is already in the response, sent to any authenticated (sometimes unauthenticated) caller. The finding is in identifying which of the extra fields are actually sensitive: a `password_hash` field, an internal `salary` or `commission` field on an employee-facing app, or another user's PII sitting in a "list users" response that the UI only renders as a name and avatar.

## 🌍 Where This Actually Bites

In an **education** platform's student-roster API, the response for a "class list" feature included each student's full guardian contact information and fee-payment status — fields the teacher-facing UI never displayed, present in the raw response purely because the endpoint returned the complete student record rather than a purpose-built subset. In **banking**-adjacent internal tooling, a "customer lookup" API used by support staff returned the customer's full transaction history and internal risk-score fields in the raw response even though the support UI only ever rendered name and account status — meaning any support staffer inspecting network traffic (or a compromised support session) had access to far more than their role was meant to see.

## 📋 How To Report It

Grade by the sensitivity of the specific extra fields found, not by the existence of over-fetching in general — a few harmless internal timestamp fields are a Low cleanup item, while password hashes, other users' PII, or financial/risk data in the response is Critical. Recommend building explicit response DTOs/serializers per endpoint that only include intentionally-exposed fields, rather than relying on the frontend to be the only thing standing between a user and the full backend object.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
