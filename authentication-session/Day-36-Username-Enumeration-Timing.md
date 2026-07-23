# Day 36 — Username Enumeration via Timing: When the Response Looks Identical but the Clock Doesn't Lie

**Severity:** Critical (when it bypasses an otherwise-hardened response) | **Category:** Authentication & Session | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Even when a login endpoint returns identical status codes and body text for every authentication failure — a genuinely hardened response — the underlying processing time often still differs measurably between valid and invalid usernames, because password hashing algorithms like bcrypt only run when a matching user record is actually found. An invalid username short-circuits before that expensive computation; a valid one doesn't.

## 🌱 Why It Exists

This is a logic-ordering issue rather than a deliberate flaw: the natural way to write the authentication check is "look up the user, if found compare the password hash" — which means the expensive bcrypt comparison is conditionally skipped for non-existent users. Nobody sets out to create a timing side-channel; it falls out naturally from writing the most obvious, efficient version of the lookup logic.

## ⚔️ How To Find It

This only matters as a distinct technique when the response-based approach comes up empty — identical status, identical body, identical length across valid and invalid usernames. At that point, measure response time directly, ideally with a tool built for precision timing (Burp's Turbo Intruder with a single concurrent connection) rather than a naive sequential script, since network jitter easily swamps a 50-300ms signal if measurement isn't careful.

```python
# Median of several samples per username, single-threaded for accuracy
baseline = measure_latency("definitely-fake-user")
for user in wordlist:
    if measure_latency(user) - baseline > threshold:
        flag_as_candidate(user)
```

## 💥 How To Exploit It

The same downstream value as response-based enumeration applies — a confirmed username list feeding a targeted password-spray or credential-stuffing attempt — but this variant specifically matters because it bypasses a control the team already believed was solved. Reporting it demonstrates that generic error messages alone are insufficient without also addressing timing, which is a meaningfully more sophisticated finding than the basic response-difference version.

## 🌍 Where This Actually Bites

In a **banking**-adjacent identity platform that had explicitly hardened its login error messages to a uniform response after a prior finding, timing measurement still reliably distinguished valid from invalid usernames — the fix addressed the visible symptom without addressing the underlying computational difference, meaning the "closed" finding was still fully exploitable through a different measurement.

## 📋 How To Report It

Grade this Critical specifically when it defeats a control the client already believes is in place — call that out explicitly in the report, since it's the strongest argument for prioritizing the fix over a competing backlog item. The correct remediation is computing a dummy hash comparison against a fixed placeholder value even when no user is found, so the computational cost — and therefore the timing — is constant regardless of username validity; recommend this specific fix rather than a generic "add rate limiting," since rate limiting slows enumeration but doesn't eliminate the underlying timing signal.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
