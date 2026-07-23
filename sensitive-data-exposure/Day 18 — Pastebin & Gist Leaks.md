# Day 18 — Pastebin & Gist Leaks: Secrets Shared for Help, Indexed Forever

**Severity:** Medium → Critical (depends on what's live) | **Category:** Sensitive Data Exposure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Paste sites (Pastebin, GitHub Gist, GitLab Snippets) exist so developers can quickly share a code snippet or config file with a colleague. The problem is that public pastes are indexed by search engines and stay online indefinitely — long after the "quick share" purpose has been served, long after the person who posted it has forgotten it exists.

## 🌱 Why It Exists

This is a human workflow problem, not a technical one. Someone hits a bug late at night, needs a second pair of eyes fast, and pastes a config file or error log to share over Slack or Discord — with the actual credentials still in it, because removing them first felt like an unnecessary extra step under time pressure. The paste does its job in five minutes and is then simply never revisited or deleted.

## ⚔️ How To Find It

```bash
site:pastebin.com "target.com" "password"
site:gist.github.com "target.com" "api_key"
"@target.com" site:pastebin.com
```

For ongoing coverage rather than a one-time check, a monitoring tool like PasteHunter watching for your target's domain name is worth setting up before an engagement even starts — leaks discovered mid-engagement rather than pre-planned are common enough that a standing watch pays for itself.

## 💥 How To Exploit It

Archive the paste immediately with a screenshot and a raw-content download — paste sites get taken down quickly once discovered, sometimes within hours of a report, so your evidence needs to exist independently of the live URL. Verify credentials minimally and only within scope (a single authenticated API call to confirm a key is live, not broad exploitation) before writing up the finding.

## 🌍 Where This Actually Bites

In a **freight logistics** engagement, a developer had pasted a full API integration snippet to Gist while asking a public forum for debugging help — complete with the partner API key for a live carrier-rate integration, discoverable months later with a two-word search. In **income tax** and **banking** contexts specifically, I treat any employee-identifiable paste (an `@company.com` email address alongside a credential, even an old or rotated one) as worth flagging on its own: it confirms a pattern of casual credential sharing that's worth raising as a process issue with the client, independent of whether that specific secret is still live.

## 📋 How To Report It

Grade by whether the exposed credential is still live — a rotated key found in an old paste is Medium (process finding, evidence of a recurring habit); a verified-live database password or API key is Critical. Mask the actual secret value in your written report even though you've confirmed it, include the paste URL and an archived screenshot as evidence, and always state clearly whether you verified liveness and how, since an unverified claim ("this might still work") carries far less weight with a client than a confirmed one.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
