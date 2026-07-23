# Day 31 — Debug Mode in Production: One Config Flag, Total Transparency

**Severity:** High → Critical (framework-dependent) | **Category:** Security Misconfiguration | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Development-mode debug flags (`DEBUG = True` in Django, `APP_DEBUG=true` in Laravel) are built to show developers everything about a failing request — full stack traces, local variable values, the entire settings/config object. Left enabled in production, that same detailed diagnostic output is served to whoever happens to trigger the error, deliberately or otherwise.

## 🌱 Why It Exists

Debug mode is essential during development and is almost always enabled by default in a fresh framework install — the gap is purely a deployment-configuration oversight: the flag that needs to flip from `True` to `False` for production simply doesn't, usually because the environment-specific config file was cloned from a working development setup rather than built fresh for the production environment.

## ⚔️ How To Find It

```bash
curl -s https://target.com/definitely-nonexistent-path | grep -iE "debug|traceback|django|ignition"
```

Framework-specific debug pages have recognizable fingerprints (Django's yellow error page, Laravel's Ignition UI, Flask's Werkzeug console) — a single malformed or nonexistent request is usually enough to trigger one if debug mode is active anywhere in the stack.

## 💥 How To Exploit It

The severity swings enormously by framework. Django and Laravel debug pages leak configuration values directly — `SECRET_KEY`, database credentials, mail server passwords — which is a severe information-disclosure finding but requires a follow-up step (forging a session, connecting to the database) to become full compromise. Flask's Werkzeug debugger is categorically worse: it exposes an **interactive Python console** directly in the browser with no separate exploitation step required — remote code execution the moment the console is reachable.

## 🌍 Where This Actually Bites

In a **banking**-adjacent engagement, a Django-based internal tool left in debug mode exposed the full `settings.py` on any malformed request, including a `SECRET_KEY` that — once confirmed — allowed forging valid session cookies for any user without ever touching a login form. In an **income tax** platform's internal reporting tool, a Flask service still running with debug mode enabled exposed the interactive console directly — a single confirmed request there is full RCE on that host, no chaining required at all.

## 📋 How To Report It

Grade High as a baseline for any framework leaking configuration/credentials via debug output, and escalate immediately to Critical if the exposed debug tooling includes any form of code execution (Flask's Werkzeug console is the clearest example — treat its mere presence as Critical, since exploitation is trivial and immediate). The fix is nearly always a single environment-config line, which is worth stating explicitly in the report — this is a fast, high-confidence remediation for the client to action, not a structural rebuild.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
