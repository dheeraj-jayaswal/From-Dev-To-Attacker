# Day 35 — Username Enumeration via Response Differences

**Severity:** High | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

When a login, registration, or password-reset endpoint returns a visibly different response for a valid username than for an invalid one — a different status code, error message, or response length — an attacker can feed a wordlist through the endpoint and walk away with a confirmed list of real accounts on the platform, without ever needing a correct password.

## 🌱 Why It Exists

This is a natural byproduct of writing clear, helpful error messages: a developer building a login flow writes "user not found" and "incorrect password" as two distinct, genuinely more debuggable error paths, without weighing that the distinction itself is the information an attacker wants. It's a UX-first decision that nobody consciously frames as a security tradeoff at the time it's made.

## ⚔️ How To Find It

```bash
# Establish a baseline with a known-valid and known-fake username, compare status/length/body
curl -s -o /dev/null -w "%{http_code} %{size_download}\n" -X POST https://target.com/api/login \
  -d '{"username":"knownuser","password":"wrong"}'
```

Run a wordlist through Burp Intruder or a scripted equivalent, sorting by status code and response length rather than reading each response individually — the anomaly (a small number of entries returning a different status or size than the bulk baseline) is what you're looking for, not any single response in isolation.

## 💥 How To exploit It

Enumeration on its own isn't compromise — it's the input list for every subsequent authentication attack: targeted password spraying against only confirmed-valid usernames, credential-stuffing lists filtered down to accounts that actually exist, or social-engineering targeting where confirming an email is a company employee is itself useful.

## 🌍 Where This Actually Bites

In a **banking**-adjacent engagement, a corporate SSO login endpoint returned a distinct status code for valid employee usernames — confirming which email-format usernames belonged to real employees, directly narrowing a subsequent phishing or password-spray campaign to a validated target list rather than a guess. In **retail/ecommerce**, the same pattern on a customer-facing registration endpoint ("email already registered" vs. "available") let me confirm whether a specific email address had an account at all — a lower-stakes context, but still a privacy-relevant disclosure worth flagging, since it lets anyone check whether a given email is a customer of the platform.

## 📋 How To Report It

Grade the severity by context more than by the mechanism itself — enumeration against a corporate SSO or a financial platform's login is High to Critical given what it enables downstream, while enumeration on a low-sensitivity public registration flow may only be Low to Medium. Recommend a uniform response — same status code, same generic message, same approximate response size — across all authentication failure paths, and note in the report that this needs to be tested end-to-end after the fix, since it's common for a team to fix the obvious status-code difference while leaving a subtler timing or length difference in place.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
