# Day 45 — XSS: Old Bug, Still the Easiest Path to Session Theft

**Severity:** Medium → Critical (depends on what's missing alongside it) | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Cross-Site Scripting happens when user-supplied input is rendered back into a page as executable HTML/JavaScript rather than as inert text — reflected (in the immediate response to a crafted request), stored (persisted and served to other users later), or DOM-based (client-side JavaScript writing user-controlled data into the page without server involvement at all).

## 🌱 Why It Exists

Modern frameworks (React, Vue, Angular) auto-escape output by default, which has genuinely reduced how often this shows up — but it persists in a few specific, predictable places: raw HTML rendering functions used deliberately for rich-text content (`dangerouslySetInnerHTML` in React, `v-html` in Vue), legacy server-rendered pages that predate the framework migration, and any place a developer needed to render actual HTML markup from user input (a rich-text editor's output) and reached for the raw-render escape hatch to make that work.

## ⚔️ How To Find It

```bash
# Basic reflected probe
curl "https://target.com/search?q=<script>alert(document.domain)</script>"
```

Stored XSS deserves more deliberate hunting than reflected — check every field that renders elsewhere in the application (a display name shown to other users, a support-ticket comment visible to staff, a product review), since the injection point and the execution point are often different pages entirely, which makes stored XSS easy to miss if you only test the page where you submitted the payload.

## 💥 How To Exploit It

The real-world impact is almost always session/credential theft — a payload that exfiltrates the victim's session cookie or auth token to an attacker-controlled endpoint, achieving full account takeover the moment a victim views the injected content. Whether that impact is fully realized or significantly contained depends entirely on what security headers are in place (a missing CSP, covered on Day 34, is what turns a found XSS from a partially-mitigated nuisance into complete, unrestricted script execution).

## 🌍 Where This Actually Bites

In an **education** platform's LMS, stored XSS in a student's display name field executed in the context of every teacher and administrator who viewed the class roster — a single malicious enrollment, and every staff member's session was exposed the moment they opened that page. In **retail/ecommerce**, stored XSS in a product review field is a recurring pattern precisely because review content is rendered on high-traffic product pages, meaning a single successful injection reaches every visitor to that product rather than one targeted victim.

## 📋 How To Report It

Grade by exploitation context, not just injection type — reflected XSS requiring a victim to click a crafted link is genuinely less severe than stored XSS that fires automatically for every visitor to a page, and the report should reflect that distinction rather than treating all XSS uniformly. Demonstrate concrete impact (session-cookie exfiltration to a controlled endpoint) rather than a bare `alert()` popup — the latter is a common finding pattern but understates severity to a client who hasn't seen what the bug actually enables. Recommend context-aware output encoding as the fix, and flag any use of raw-HTML-render functions on user-influenced data as needing a dedicated review, since blocklisting specific tags/patterns is reliably incomplete.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
