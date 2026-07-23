# Day 19 — Backup File Discovery: What Editors and Deployment Scripts Leave Behind

**Severity:** Low (as a technique) → Critical (based on contents) | **Category:** Sensitive Data Exposure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Backup file discovery targets the accidental duplicates that text editors, manual developer habits, and deployment tooling leave sitting in the same web-accessible directory as the original file — `config.php.bak`, `wp-config.php~`, a stray `database.sql` dump. The web server has no concept of "this is a backup, don't serve it" — it serves whatever file exists at whatever path is requested, extension and all.

## 🌱 Why It Exists

Editors like vim and nano auto-create backup/swap files as a safety net during editing — a genuinely useful feature that becomes a liability the moment editing happens directly on a production server instead of in a separate environment. Manual habits compound it: a developer running `cp config.php config.php.bak` before a risky change, fully intending to delete the backup afterward, and simply forgetting once the change works and attention moves elsewhere.

## ⚔️ How To Find It

```bash
# Test backup extensions against every known config/settings filename
for ext in .bak .old .orig .backup ~ .swp; do
  curl -s -o /dev/null -w "%{http_code} config.php$ext\n" "https://target.com/config.php$ext"
done
```

Tailor the filename list to the stack you're testing — `wp-config.php` variants for WordPress, `settings.py` variants for Django, `application.properties` variants for Spring Boot — since generic wordlists waste time on extensions that don't apply to the target's actual technology.

## 💥 How To Exploit It

A `.git/` directory left exposed is the highest-value variant of this category — `git-dumper` reconstructs the entire repository, and checking history (`git log --all -p`) frequently surfaces credentials that were removed from the *current* code but never purged from any earlier commit. For a straightforward config backup, the exploitation is just reading the file — no chaining required, since the credentials are sitting there in plaintext.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, a `config.php.bak` file left over from a manual hotfix contained the production database password unchanged from the live `config.php` — meaning the "backup" was functionally a live credential dump sitting one file-extension guess away from the real config. In **education** sector platforms running on shared hosting, I've found `.sql` database dumps left in a web-accessible `/backup/` directory from an automated nightly export job — nobody had configured it to write outside the web root, so every night's dump was quietly re-exposed.

## 📋 How To Report It

The technique itself is Low severity; grade the actual finding by what's inside the file, not by the fact that a backup exists — a `.bak` of a static HTML page is a non-issue, while a `.env.backup` with live database and API credentials is Critical. Always note in the remediation that the fix needs two parts: remove the specific exposed file, and address the underlying habit or process (editing directly on production, deployment scripts writing dumps into the web root) that will otherwise produce the same finding again on the next config change.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
