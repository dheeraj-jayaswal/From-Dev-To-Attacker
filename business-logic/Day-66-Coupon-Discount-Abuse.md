# Day 66 — Coupon & Discount Abuse: Business Logic Bugs That Cost Real Money

**Severity:** Medium → High (scales with volume) | **Category:** Business Logic | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Beyond the race-condition variant of coupon reuse covered separately, discount logic has its own class of sequential, non-timing-dependent bugs: stacking multiple discount codes that were meant to be mutually exclusive, applying a first-time-customer discount to an account that's already ordered before, or manipulating a cart to qualify for a threshold-based promotion (buy-2-get-1-free) without genuinely meeting its intended condition.

## 🌱 Why It Exists

Discount logic tends to be built incrementally, one promotion type at a time, and each new discount rule is usually tested in isolation against a clean cart rather than in every possible combination with every other active promotion — the interaction between two independently correct discount rules is exactly where the gap appears, since neither rule's own logic is wrong in isolation.

## ⚔️ How To Find It

Systematically test combinations rather than single discount codes in isolation: apply two codes that marketing intends to be mutually exclusive and check whether both discounts actually stack in the final total; test whether a "first order" discount checks the account's actual order history or just a client-supplied flag; test threshold promotions with edge-case cart compositions (returning and re-adding an item to trigger a quantity-based discount without an intended qualifying purchase).

```bash
curl -X POST https://target.com/api/cart/apply-coupon -d '{"code":"WELCOME10"}'
curl -X POST https://target.com/api/cart/apply-coupon -d '{"code":"SUMMER20"}'
# Check final total — did both apply, when only one should have?
```

## 💥 How To Exploit It

Once a stacking or eligibility-check gap is confirmed, the exploitation is simply repeating it at scale — a stackable discount combination or an incorrectly-scoped "first order" promotion doesn't require any special access, just repeated use by any customer who discovers it, which is exactly why these findings matter at a business level even when no data is exposed and no account is compromised.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, two promotional codes explicitly documented internally as mutually exclusive both applied successfully to the same cart, compounding into a discount far beyond what either promotion alone was designed to offer — a finding with directly quantifiable revenue impact once the client understood how easily it could be discovered and shared publicly. In the same sector, a "first-time customer" discount checked only a client-supplied `isFirstOrder` flag rather than the account's actual order history — any returning customer could simply set that flag and receive new-customer pricing indefinitely, repeated on every order.

## 📋 How To Report It

Grade by realistic exploitation scale rather than per-incident value — a discount bug worth a few dollars per use is still High severity if it's trivially discoverable and shareable (coupon-stacking bugs frequently spread across deal-forums once found), since the actual business exposure is the aggregate abuse across potentially thousands of customers, not a single transaction. Recommend re-validating discount eligibility and mutual-exclusivity rules entirely server-side at the final checkout calculation step, and specifically flag any "self-reported" eligibility flag (like a first-order indicator) as needing to be derived from actual account history rather than trusted from the client.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
