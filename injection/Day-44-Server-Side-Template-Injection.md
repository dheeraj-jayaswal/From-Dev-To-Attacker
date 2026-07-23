# Day 44 — Server-Side Template Injection: When the Template Engine Trusts the Wrong Input

**Severity:** Critical | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Template engines (Jinja2, Twig, Freemarker, Velocity) are designed to execute template syntax and render the result — that's the entire point. SSTI happens when user-supplied input gets passed into the template rendering step itself rather than being inserted as inert data into an already-defined template, letting an attacker submit template syntax that the engine dutifully executes, frequently escalating to arbitrary code execution depending on the engine.

## 🌱 Why It Exists

This typically comes from a feature that needs some degree of dynamic templating driven by user input — a customizable email template, a report generator letting users define their own formatting, a "preview" feature rendering user-submitted content. The distinction between "render this user content as a value inside a fixed template" and "let user content define part of the template itself" is subtle in code and easy to get wrong, especially when a template engine's convenience functions make both approaches look similarly simple to implement.

## ⚔️ How To Find It

```bash
# Universal probe — if math evaluates, template injection is confirmed
curl "https://target.com/api/render?name={{7*7}}"
# Response containing "49" instead of the literal string confirms SSTI
```

Identify the specific engine before attempting exploitation — Jinja2, Twig, and Freemarker each have different syntax and different paths to code execution, and a payload built for the wrong engine will simply fail to trigger, potentially leading you to (incorrectly) rule out SSTI entirely.

## 💥 How To Exploit It

Once the engine is confirmed, the escalation path from "template expression executes" to "arbitrary code execution" is engine-specific but well-documented — Jinja2's object introspection chain (`{{ ''.__class__.__mro__[1].__subclasses__() }}`) reaching a subprocess-capable class is the canonical example. Treat the `7*7` confirmation as step one only; a report that stops there without attempting the RCE escalation understates the actual severity.

## 🌍 Where This Actually Bites

In a **banking**-adjacent client's customer-communication platform, a "preview your notification template" feature for internal staff passed user input directly into Jinja2's render function rather than treating it as a value — full RCE on the notification service, which had network access to several other internal systems given its role sending transactional messages. In **freight logistics**, a shipment-label customization feature let business customers define their own label templates — a legitimate need for flexibility, implemented by rendering the customer-supplied template string directly rather than restricting them to a safe subset of formatting tags.

## 📋 How To Report It

Grade Critical and push the proof of concept through to actual code execution rather than stopping at the `{{7*7}}` confirmation — the difference between "template injection confirmed" and "remote code execution demonstrated" matters enormously for how a client prioritizes the fix. Recommend a sandboxed or logic-less templating approach for any user-influenced template content (Jinja2's sandboxed environment, or better, a restricted templating language that has no path to arbitrary code execution by design) rather than attempting to blocklist dangerous syntax, which is reliably incomplete against these engines' introspection capabilities.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
