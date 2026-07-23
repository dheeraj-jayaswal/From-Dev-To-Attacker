# Day 27 — Framework & CMS Version Disclosure: Six Places the Version Hides

**Severity:** Medium | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Beyond raw HTTP headers, the application framework or CMS version leaks through generator meta tags, versioned asset query strings (`?ver=5.8.3`), cookie naming conventions (`laravel_session`, `ci_session`), and error pages. CMSes especially — WordPress, Drupal, Joomla — have disproportionately large CVE databases, which makes a confirmed version number far more consequential here than for a generic web server.

## 🌱 Why It Exists

CMS platforms add version metadata deliberately, originally to help with debugging and theme/plugin compatibility checks — WordPress's `?ver=` query strings on every asset exist specifically so browsers cache-bust correctly across version updates, not to help attackers, but the side effect is identical either way. Removing these markers requires an explicit hardening step most teams never prioritize because it doesn't feel like "real" security work.

## ⚔️ How To Find It

```bash
whatweb https://target.com -v
curl -s https://target.com | grep -oE 'ver=[0-9.]+' | sort -u
```

`whatweb` in verbose mode is the fastest single check — it aggregates generator tags, cookie fingerprints, and common asset patterns in one pass rather than requiring six separate manual checks.

## 💥 How To Exploit It

Once the exact version is confirmed, run it against the platform-specific vulnerability database rather than a generic CVE search — `wpscan` against a confirmed WordPress version checks plugins and themes too, which is usually where the actual exploitable CVE lives rather than WordPress core itself. Drupal's `CHANGELOG.txt`, when reachable, gives the version in plain text with zero fingerprinting effort required.

## 🌍 Where This Actually Bites

In an **education** sector engagement running a WordPress-based public information site, version disclosure via asset query strings confirmed a version with a known plugin vulnerability — the WordPress core itself was current, but a specific plugin bundled with that version had a public SQL injection CVE that the client's patching process, focused on core updates, had missed entirely. This is a recurring theme with CMS platforms specifically: core version hygiene is usually good, plugin/theme hygiene is where the actual exploitable gap lives, and confirming the core version is often just step one toward enumerating the plugin list.

## 📋 How To Report It

Grade Medium for the disclosure itself, and treat a confirmed CMS version as a trigger to specifically check plugin/theme/extension versions rather than stopping at core — the core CVE is rarely where the real risk sits. When reporting, separate the "version is disclosed" finding (low-effort remediation: strip generator tags, remove version query strings) from any specific exploitable CVE you confirm through follow-up testing, since those two things have very different remediation urgency.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
