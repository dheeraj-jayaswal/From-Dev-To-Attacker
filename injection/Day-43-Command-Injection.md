# Day 43 — OS Command Injection: When the App Was Built to Call the Shell

**Severity:** Critical | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Command injection occurs when an application passes user-controlled input into a system shell call — often through a convenience wrapper around a command-line tool (image conversion, PDF generation, network diagnostics) — without properly escaping shell metacharacters, letting an attacker append their own commands to the one the developer intended to run.

## 🌱 Why It Exists

This almost always traces back to a legitimate feature that needed to shell out to an existing command-line tool rather than reimplementing its functionality in the application's own language — calling `ffmpeg`, `imagemagick`, or a network utility directly is often the fastest, most reliable way to get a feature working, and building the shell command via string concatenation with user input feels like the natural, quick way to wire it up.

## ⚔️ How To Find It

```bash
# Test shell metacharacters in any parameter that plausibly reaches a system call
curl "https://target.com/api/ping?host=127.0.0.1;id"
curl "https://target.com/api/convert?filename=test.jpg%20%26%26%20whoami"
```

Features that plausibly shell out — file conversion, network diagnostic tools (ping/traceroute utilities exposed as a feature), image/PDF processing, and anything involving a filename passed to an external tool — deserve specific, deliberate testing even when there's no obvious sign of shell usage from the outside.

## 💥 How To Exploit It

Once a metacharacter (`;`, `&&`, `|`, backticks) successfully executes an appended command, escalate from confirmation (`id`, `whoami`) to establishing a reverse shell for full interactive access — command injection is one of the few web vulnerability classes that converts directly to complete server compromise without needing to chain through any additional bug.

## 🌍 Where This Actually Bites

In an **education** sector platform offering a "test your connection" diagnostic tool for students taking online exams, the ping-utility feature passed the user-supplied hostname directly into a shell `ping` command — full command injection on a feature built for a completely benign, low-stakes purpose, sitting on the same server as the exam-content and grading systems. In **retail/ecommerce**, a bulk product-image upload feature shelling out to an image-processing tool accepted the original filename unsanitized — a filename crafted with shell metacharacters achieved command execution the moment the batch conversion job ran.

## 📋 How To Report It

Grade Critical without qualification — command injection is remote code execution, full stop, and doesn't need chaining or additional context to justify the severity. Document the exact command executed as proof (a benign confirmation command like `id` or `hostname` is sufficient evidence; avoid anything destructive or persistent). Recommend eliminating shell invocation entirely where possible in favor of language-native libraries, and where shelling out is unavoidable, using parameterized subprocess calls that don't pass through a shell interpreter at all, rather than attempting to sanitize or escape user input into a shell string.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
