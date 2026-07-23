# Day 2 — IDOR: The Bug That Doesn't Need a Single Line of "Hacking"

**Severity:** Medium → Critical (context-dependent) | **Category:** Access Control | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Insecure Direct Object Reference (IDOR) happens when an application lets you access a
resource — an order, an invoice, a document, an account — by referencing its identifier
directly (an ID in a URL, a hidden form field, an API path parameter), without properly
checking whether *you* are actually allowed to see *that* resource.

```
GET /api/v1/statements/104521    ← your statement
GET /api/v1/statements/104522    ← someone else's statement
```

Change one number, get someone else's data. No payload, no exploit code — just a
different integer.

## 🌱 Why It Exists

This is, hands down, the most common root cause I see from my developer background: the
team implemented **authentication** (are you logged in?) but never layered
**authorization** (are you allowed to see *this specific* record?) on top of it. It's an
easy mistake to make under deadline pressure — the happy path works, QA tests as the
logged-in user viewing their own data, and the "what if I request someone else's ID"
case simply never gets written as a test case, because it doesn't occur to anyone to
write it.

## ⚔️ How To Find It

1. Map every endpoint that takes an identifier as input — IDs in the URL path, query
   string, request body, or even in a JWT claim that the client controls.
2. Create two test accounts at the same privilege level.
3. Capture a request as Account A, then replay it with Account B's session
   token/cookie while keeping Account A's resource ID.

```bash
# Burp Repeater workflow, conceptually:
# 1. Log in as User A, capture: GET /api/orders/9001  with Cookie: session=A
# 2. Swap in User B's session cookie, keep the same order ID
# 3. If the response still returns User A's order data → IDOR confirmed
```

Don't stop at sequential numeric IDs — also test:
- **UUIDs** that were leaked elsewhere in the app (another endpoint that lists them)
- **Encoded/hashed IDs** (base64 an ID, decode it, increment, re-encode)
- **Second-order IDOR** — where the ID isn't in the request you control, but gets stored
  and used later in a background/async process

## 💥 How To Exploit It

Once confirmed, the exploitation is usually just enumeration:

```bash
for id in $(seq 1000 1050); do
  curl -s -H "Cookie: session=$USERB_SESSION" \
       "https://target.com/api/orders/$id" \
       -o "loot/order_$id.json"
done
```

The real work is in proving *business impact*, not just "I could see one other record."
A single IDOR hit is a bug; a script showing you can pull 500 other customers' records
in under a minute is the finding that gets a CVSS 9+ and gets fixed in the next sprint.

## 🌍 Where This Actually Bites

This is, in my experience, the single most domain-sensitive vulnerability class there
is:

- **Income Tax platforms** — an IDOR on a filed-return document endpoint means exposure
  of PAN numbers, income details, and bank account information for potentially every
  taxpayer using the portal. This is the kind of finding that gets escalated straight to
  the CISO within the hour.
- **Banking** — IDOR on statement or transaction-history endpoints is a direct
  regulatory incident (data privacy law violations, potential mandatory breach
  disclosure).
- **Retail / E-commerce** — order history, saved payment methods (even tokenized),
  addresses — IDOR here is a privacy and fraud-enablement risk (an attacker enumerating
  addresses + order values to target high-value customers).
- **Freight Logistics** — shipment tracking IDs are frequently sequential or predictable
  by design (for customer convenience), which makes IDOR nearly free to find, and it
  exposes shipper/consignee details, cargo manifests, and commercial terms between
  business partners.
- **Education** — student records, grades, and fee-payment history behind IDOR is a
  FERPA-style compliance problem in addition to the obvious privacy harm.

The technical bug is identical everywhere. The severity conversation with the client is
never the same twice — this is exactly why I always bring domain context into the
report, not just a CVSS score.

## 📋 How To Report It

- State the object type and the exact parameter that's insecurely referenced.
- Include a redacted proof — two side-by-side responses (your data vs. someone else's),
  with sensitive fields masked.
- Quantify blast radius: "IDs are sequential and span at least 400,000 active records" is
  far more persuasive than "IDOR found."
- Recommend **object-level authorization checks on every request**, not just
  session-level authentication — and recommend a code-review pass across *all* similar
  endpoints, since IDOR is rarely a single-endpoint bug; it's usually a pattern repeated
  across an entire API surface built by the same team.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
