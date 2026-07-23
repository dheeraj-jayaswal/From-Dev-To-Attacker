# Day 37 — Login Brute Force & Credential Stuffing: The Boring Bug That Still Pays

**Severity:** High → Critical (no MFA) | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Brute force tries password guesses against a known or enumerated username; credential stuffing replays real username:password pairs from prior breach dumps against a completely unrelated target, betting on password reuse. Both attacks are only viable when the login endpoint doesn't rate-limit or lock out repeated failed attempts — a genuinely simple control whose absence is still one of the most common, highest-severity findings in modern engagements.

## 🌱 Why It Exists

Rate limiting is infrastructure work that sits outside the core feature being built, and it's easy for a login endpoint to ship functionally complete — accepting correct credentials, rejecting incorrect ones — without anyone adding the specific control that limits *how many* incorrect attempts are allowed per minute. It's a control that has to be deliberately added; nothing about a basic login implementation includes it by default.

## ⚔️ How To Find It

```bash
for i in $(seq 1 25); do
  curl -s -o /dev/null -w "%{http_code}\n" -X POST https://target.com/api/login \
    -d "{\"username\":\"admin\",\"password\":\"test$i\"}"
done
```

Twenty-five identical response codes with no `429`, no lockout message, and no CAPTCHA trigger confirms the absence of protection — at that point, a real attack is simply a wordlist and a tool like Hydra or Burp Intruder pointed at the same endpoint.

## 💥 How To Exploit It

Password reuse rates make credential stuffing the higher-value variant in most engagements: a breach-derived list of real username:password pairs, replayed against an unrelated target, typically succeeds against a meaningful percentage of accounts purely because people reuse passwords across services. This requires no cleverness — just an unprotected endpoint and a list.

## 🌍 Where This Actually Bites

In an **income tax** platform engagement, the taxpayer login portal had no rate limiting and no MFA enforcement — a credential-stuffing test against a small sample of breach-derived pairs (used strictly for demonstrating the finding, with client authorization) succeeded against a nontrivial fraction, confirming that the absence of basic throttling directly enabled account takeover on a platform holding some of the most sensitive personal financial data a person has. In **retail/ecommerce**, the same gap on a customer account login is lower individual stakes but higher volume — unprotected login endpoints on high-traffic consumer platforms are exactly what large-scale credential-stuffing campaigns are built to exploit at scale.

## 📋 How To Report It

Grade Critical when there's no MFA and no rate limiting together — this combination is close to as bad as authentication gets, since a working password guess is immediate, unmitigated account takeover. Even with MFA present, absence of rate limiting is still High, since it enables account lockout/DoS style abuse and confirms valid credentials even if the second factor blocks full takeover. Recommend the standard layered fix explicitly: per-IP and per-account rate limiting, progressive lockout with backoff, and CAPTCHA after a small number of failures — a single control alone is rarely sufficient on its own.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
