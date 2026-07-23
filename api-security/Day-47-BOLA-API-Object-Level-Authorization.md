# Day 47 — BOLA: The #1 Finding in Every API Assessment

**Severity:** Critical | **Category:** API Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Broken Object Level Authorization is IDOR's API-native form, and OWASP ranks it the number-one API security risk for a reason grounded in how APIs get built: every endpoint takes an object identifier, and unless each one independently verifies the authenticated caller actually owns that specific object, swapping the ID in the request path reaches someone else's data.

## 🌱 Why It Exists

APIs multiply this risk relative to traditional web apps because a single resource is typically reachable through several HTTP methods (`GET`, `PUT`, `PATCH`, `DELETE`) that each need their own independent authorization check — teams frequently protect the `GET` (since that's the one that shows up first in testing and code review) while `PUT`/`DELETE` on the identical resource path get built later, by someone assuming the existing pattern already handles authorization consistently across methods.

## ⚔️ How To Find It

```bash
# Capture as User A, replay with User B's token, same object ID, across every method
curl -H "Authorization: Bearer $USER_B_TOKEN" https://target.com/api/orders/9001
curl -X PUT -H "Authorization: Bearer $USER_B_TOKEN" https://target.com/api/orders/9001 -d '{...}'
curl -X DELETE -H "Authorization: Bearer $USER_B_TOKEN" https://target.com/api/orders/9001
```

Test every method against every object-bearing endpoint systematically rather than assuming a passing `GET` test means `PUT`/`DELETE` are equally protected — this is the single most common mistake in API pentest coverage.

## 💥 How To Exploit It

A read-only BOLA is a data-exposure bug; a write/delete BOLA is materially worse, letting an authenticated attacker modify or destroy another user's records entirely without ever compromising that user's credentials. Automate the sweep across a large ID range once a single instance is confirmed, to demonstrate scale rather than a single isolated record.

## 🌍 Where This Actually Bites

In an **income tax** filing API, `GET` on a filed-return endpoint correctly enforced ownership, but the underlying `PUT` used for amendment submissions did not — meaning an authenticated taxpayer could not just view but actually modify another taxpayer's filed return data. In a **freight logistics** partner API, `DELETE` on a shipment-tracking record had no ownership check at all, letting any authenticated partner account cancel or remove tracking records belonging to a completely different partner company.

## 📋 How To Report It

Grade Critical for any write or delete-capable BOLA without qualification, and grade read-only BOLA by data sensitivity — PII or financial data is Critical, less sensitive data may be High. Always state explicitly in the report which HTTP methods were tested and which were found vulnerable versus protected, since a partial fix that only closes the specific method you demonstrated (while leaving siblings on the same resource open) is the most common incomplete-remediation outcome with this bug class.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
