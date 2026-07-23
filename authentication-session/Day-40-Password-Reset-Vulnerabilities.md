# Day 40 — Password Reset Vulnerabilities: The Recovery Flow Is a Second Front Door

**Severity:** High → Critical | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

A password reset token functions as a temporary credential — whoever holds it can set a new password and take over the account. The flow is only as strong as the weakest link in generating, delivering, and invalidating that token: a predictable value, a URL that leaks via the `Referer` header, a Host-header-controlled reset link, or a token that remains valid after it's already been used once.

## 🌱 Why It Exists

Password reset is usually built after the main authentication flow, by developers focused on making it convenient (a working email link, minimal friction) rather than treating it with the same rigor as login itself — it's easy to mentally categorize as a "secondary" flow, even though it grants exactly the same level of access as a correct password.

## ⚔️ How To Find It

```bash
# Host header injection — check whether the reset URL trusts the request's Host
curl -X POST https://target.com/forgot-password -H "Host: attacker.com" -d '{"email":"victim@target.com"}'
```

Beyond Host injection, check token predictability directly (is it timestamp-based, sequential, or a simple hash of the email — all guessable within a practical search space), confirm whether a used token still works on replay, and check how long a token actually remains valid by simply waiting before use.

## 💥 How To Exploit It

Host header injection is the most severe variant when present: the reset email itself gets built using the attacker-controlled Host, meaning the victim receives a legitimate-looking email containing a link to the attacker's domain — capturing the token requires no interaction beyond the victim clicking a link that looks entirely normal. A predictable token is exploitable more directly still — brute-forcing a timestamp-based token within a narrow window requires only a few hundred attempts, not millions.

## 🌍 Where This Actually Bites

In a **banking**-adjacent platform, the password-reset endpoint built its email link using the request's `Host` header without validation — a crafted request with an attacker-controlled Host produced a legitimate reset email pointing to the attacker's domain, with the token captured the moment the victim clicked what looked like a normal reset link. In an **income tax** filing platform, reset tokens turned out to still be valid after already being used once — meaning a token intercepted or logged anywhere (proxy logs, browser history, a shared computer) remained a standing account-takeover vector indefinitely rather than expiring after its intended single use.

## 📋 How To Report It

Grade Critical for Host-header injection and for token reuse after use — both lead directly to full account takeover with minimal attacker effort. Grade High for predictable tokens or tokens with excessive validity windows, since exploitation requires slightly more work (a timing window, a brute-force attempt) but remains straightforward. Recommend building reset URLs exclusively from a server-side configured base URL — never from any client-supplied header — and enforce single-use, short-expiry tokens invalidated immediately upon successful use, which closes the reuse variant at the same time.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
