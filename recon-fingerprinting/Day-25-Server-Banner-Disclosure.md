# Day 25 — Server Banner Disclosure: The Smallest Leak With the Longest Chain

**Severity:** Low (standalone) → High (if the version has a known exploit) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Most web servers identify themselves in the `Server` HTTP response header by default — `Apache/2.4.49`, `nginx/1.18.0` — down to the exact patch version. On its own this is just information. The moment that version has a known, exploitable CVE, the banner turns a blind attack attempt into a targeted one with a known working exploit already published.

## 🌱 Why It Exists

Web server software ships with self-identifying headers enabled out of the box, and disabling that identification is a deliberate hardening step (`ServerTokens Prod` in Apache, `server_tokens off` in nginx) that a team has to explicitly apply — it's not the default anywhere, and unless hardening is a checklist item at deployment time, it simply never happens.

## ⚔️ How To Find It

```bash
curl -sI https://target.com | grep -iE "server|powered|generator"
```

At scale, `httpx -tech-detect` across every subdomain you've enumerated turns this into a single pass rather than a manual check per host — worth running immediately after subdomain enumeration completes, before deeper testing even starts.

## 💥 How To Exploit It

The workflow is mechanical: extract the exact version string, check `searchsploit` or NVD for a matching CVE, and if one exists with a public PoC, the banner has just told you exactly which exploit to try first rather than requiring you to fingerprint blindly. Apache 2.4.49's path-traversal-to-RCE (CVE-2021-41773) is the canonical example — version disclosure alone turned into full RCE with a single curl command once the CVE was public.

## 🌍 Where This Actually Bites

In an **education** sector engagement running a mix of legacy and current infrastructure, banner disclosure across the full subdomain list surfaced one host still running an Apache version with a public path-traversal CVE — a server that had clearly been patched everywhere else in the estate except this one instance, likely missed during a rolling update. In general, I treat banner disclosure across dozens of subdomains as a fast way to spot *inconsistent* patching — the interesting finding usually isn't the most common version across the estate, it's the one outlier that didn't get the same update everyone else did.

## 📋 How To Report It

Grade Low if the disclosed version has no known exploitable CVE — it's genuinely just information at that point, and inflating it to justify a finding erodes report credibility. Grade High to Critical the moment a matching CVE with a working public exploit exists, and demonstrate it rather than just citing the CVE number — a proof-of-concept command showing actual exploitation (or at minimum, confirmed vulnerable behavior) makes the finding immediately actionable for the client's patching team.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
