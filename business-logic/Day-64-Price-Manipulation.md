# Day 64 — Price Manipulation: When the Client Is Trusted to Do the Math

**Severity:** High → Critical | **Category:** Business Logic | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Price manipulation happens when a checkout, cart, or order-submission flow trusts a price, discount, or total value sent from the client rather than recalculating it authoritatively on the server — letting an attacker simply edit the request to submit whatever price they'd prefer to pay, rather than the one the storefront actually displayed.

## 🌱 Why It Exists

Sending the computed price back to the server as part of the order-submission payload is often faster to implement than having the server recompute the entire cart total independently from product IDs and quantities alone — especially once discounts, taxes, and shipping are all factored into a single displayed number the frontend has already calculated for the user to see. Trusting that same number when the order is actually submitted is the shortcut that creates the vulnerability.

## ⚔️ How To Find It

```bash
# Intercept a checkout request and simply edit the price/total field before forwarding
curl -X POST https://target.com/api/checkout -H "Content-Type: application/json" \
  -d '{"productId": 4821, "quantity": 1, "price": 0.01}'
```

Test every field that plausibly contributes to a final charge independently — unit price, quantity (including negative quantities, covered separately), discount amount, and shipping cost — since some checkouts recompute one of these server-side while still trusting others.

## 💥 How To Exploit It

Confirming the order actually completes at the manipulated price (not just that the request was accepted, but that payment processing and order fulfillment proceed using the attacker-submitted value) is the concrete proof needed — a request that's accepted but silently recalculated server-side before charging isn't actually exploitable, and it's important to verify the full flow rather than stopping at request acceptance.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, the checkout API accepted a client-submitted `unitPrice` field directly, with no server-side lookup against the actual product catalog price — a trivially edited request completed real orders at a fraction of the actual price, fully processed through to fulfillment in the test environment. In **freight logistics**, a rate-quote-to-booking flow trusted the quoted shipping rate as submitted by the client at booking time rather than re-verifying it against the rate engine — a gap between quote generation and booking confirmation that allowed a stale or manipulated rate to be locked in for the actual shipment.

## 📋 How To Report It

Grade Critical when the manipulated price is confirmed to complete an actual transaction, with quantifiable per-transaction financial impact stated explicitly — this is one of the clearer, most business-relevant findings to communicate to a non-technical stakeholder, since the impact translates directly into a rupee/dollar figure. Recommend the server independently recalculate the entire order total from trusted server-side data (product catalog prices, current discount rules, tax tables) at the moment of final submission, never trusting any price-related value the client includes in the checkout payload.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
