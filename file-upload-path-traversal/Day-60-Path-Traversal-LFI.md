# Day 60 — Path Traversal & LFI: When "Read This File" Trusts the Filename Too Much

**Severity:** Critical | **Category:** File Upload & Path Traversal | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Path traversal happens when a filename or path parameter is passed to a file-read function without validating that it stays within an intended directory — `../../` sequences let an attacker walk out of the expected folder and read arbitrary files elsewhere on the file system. Local File Inclusion is the more severe cousin, common in PHP applications, where the same traversal can reach files that get *executed* rather than just read, frequently escalating straight to remote code execution.

## 🌱 Why It Exists

Any feature that takes a filename as a parameter and constructs a file path from it — a document viewer, a template selector, a log-file downloader — creates this risk the moment the parameter isn't validated against an expected allow-list or properly canonicalized before use. The convenient implementation (`open(base_dir + user_filename)`) works for every legitimate filename a normal user would ever submit, and only breaks when someone deliberately submits `../../../etc/passwd` instead.

## ⚔️ How To Find It

```bash
curl "https://target.com/api/download?file=../../../../etc/passwd"
curl "https://target.com/api/view?page=../../../var/log/apache2/access.log"
```

Test both raw traversal sequences and URL-encoded variants (`%2e%2e%2f`), since some filters catch the literal `../` string but not its encoded form. For PHP specifically, test whether the include mechanism can be chained with log poisoning — injecting PHP code into a log file (via a crafted User-Agent header, for instance) that's then included and executed via the traversal.

## 💥 How To Exploit It

Reading `/etc/passwd` or an application config file confirms the traversal; the higher-value target is usually the application's own configuration or environment file, which frequently contains database credentials directly. Where LFI applies (PHP `include`/`require` on a traversal-controlled path), the log-poisoning chain converts a file-read primitive into full code execution — genuinely one of the more elegant escalation chains in web exploitation.

## 🌍 Where This Actually Bites

In an **income tax** platform's document-preview feature, a `file` parameter used to display previously-uploaded tax documents accepted traversal sequences unfiltered — reading arbitrary files on the server including the application's own database configuration, which held credentials for the full taxpayer records database. In **freight logistics**, a "download shipment manifest" feature constructed its file path directly from a user-supplied filename parameter — traversal there reached internal configuration files revealing the credentials used for the partner-integration database connection.

## 📋 How To Report It

Grade Critical for confirmed read access to application configuration or credential files, and Critical without qualification for any confirmed LFI-to-RCE chain. Recommend canonicalizing the resolved path and verifying it remains within the intended base directory before any file operation (rather than attempting to filter `../` sequences, which is reliably incomplete against encoding variants), and where possible, mapping user-supplied identifiers to an internal allow-list of permitted files rather than constructing a file system path from user input at all.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
