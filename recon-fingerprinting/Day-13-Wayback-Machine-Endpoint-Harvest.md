# Day 13 — Wayback Machine Endpoint Harvest: Mining the Internet's Memory

**Severity:** Medium | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

The Internet Archive's Wayback Machine has been crawling and storing snapshots of the web since 1996 — including old API routes, admin panels, config files, and backup files that a company removed from their current site years ago but that still exist in archived crawl history. For recon purposes, it's a time machine into a target's past deployments, and the past is almost always less secure than the present.

## 🌱 Why It Exists

The gap here is procedural rather than technical: when a team removes a feature, they typically remove the frontend UI element and the documentation, but the backend route handling that request often keeps working, because nobody's job is explicitly "go delete the now-unused endpoint." The server doesn't know the feature was "removed" — it just stops receiving requests for a route that, technically, is still there and still responding, built to whatever security standard existed at the time.

## ⚔️ How To Find It

```bash
# Multi-source harvest (Wayback + Common Crawl + OTX + URLScan)
gau target.com --subs --threads 10 -o all_urls.txt

# Filter for what actually matters
grep -iE "\.(bak|env|sql|config|log|old)$" all_urls.txt        # backup/config files
grep -iE "/api/v[0-9]+" all_urls.txt | sort -u                # old API versions
grep -iE "admin|debug|internal|staging" all_urls.txt | sort -u # sensitive paths

# Check which of the interesting ones still respond
cat interesting_urls.txt | httpx -silent -status-code | grep -v " 404 "
```

The filtering step matters more than the harvesting step — a mature target can return millions of archived URLs, and the signal is in a handful of specific patterns, not the raw volume.

## 💥 How To Exploit It

The exploitation is simply confirming the archived URL still resolves and behaves the way it did when it was archived. The most consistently valuable finds are backup/config files that leak credentials directly (`.env.bak`, `config.php.old`), and superseded API versions that never had the auth or validation fixes applied to the current version — the frontend moved to `/api/v3/`, but `/api/v1/` never got decommissioned and never got the security patch either.

## 🌍 Where This Actually Bites

In an **income tax** platform engagement, Wayback harvesting surfaced an archived `/api/v1/` route that predated a major authentication overhaul — the current `/api/v3/` correctly enforced ownership checks, but the old `v1` path, still live, returned taxpayer records with no ownership validation at all. In a **banking** context, an archived backup-file reference (`config.php.bak`, snapshotted years earlier) turned out to still be downloadable, containing database credentials that — while rotated since — confirmed a broader pattern of backup files never being cleaned up from the web root, a finding worth flagging as a systemic issue rather than a one-off.

## 📋 How To Report It

Grade by what the live endpoint actually exposes when tested — a live credential-bearing config file is Critical regardless of how old the archive snapshot is; a dead endpoint that 404s today is not reportable at all, just noise to filter out. Frame the remediation as two separate fixes: patch or remove the specific exposed asset, and establish a decommissioning checklist so that removing a feature's frontend also triggers removal of its backend route — otherwise this exact finding recurs with the next deprecated feature.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
