# Day 28 — Admin Panel Discovery: Every Application Has a Backend Door

**Severity:** High → Critical (based on auth status) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Admin panels concentrate the highest-value functionality in any application — user management, data export, configuration control — behind what's frequently the least-tested authentication path, because internal QA and security review naturally focus on the customer-facing flows that generate the most usage and the most bug reports.

## 🌱 Why It Exists

CMS and framework defaults make admin paths highly predictable (`/wp-admin`, `/administrator`, `django's /admin/`), and that predictability is a deliberate framework design choice, not a mistake — the actual gap is that teams rarely revisit the admin path's authentication rigor with the same scrutiny applied to customer login, on the assumption that "it's not public-facing" even though the path itself is guessable in seconds.

## ⚔️ How To Find It

```bash
ffuf -u https://target.com/FUZZ -w seclists/AdminPanels.fuzz.txt -mc 200,301,302,401,403
```

A 401/403 response still confirms the path exists and is worth documenting — it's a lower-severity finding than an open panel, but it confirms attack surface and is worth a quick default-credential check regardless, since admin/admin and similarly weak defaults show up on forgotten tooling more often than they should.

## 💥 How To Exploit It

The escalation path from "found the panel" to "confirmed impact" runs through three checks in order: is there any authentication at all; if so, do default or trivially weak credentials work; and separately — regardless of the panel's own login — does a regular authenticated user token reach admin-only API endpoints directly, since frontend route guards frequently aren't backed by equivalent server-side authorization checks.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, an admin panel subdomain (`admin.client.com`) was reachable with zero authentication — a separate deployment from the main storefront that had clearly been set up by a different team at a different time, missing the auth middleware the main application had. In **income tax** platform testing, I've confirmed cases where the admin panel itself required proper login, but specific admin API endpoints underneath it accepted a regular taxpayer-level authenticated token — the UI correctly hid admin functions from regular users, but the backend never verified the role independently of what the frontend chose to render.

## 📋 How To Report It

Grade Critical for any admin panel reachable with zero authentication or trivially weak default credentials — this needs no further chaining to justify the severity. Grade the "authenticated user reaches admin endpoint" variant as Critical as well, but document it distinctly, since the remediation is different: one is "add authentication to this panel," the other is "add server-side role verification independent of the UI," and conflating the two in a report can lead to an incomplete fix that only addresses the panel's login page.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
