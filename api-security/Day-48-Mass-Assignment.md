# Day 48 — Mass Assignment: The Field the Frontend Never Sent, but the Backend Still Accepted

**Severity:** High → Critical | **Category:** API Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Many web frameworks let a JSON request body bind directly to a database model's fields for convenience — one line of code maps the entire request onto the object. Mass assignment happens when that binding accepts *every* field the client chooses to send, including ones the frontend never exposes in its own forms, like `role`, `isAdmin`, or `accountBalance`.

## 🌱 Why It Exists

The convenient binding pattern (`User.update(req.body)`) is genuinely faster to write than manually allow-listing every permitted field, and it works correctly for the happy path the developer tested — the frontend's own form only ever sends the fields it exposes, so nothing looks wrong during normal development and QA. The gap only becomes visible when someone sends a request the frontend was never built to send in the first place.

## ⚔️ How To Find It

```bash
# Add plausible privilege-adjacent fields to a normal, legitimate request
curl -X POST https://target.com/api/register -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"Test123!","role":"admin","isVerified":true}'
```

Test this specifically at registration and profile-update endpoints — registration is the highest-value target since a successful mass-assignment there creates a privileged account from a completely unauthenticated starting point.

## 💥 How To Exploit It

Confirming the field was accepted is straightforward — check whether the created/updated resource actually reflects the injected value (does the new user genuinely have `role: admin`, or was the field silently ignored). Once confirmed at registration specifically, this is a direct, self-service path to a privileged account with zero social engineering or credential theft required.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** registration API, `accountTier` and `loyaltyDiscount` fields were silently accepted in the signup JSON body despite never being exposed in the actual signup form — a direct, repeatable path to elevated-tier pricing that has genuine, quantifiable financial impact at scale rather than being a purely theoretical finding. In an **education** platform's user-profile update endpoint, a `role` field distinguishing `student` from `faculty` was accepted from any authenticated session — a student account could self-promote to faculty-level access with a single crafted request.

## 📋 How To Report It

Grade Critical for any privilege-adjacent field (role, admin flag, verification status) accepted via mass assignment, and grade financially-adjacent fields (discount tier, pricing fields) as High to Critical depending on quantifiable business impact. Recommend explicit allow-listing of bindable fields at every endpoint that accepts a request body mapped to a model — deny-listing the "dangerous" fields is unreliable, since new sensitive fields added to a model later won't automatically be covered by a deny-list built against today's schema.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
