# Day 50 — Lack of Rate Limiting: Unlimited Resource Consumption by Design

**Severity:** Medium → Critical (depends on the endpoint) | **Category:** API Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

An API without rate limiting will process as many requests as a client can send — which is fine for a well-behaved frontend, and a direct enabler of abuse for anyone else. This shows up in several distinct forms: brute force on auth endpoints (covered separately under authentication), but also resource exhaustion on expensive endpoints, scraping of paginated data at unlimited speed, and OTP/notification endpoints that can be triggered without bound.

## 🌱 Why It Exists

Rate limiting is infrastructure work that sits outside the feature being built, exactly like the lockout gap covered under authentication — an endpoint functions correctly for its intended use case without it, and the specific abuse patterns it prevents only become obvious when someone deliberately tests for them, not during normal feature development and QA.

## ⚔️ How To Find It

```bash
# Rapid-fire an endpoint and watch for 429 or any throttling signal
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code} " https://target.com/api/expensive-report
done
```

Test specifically against computationally expensive endpoints (report generation, search with complex filters, bulk export) and against any endpoint that triggers an external side effect per call (sending an SMS OTP, sending an email) — these have real-world cost implications beyond simple server load.

## 💥 How To Exploit It

Against an expensive endpoint, unlimited concurrent requests can degrade or take down the service entirely — a straightforward denial-of-service path with no other vulnerability required. Against an SMS/email-triggering endpoint, unlimited calls translate directly into a real financial cost to the business per request sent, and can be weaponized to harass a specific victim's phone with repeated OTP messages.

## 🌍 Where This Actually Bites

In a **freight logistics** platform, a shipment-cost calculator endpoint performing a complex multi-carrier rate lookup had no rate limiting — a moderate concurrent request volume degraded response times for all legitimate users on the same infrastructure, a real availability impact from an endpoint that looked, on paper, like a harmless calculator. In **retail/ecommerce**, an OTP-based checkout verification step had no limit on how many times the SMS could be re-triggered — beyond the direct cost per SMS sent, this is directly abusable to harass a specific customer's phone number by repeatedly initiating checkout with their number.

## 📋 How To Report It

Grade by consequence — DoS-capable resource exhaustion on a core service is High to Critical; unlimited OTP/notification triggering is Medium to High depending on both cost and harassment potential; unlimited pagination scraping of non-sensitive public data is often just Low. Recommend layered rate limiting (per-IP and, where authentication exists, per-account) with response headers indicating remaining quota, and flag any endpoint triggering a real-world side effect (SMS, email, push notification) as needing its own dedicated, stricter limit independent of the general API rate limit.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
