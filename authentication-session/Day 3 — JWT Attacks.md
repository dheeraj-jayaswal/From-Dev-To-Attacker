# Day 3 — JWT Attacks: When "Stateless" Becomes "Unchecked"

**Severity:** High → Critical | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

JSON Web Tokens (JWT) are a compact, self-contained way to represent a user's identity
and claims — `{header}.{payload}.{signature}`. The appeal for developers is that the
server doesn't need to keep session state; it just verifies the signature and trusts
whatever's in the payload. JWT attacks target the gap between "the token is well-formed"
and "the server actually verified it correctly."

## 🌱 Why It Exists

From the developer side, JWT libraries are deceptively easy to misuse. A team migrating
from session-cookie auth to JWT-based auth (very common when a monolith gets broken into
microservices) often reaches for whatever JWT library is fastest to integrate, without
fully understanding the verification options it exposes. The result is one of a handful
of recurring mistakes:

- The server trusts the `alg` field in the token header instead of enforcing one
  expected algorithm
- The signing secret is weak enough to brute-force offline
- The server never actually calls the signature-verification function on certain code
  paths (a classic "we validate on login but not on every subsequent request" gap)

## ⚔️ How To Find It

Start by just decoding the token — no tool needed beyond base64:

```bash
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
# {"alg":"HS256","typ":"JWT"}
```

Look at what's in the payload. Role claims, user IDs, tenant/organization IDs sitting in
plaintext in the payload are all worth testing for tampering, since the only thing
stopping you from editing them is the signature check on the server side.

**Test 1 — Algorithm confusion (`alg: none`):**
```
Change header to {"alg":"none","typ":"JWT"}, drop the signature entirely,
keep the trailing dot: header.payload.
```
Some poorly configured verifiers will accept an unsigned token if `none` is an allowed
algorithm.

**Test 2 — RS256 → HS256 downgrade:**
If the server uses RS256 (asymmetric — public key verifies, private key signs) but the
verification code doesn't pin the expected algorithm, you can often re-sign a forged
token using **HS256**, with the server's own known-public RSA public key used as the
HMAC secret. Tools like `jwt_tool` automate this check directly:
```bash
python3 jwt_tool.py <token> -X k -pk public_key.pem
```

**Test 3 — Weak secret brute force (HS256):**
```bash
hashcat -a 0 -m 16500 jwt_hash.txt rockyou.txt
```

## 💥 How To Exploit It

Once you can forge or tamper a token, the exploit is usually a privilege or tenant
boundary jump: change `"role":"customer"` to `"role":"admin"`, or change
`"tenant_id":"1042"` to a different tenant's ID, re-sign (or exploit the `none`/downgrade
path so no valid signature is even needed), and replay the request.

```bash
# After forging the payload, re-encode and send:
curl -H "Authorization: Bearer $FORGED_JWT" https://target.com/api/admin/users
```

## 🌍 Where This Actually Bites

- **Banking / Income Tax** — JWTs are frequently used for the API layer behind mobile
  banking or e-filing apps. A `tenant_id` or `account_id` claim that isn't
  re-validated server-side against the authenticated user, combined with a weak or
  algorithm-confused signature check, is a direct path to cross-account access —
  arguably the worst possible finding in these domains.
- **Retail / E-commerce** — session JWTs that embed cart/pricing logic or loyalty-tier
  claims can be tampered to apply unauthorized discounts or tier-based benefits if the
  backend trusts the claim instead of re-checking against the database.
- **Freight Logistics** — partner/API-to-API auth between a shipper's system and the
  logistics platform is very commonly JWT-based (OAuth client-credentials flow feeding
  into a JWT). A weak secret here doesn't just expose one user's data — it can expose an
  entire partner integration to impersonation.
- **Education** — role claims (`student` vs `faculty` vs `admin`) sitting unchecked in a
  JWT payload is one of the more common findings in campus/LMS platforms, since these
  systems often bolt JWT auth onto an older codebase without a full authorization audit.

## 📋 How To Report It

- Show the exact claim you tampered and the before/after payload.
- Demonstrate the resulting privilege change concretely (e.g., an admin-only endpoint
  now returning 200 instead of 403).
- Call out the *specific* misconfiguration: unpinned algorithm, `none` acceptance, weak
  secret, or missing server-side re-validation of claims — because the fix differs for
  each ("pin algorithm + reject none," "rotate to a strong random secret + enforce
  minimum entropy," "re-check claims against the database on every privileged action").
- Recommend short token lifetimes + refresh-token rotation regardless of which specific
  bug was found — it limits blast radius for whatever the *next* JWT issue turns out to
  be.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
