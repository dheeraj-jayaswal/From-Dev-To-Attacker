# Day 39 — 2FA/MFA Bypass: The Last Line of Defense, Skipped

**Severity:** Critical | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

MFA is meant to be the control that stops an attacker even when they have valid credentials. A bypass means the second factor exists in the login flow's happy path but isn't actually enforced server-side — reachable protected endpoints directly after step one, a brute-forceable OTP with no rate limit, or a response the client can silently manipulate to claim success.

## 🌱 Why It Exists

The most common variant — reaching protected pages by skipping the 2FA step entirely — happens because the session cookie is issued after step one (username/password) for convenience, and the application then trusts that cookie for all subsequent requests without a separate server-side flag confirming the second factor was actually completed. The developer's mental model is "user will naturally go to the 2FA page next," which holds for a normal browser flow but not for a direct API request that skips the UI navigation entirely.

## ⚔️ How To Find It

```bash
# Complete only step 1, then attempt a protected endpoint directly
curl -s https://target.com/dashboard -H "Cookie: session=$STEP1_SESSION"
```

If that returns full authenticated content without ever submitting an OTP, 2FA is decorative rather than enforced. Separately, test the OTP endpoint's own rate limiting in isolation (a 6-digit code with no throttling is brute-forceable well within its validity window), and check whether intercepting and modifying the verification response (`"success": false` → `true`) is trusted client-side rather than re-verified server-side.

## 💥 How To Exploit It

Any one of these bypasses fully defeats MFA on its own — there's no need to chain them. The direct-access variant is the fastest to confirm and the most damning to report, since it demonstrates the second factor was never actually enforced at any point in the request lifecycle, only presented as a UI step.

## 🌍 Where This Actually Bites

In a **banking**-adjacent platform, the session cookie issued after password entry granted full account access to every endpoint I tested, entirely without ever submitting the OTP — MFA was fully enabled on the account and fully bypassable in practice. In **retail/ecommerce** account-recovery flows, I've found backup/recovery codes reusable rather than single-use, meaning a single leaked backup code (from a support ticket, a screenshot, a paste-site leak) provided standing, repeatable MFA bypass rather than a one-time exception.

## 📋 How To Report It

Grade Critical without qualification — MFA bypass directly nullifies the platform's strongest authentication control and typically qualifies for the highest severity tier in almost any bug bounty or pentest scoring framework. Recommend the structural fix rather than a patch: issue a limited pre-auth session after step one that only permits access to the 2FA verification endpoint itself, and upgrade to a fully authenticated session only after the server independently confirms OTP success — never trust a client-supplied "verified" flag for this decision.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
