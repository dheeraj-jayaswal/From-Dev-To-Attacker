# Day 67 — Negative Quantity & Value Manipulation: When the Math Was Never Meant to Go Backward

**Severity:** High → Critical | **Category:** Business Logic | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Numeric fields representing quantity, weight, or amount are frequently validated only for type (is it a number) and not for sign or reasonable range — accepting a negative quantity in an order, a negative weight in a shipping calculation, or a negative amount in a refund request lets the underlying arithmetic run in reverse, often crediting the attacker rather than charging them.

## 🌱 Why It Exists

Type validation ("is this a valid number") is a natural, common check to add; range and sign validation ("is this number sensible for what it represents") is a separate, easy-to-forget check that requires thinking specifically about what values are actually meaningful in business terms — a quantity field being a valid integer is necessary but not sufficient, and the distinction between those two checks is exactly where this bug survives.

## ⚔️ How To Find It

```bash
curl -X POST https://target.com/api/cart/add -d '{"productId": 4821, "quantity": -5}'
curl -X POST https://target.com/api/refund -d '{"orderId": 9001, "amount": -50.00}'
```

Test negative values specifically on any field feeding into a total calculation — quantity, weight, discount amount, refund amount — and check whether the resulting total moves in the mathematically consistent (and business-illogical) direction: a negative quantity should, if unvalidated, *reduce* the total rather than erroring out.

## 💥 How To Exploit It

A negative quantity added alongside legitimate positive-quantity items can reduce a cart's total below the actual cost of the positive items — effectively a partial discount manufactured purely through arithmetic, or in the extreme case, a negative total that the payment gateway interprets as a credit rather than a charge. A negative refund amount, submitted where the field is expected only to ever be positive, similarly can invert an intended deduction into a credit.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, adding a large negative quantity of a low-value item alongside a genuine high-value item reduced the cart total to a fraction of the real cost — the checkout logic summed line-item totals without validating that any individual quantity was non-negative, and the payment gateway processed the resulting (still-positive but heavily discounted) total without complaint. In **freight logistics**, a shipment-weight field accepting negative values fed directly into a per-kilogram rate calculation — a crafted negative weight on one line item of a multi-item shipment reduced the total calculated freight charge below what the actual shipment weight warranted.

## 📋 How To Report It

Grade Critical whenever the manipulation is confirmed to complete an actual transaction at an incorrect (attacker-favorable) value, with the exact financial delta stated explicitly as evidence. Recommend validating every quantity/amount/weight field for both correct type and a sensible, business-appropriate range (non-negative, and typically an explicit reasonable upper bound too) at the point of input, not just at the final total-calculation step — validating only the final computed total misses the underlying per-field manipulation that produced it.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
