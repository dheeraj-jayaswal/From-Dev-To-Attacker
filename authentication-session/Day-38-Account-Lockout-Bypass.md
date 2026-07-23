# Day 38 — Account Lockout Bypass: A Broken Lock Is Worse Than No Lock

**Severity:** Critical | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

A lockout mechanism exists — the application genuinely believes it's protected against brute force — but the specific way it's implemented can be sidestepped: rotating a spoofable IP header, racing multiple simultaneous requests past the counter, or exploiting a separate, weaker lockout policy on an adjacent endpoint like OTP verification. The control isn't absent; it's circumventable, which is arguably worse because it creates false confidence that no further protection is needed.

## 🌱 Why It Exists

Each bypass variant traces back to a specific, understandable implementation shortcut: tracking lockout by IP address trusts `X-Forwarded-For`, a header the client fully controls, because validating the "real" client IP correctly behind proxies and CDNs is a genuinely fiddly problem that gets solved with the simplest available header rather than the most secure one. Race-condition bypasses exist because the natural way to write "check counter, then increment" isn't atomic by default, and making it atomic requires deliberate extra work most teams don't know is necessary.

## ⚔️ How To Find It

```bash
# Header spoofing — rotate X-Forwarded-For per attempt
for i in $(seq 1 50); do
  curl -s -X POST https://target.com/login \
    -H "X-Forwarded-For: 10.0.$((i/255)).$((i%255))" \
    -d "username=admin&password=guess$i"
done
```

Test the race-condition variant with Turbo Intruder configured for high concurrency and a single-packet burst — if the lockout counter isn't atomic, a burst of simultaneous requests can slip past the check before any of them increment it. Separately, always test the OTP/2FA verification endpoint's own lockout policy in isolation — it's frequently weaker than the main login's, since it was built by a different feature team at a different time.

## 💥 How To Exploit It

Once any bypass is confirmed, the lockout is functionally absent — proceed exactly as with an unprotected login endpoint, since the control that would have stopped a brute-force or credential-stuffing attempt no longer applies.

## 🌍 Where This Actually Bites

In a **banking**-adjacent platform, the main login enforced a solid 5-attempt lockout, but rotating `X-Forwarded-For` per request bypassed it completely — the lockout logic trusted the header for per-IP tracking, and it took under a second per rotated value to send effectively unlimited attempts. In **education** sector platforms, I've found the primary login well-protected while the password-reset OTP endpoint — built later, by a different part of the team — had no rate limiting at all on a 6-digit code, meaning the "protected" account was still fully brute-forceable through the side door of account recovery.

## 📋 How To Report It

Grade Critical for any confirmed bypass — a header-spoofing or race-condition bypass of an otherwise-real lockout mechanism deserves the same severity as no lockout at all, since the practical effect on an attacker's capability is identical. In the report, be explicit that the client should test every bypass class (header spoofing, race condition, and the separate OTP/reset endpoint) rather than fixing only the one you demonstrated — these bypasses are commonly present as a set, not in isolation, because they share the same root cause of an incompletely-designed lockout mechanism.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
