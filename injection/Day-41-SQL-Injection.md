# Day 41 — SQL Injection: The Oldest Bug That Still Ends Careers

**Severity:** Critical | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

SQL injection happens when user-controlled input is concatenated directly into a database query instead of being passed as a parameter, letting an attacker alter the query's actual logic — turning a `WHERE id = 1` lookup into a query that returns every row, every table, or in the worst case, executes commands on the database server itself.

## 🌱 Why It Exists

This survives in 2026 codebases for a specific reason: it's rarely a greenfield mistake anymore — parameterized queries are the default in every modern ORM. It shows up in legacy code predating ORM adoption, in raw-query "escape hatches" developers reach for when the ORM can't easily express a complex query, and in stored procedures where the same concatenation habit gets replicated inside the database layer itself, outside the application code a security review would normally focus on.

## ⚔️ How To Find It

```bash
# Confirm with a single quote, then automate with sqlmap
curl "https://target.com/api/product?id=1'"
sqlmap -u "https://target.com/api/product?id=1" --batch --level=3 --risk=2
```

Blind and time-based variants are worth testing explicitly even when the error-based probe shows nothing — a query that never reflects an error message can still be exploited by observing response timing (`id=1 AND SLEEP(5)`) or boolean differences (`id=1 AND 1=1` vs `id=1 AND 1=2`).

## 💥 How To Exploit It

Beyond straightforward data exfiltration via `UNION SELECT`, check whether the database account has file-read/write privileges (`LOAD_FILE`, `INTO OUTFILE` in MySQL) or command-execution features (`xp_cmdshell` in MSSQL) — these turn a data-disclosure bug into full server compromise, and it's a mistake to stop investigating the moment data extraction succeeds.

## 🌍 Where This Actually Bites

In an **income tax** filing platform, a legacy raw-query search feature (predating the rest of the application's ORM migration) was vulnerable to blind SQLi on a taxpayer-ID lookup — full database extraction would have meant every filed return, PAN number, and bank account on record. In **retail/ecommerce**, I've found SQLi in a "sort by" query parameter specifically, because developers frequently treat sort/filter parameters as safe since they're not "real" user input in the traditional sense — they come from a dropdown, not a text field — and skip parameterizing them for that reason alone.

## 📋 How To Report It

Grade Critical whenever confirmed, and demonstrate impact concretely — a `UNION SELECT` pulling the database version and current user is a solid PoC, but pulling a small, redacted sample of real records (with client authorization) is what actually conveys severity to a non-technical stakeholder. Recommend parameterized queries/prepared statements as the only real fix — input sanitization and blocklisting are not durable substitutes, and the report should say so explicitly to prevent the client from shipping an incomplete fix.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
