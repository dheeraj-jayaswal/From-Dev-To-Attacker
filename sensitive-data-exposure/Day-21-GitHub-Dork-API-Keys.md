# Day 21 — GitHub Dorking: Finding Live API Keys in Public Code

**Severity:** Medium (discovery) → Critical (verified live) | **Category:** Sensitive Data Exposure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

GitHub indexes the full text of public repositories, and its search operators (`org:`, `filename:`, `path:`) let you query that index precisely. For recon purposes, this means every accidentally-committed API key, cloud credential, or signing secret across a company's public repos — and their employees' personal repos, if they've ever pushed company code there — is searchable in seconds, entirely passively.

## 🌱 Why It Exists

Local development requires real credentials to test against real services, and the fastest path for a developer under deadline pressure is to paste that key directly into the code they're already editing, intending to move it to an environment variable "before this gets committed." The commit happens anyway — sometimes because `git add .` swept up more than intended, sometimes because the "move to env vars" step just got deprioritized once the feature worked.

## ⚔️ How To Find It

```
org:target-company "AKIA"
org:target-company "sk_live_"
org:target-company filename:.env "SECRET"
```

`truffleHog --only-verified` and `gitleaks detect` are worth running as a standard pair on any repo you clone during an engagement — the `--only-verified` flag on truffleHog specifically matters, since it filters to credentials confirmed live against the actual service rather than pattern-matches that could be expired, rotated, or test-only values.

## 💥 How To Exploit It

Once a key is found, the verification step is a single, narrowly-scoped API call to the relevant service confirming the key is accepted — not broad exploitation of what the key grants access to. That single confirmation is enough evidence for a Critical report; going further than confirming validity risks turning a legitimate finding into an authorization problem of your own.

## 🌍 Where This Actually Bites

In a **freight logistics** context, a dork against a client's GitHub org surfaced a live carrier-integration API key committed to a public utility repo an engineer had pushed independently of the main codebase — the key granted rate-lookup and label-generation access across the client's entire carrier account, well beyond what that specific utility script needed. In **retail/ecommerce**, I've found Stripe test-mode keys committed publicly that turned out, on verification, to actually be live production keys mislabeled by the developer — a reminder that key prefixes and naming conventions in code are not something to trust without direct verification.

## 📋 How To Report It

State clearly whether the key was verified live and exactly how (the specific, minimal API call made) — this distinction matters enormously to how a client prioritizes the fix. Recommend rotation as the first action regardless of severity grading, since a key visible in a public repo should be treated as compromised the moment it's found, independent of whether you've personally verified it works. Note the exposing repository and commit for the client's own tracking, but avoid including the actual secret value in the written report — reference that it's been shared separately or masked.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
