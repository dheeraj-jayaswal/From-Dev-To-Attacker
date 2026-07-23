# Day 42 — NoSQL Injection: The "We Don't Use SQL, So We're Safe" Myth

**Severity:** High → Critical | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

NoSQL databases like MongoDB accept query objects rather than strings, and when an API passes user input directly into a query object without type-checking, an attacker can submit operators (`$ne`, `$gt`, `$regex`) instead of plain values — turning `{"password": "guessedvalue"}` into `{"password": {"$ne": null}}`, which matches any non-null password and authenticates without knowing the real one.

## 🌱 Why It Exists

The specific misconception that produces this bug is real and common: teams that migrated from SQL genuinely believe injection risk left with the SQL database, since the classic quote-and-semicolon SQLi payload doesn't apply here at all. What actually causes the vulnerability is JSON body parameters being passed straight into a MongoDB query without validating that they're the expected primitive type (a string, not an object) — a check that has to be added deliberately and isn't part of using MongoDB by default.

## ⚔️ How To Find It

```bash
# Login bypass via operator injection in a JSON body
curl -X POST https://target.com/api/login -H "Content-Type: application/json" \
  -d '{"username":"admin","password":{"$ne":null}}'
```

Test the same operator-injection pattern in search and filter parameters, not just login — `$regex` in a search field can be used both to bypass filtering logic and, with a crafted pattern, to cause measurable processing-time differences that leak data character-by-character (a NoSQL equivalent of blind SQLi).

## 💥 How To Exploit It

The login-bypass variant is immediate account compromise with no further work needed. Beyond auth bypass, `$regex`-based blind extraction lets you pull field values you shouldn't have access to one character at a time by measuring which regex patterns match — slower than SQLi's `UNION SELECT`, but just as complete an information-disclosure path once automated.

## 🌍 Where This Actually Bites

In a **freight logistics** platform using MongoDB for its shipment-tracking API, a search endpoint accepted user input directly into a query filter — `$regex` injection there let me extract shipment reference numbers belonging to other customers' accounts, character by character, purely through timing and match behavior. In **education** platform login flows built on Node.js/MongoDB stacks specifically, the `$ne` operator bypass on a login endpoint is one of the more common findings I encounter, precisely because the migration away from SQL databases in that stack often isn't accompanied by an equivalent security review of the new query patterns.

## 📋 How To Report It

Grade Critical for any confirmed authentication bypass via operator injection — it's full account takeover with essentially zero attacker effort once found. Grade High for blind data-extraction variants via `$regex`, since exploitation requires more automation effort even though the eventual data exposure can be just as complete. Recommend strict input-type validation (reject any non-string value where a string is expected) at the API boundary, before the value ever reaches the database query — this closes the entire class of bug regardless of which specific operator an attacker tries next.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
