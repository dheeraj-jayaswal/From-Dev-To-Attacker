# Day 63 — Race Conditions: Exploiting the Gap Between Check and Action

**Severity:** High → Critical (financial impact) | **Category:** Business Logic | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Almost every business rule enforced in code follows a "check, then act" pattern — check the coupon hasn't been used, then mark it used; check the account balance is sufficient, then debit it. A race condition exploits the gap between those two steps: if many requests hit the same check simultaneously, all of them can pass the check *before* any single one completes the corresponding update, letting an attacker redeem a coupon multiple times, or withdraw more than an account actually holds.

## 🌱 Why It Exists

Writing check-then-act logic that's genuinely atomic (a database transaction with proper locking, or an atomic increment/decrement operation) requires deliberately reasoning about concurrent access — the straightforward, sequential version of the logic reads correctly and passes every functional test that sends requests one at a time, and the race only becomes visible under genuinely concurrent load, which most QA processes don't specifically simulate.

## ⚔️ How To Find It

```python
# Fire the same request many times, simultaneously, not sequentially
import concurrent.futures, requests
def redeem(): return requests.post("https://target.com/api/redeem-coupon", json={"code": "SAVE20"})
with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
    results = list(ex.map(lambda _: redeem(), range(20)))
```

Burp Suite's Turbo Intruder, configured for high concurrency with single-packet attack mode, is the standard tool for this — the goal is getting as many identical requests to arrive at the server within the smallest possible time window, since the exploit window itself is frequently a matter of milliseconds.

## 💥 How To Exploit It

Any endpoint enforcing a single-use limit, a balance check, or an inventory count is a candidate: coupon-redemption limits, gift-card balance debits, "claim this limited stock item" flows, and account-verification steps that should only complete once. Confirming the race is as simple as observing that more redemptions/debits succeeded than the business rule should have allowed.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, a single-use discount coupon could be redeemed simultaneously across 20 concurrent requests, all succeeding — a direct, quantifiable financial loss per exploited coupon code, and trivially repeatable at scale against any coupon code an attacker obtained. In **banking**-adjacent fintech testing, a funds-transfer endpoint's balance check and debit step weren't wrapped in a single atomic transaction — concurrent transfer requests from an account with a small balance succeeded well beyond what the account actually held, a directly quantifiable overdraft created purely through request timing rather than any credential compromise.

## 📋 How To Report It

Grade by direct financial or business impact — coupon/discount race conditions are High given quantifiable but bounded loss per incident; balance-check races on financial platforms are Critical given the potential for unbounded overdraft exploitation at scale. Demonstrate the race with a concrete before/after count (redemption count, resulting balance) rather than just describing the technique, and recommend enforcing atomicity via database-level transactions with appropriate locking, or an atomic decrement operation, rather than any application-layer "check then act" pattern regardless of how carefully it's sequenced in code.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
