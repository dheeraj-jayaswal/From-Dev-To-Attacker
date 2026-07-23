# Day 59 — Unrestricted File Upload: The Feature That Lets You Put Your Own Code on Their Server

**Severity:** Critical | **Category:** File Upload & Path Traversal | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Any upload feature that doesn't rigorously validate what's actually being uploaded — checking only the file extension, or only a client-supplied Content-Type header, rather than the file's actual content — can be tricked into accepting an executable script disguised as an allowed file type, and if that uploaded file lands somewhere the web server will execute it, the result is remote code execution.

## 🌱 Why It Exists

Validating "is this really a JPEG" is more work than checking the filename extension or the Content-Type header the browser sends — both of which are entirely client-controlled and trivially spoofed. The convenient, fast-to-implement check (extension or header) works correctly for every legitimate user, since browsers set these values accurately for real uploads; it only fails against someone deliberately lying about what they're sending.

## ⚔️ How To Find It

```bash
# Rename a PHP web shell with an allowed extension, or double-extension trick
curl -F "file=@shell.php.jpg" https://target.com/api/upload
curl -F "file=@shell.phtml" https://target.com/api/upload  # alternate executable extension
```

Test extension checks, Content-Type header manipulation, and magic-byte checks independently — some applications validate one but not the others, and a bypass frequently requires combining a technique for each layer actually present (a correct image magic-byte header, followed by PHP code appended after it, with a `.php` extension smuggled past an extension filter via double-extension or null-byte tricks).

## 💥 How To Exploit It

Once a web shell is uploaded and its storage location is known or guessable, requesting that file directly executes it server-side — from there, standard web-shell capabilities apply: file system browsing, further payload staging, and typically a path to a full reverse shell for interactive access.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, a "upload your product return photo" feature validated only the file extension — a `.php` file renamed with a `.jpg.php` double extension bypassed the check, and the upload directory was directly web-accessible, giving full RCE on the application server through a feature meant purely for customer service documentation. In an **education** platform's assignment-submission feature, file-type validation checked only the client-supplied Content-Type header — trivially overridden in the upload request — and combined with a predictable storage path, this gave a path to executing arbitrary code on a server also hosting grading and student-record systems.

## 📋 How To Report It

Grade Critical whenever a web shell upload succeeds and executes — this is remote code execution and needs no further chaining to justify severity. Recommend validating file content via magic-byte/MIME detection rather than trusting extension or Content-Type header, storing uploads outside the web root (or in object storage with no execute permission) so that even a successfully uploaded malicious file can never be directly requested and executed, and applying an allow-list of genuinely expected file types rather than a deny-list of dangerous ones.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
