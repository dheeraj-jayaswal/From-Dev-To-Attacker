# Day 22 — Hardcoded Credentials in Git History: Deleted From Code, Not From Git

**Severity:** Medium (discovery) → Critical (verified live) | **Category:** Sensitive Data Exposure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

This is distinct from a currently-visible leaked key: it's a credential that was hardcoded into a file, committed, and later deleted from the *current* version of that file — but git preserves every historical version of every committed file forever. `git log --all -p` reads that entire history, meaning "we removed it" is not the same thing as "it's gone."

## 🌱 Why It Exists

This is the single most common security misunderstanding I encounter among developers who are otherwise strong engineers: the mental model is that deleting a line and committing that deletion removes the information, the same way deleting a line from a live document removes it. Git doesn't work that way — every commit is a permanent, retrievable snapshot, and the "fix" commit sits right alongside the "introduce the secret" commit in the same retrievable history.

## ⚔️ How To Find It

```bash
git clone https://github.com/target/repo && cd repo
git log --all -p | grep -iE "(password|secret|api_key|DB_PASS)\s*[=:]\s*['\"]"

# Specifically check whether a .env file was ever committed, even if it's since been gitignored
git log --all -- .env
```

CI/CD configuration files (`Jenkinsfile`, `.gitlab-ci.yml`, `docker-compose.yml`) are worth checking as a distinct category — these frequently carry database connection strings and service credentials directly in plaintext, since they're often treated as "infrastructure config" rather than "code that needs the same credential hygiene."

## 💥 How To Exploit It

The exploitation step is verification, not further compromise — connect (read-only, minimally) to confirm a database or service credential found in history is still valid, or check whether it's been rotated since. A credential found in a two-year-old commit that's clearly been rotated is still worth reporting as a process finding, even with zero current exploitability, because it demonstrates the same commit-hygiene gap that produced it will produce the next one too.

## 🌍 Where This Actually Bites

In a **banking**-adjacent engagement, git history on an internal tooling repo revealed a database connection string with a live password — the current `config.py` had long since moved to environment variables, but the string sat unchanged in a commit from over a year earlier, and testing confirmed the password itself had never actually been rotated despite the code change. In **income tax** platform tooling, I've found SMTP credentials for the platform's notification service committed in an early setup commit and never rotated after being "removed" from the visible code — meaning years later, the mail-sending account was still accessible to anyone who ran `git log --all` against that repo.

## 📋 How To Report It

Distinguish clearly in the write-up between "credential found in history, confirmed still live" (Critical, immediate rotation required) and "credential found in history, confirmed already rotated" (Medium — a process finding about commit hygiene, not an active compromise). Recommend the client treat this as a standing practice rather than a one-time cleanup: any repository that has ever had a real secret committed to it should have that secret rotated permanently, since a `git filter-branch` or repository rewrite to scrub history is rarely complete and forks/local clones may still retain the original commits regardless.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
