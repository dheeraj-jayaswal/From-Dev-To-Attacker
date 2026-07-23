# Day 32 — Error Stack Traces: Every Crash Is a Confession

**Severity:** High | **Category:** Security Misconfiguration | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Unhandled exceptions that bubble up to the response without proper error handling reveal file system paths, class and method names, the exact failing SQL query, and sometimes internal hostnames or IPs — all from a single malformed request that the application wasn't built to gracefully absorb.

## 🌱 Why It Exists

Comprehensive error handling — catching exceptions at every layer and returning a generic, safe response — is genuinely tedious to implement thoroughly, and it's exactly the kind of defensive work that gets deprioritized under a feature-delivery deadline, since the happy path works and the unhappy path only shows up when something already went wrong for an actual user, which feels like a lower-priority problem until a tester goes looking for it deliberately.

## ⚔️ How To Find It

```bash
# Force an error with an unexpected type or malformed input
curl -s "https://target.com/api/users/not-an-integer"
curl -X POST https://target.com/api -d "{broken json" -H "Content-Type: application/json"
```

Each framework's stack trace format is recognizable enough to fingerprint quickly — Django reveals full settings, Laravel's Ignition shows `.env` values, raw Node/Express traces reveal internal file paths and often a connection-refused error naming the internal database host directly.

## 💥 How To Exploit It

Treat the stack trace as a set of individually actionable leads rather than one finding: a revealed file path is worth testing against path traversal if any file-access endpoint exists elsewhere in the app; a revealed internal hostname or IP (commonly seen in a database-connection-refused trace) is a direct SSRF target; a revealed framework version feeds straight into a CVE lookup. The stack trace itself doesn't grant access — it tells you exactly where to aim next.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, a malformed request to an order-lookup endpoint returned a raw database exception naming the internal database host and the exact failing query — from that single error, I had the internal hostname needed to attempt an SSRF probe elsewhere in the same application, and it worked. In **education** platform testing, a Java stack trace on a grading-system component revealed the full package structure and internal class names — information that let me predict several related, undocumented endpoints purely from the naming convention the trace exposed.

## 📋 How To Report It

Grade High as a baseline given how consistently this class of finding feeds into a follow-up chain, and document the specific actionable detail in the trace (the internal hostname, the file path, the framework version) rather than just attaching the raw trace — a client's engineering team needs to know exactly what was exposed, not just that an error page existed. Recommend generic error responses in production paired with server-side logging of the full detail for the development team's own use — the fix isn't "stop logging errors," it's "stop showing the log to the requester."

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
