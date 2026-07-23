# Day 14 — Directory & File Bruteforce: Removing the Link Doesn't Remove the Route

**Severity:** Medium | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Directory and file bruteforcing sends requests for large lists of common path and filename candidates against a target, then watches which ones return a real response instead of a 404. It works because removing a page from a site's navigation only removes the *link* — the server-side route or file usually keeps working until someone explicitly removes or blocks it.

## 🌱 Why It Exists

From the developer side, this is a "removed from the UI, forgot the backend" gap — the exact same root cause behind old API versions and orphaned admin panels. It also shows up structurally: default framework paths (`/actuator`, `/wp-admin`, `/adminer.php`) exist because the framework put them there, and unless a team explicitly hardens or removes them post-deployment, they're reachable by definition.

## ⚔️ How To Find It

```bash
# Calibrate first — know what a genuine 404 looks like
curl -s https://target.com/definitely-not-a-real-page | wc -c

# Then fuzz with that baseline filtered out
ffuf -u https://target.com/FUZZ -w seclists/common.txt -mc 200,301,302,401,403 -fs <baseline_size>
```

Prioritize a short, high-value manual list before running a full wordlist sweep: `.git/config`, `.env`, `/actuator/env`, `/swagger.json`, backup extensions on any known filename. These consistently pay off faster than a generic 20,000-word list.

## 💥 How To Exploit It

An exposed `.git/` directory is the highest-value single find here — `git-dumper` reconstructs the full repository, and `git log`/`git show` across history frequently surfaces credentials that were later "removed" from the current code but never purged from commit history. Exposed admin panels and database managers (`/adminer.php`, `/phpMyAdmin`) are worth a quick default-credential check before moving on, since default creds on forgotten tooling is a more common finding than it should be.

## 🌍 Where This Actually Bites

In **retail/ecommerce** platforms running a mix of legacy PHP alongside a newer stack, directory bruteforce routinely turns up leftover admin tooling from the previous platform generation — still live, still reachable, security posture frozen at whatever standard applied years ago. In **education** sector LMS platforms, I've found `.git/` directories exposed on subdomains that were clearly spun up for a specific integration project and handed off between vendors, where nobody owned the responsibility of hardening the deployment afterward.

## 📋 How To Report It

Grade based on what's actually reachable and what it exposes — a `.git/` directory yielding full source code and historical credentials is Critical; an exposed but empty admin login page with no default creds is Low/Informational. Always include the calibration detail in your report methodology (the baseline 404 size you filtered against), since it demonstrates the finding is a genuine positive and not an artifact of a misconfigured scan.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
