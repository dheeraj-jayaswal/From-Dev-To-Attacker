# Day 15 — Hidden Parameter Discovery: The Inputs Nobody Documented but the Server Still Reads

**Severity:** Medium → Critical (depends on what the parameter unlocks) | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Hidden parameter discovery is the process of finding server-side inputs — query parameters, form fields, JSON body keys — that aren't reflected anywhere in the visible UI or documentation, but that the backend still actively processes. A `?debug=true` added during development, a `role` field the frontend never sends but the backend still accepts — these live entirely outside normal testing unless you go looking for them specifically.

## 🌱 Why It Exists

This is a direct consequence of how iterative development actually works: a developer adds a flag to speed up their own local debugging or testing, it does its job, and then — because it isn't wired into any UI element — nobody circles back to remove it before shipping. The backend has no concept of "this parameter was only for internal use"; it just keeps accepting whatever field names arrive in the request, which is precisely the mass-assignment root cause showing up again in a different form.

## ⚔️ How To Find It

```bash
# Arjun automates the differential-response detection
arjun -u https://target.com/api/users/1 -m GET -H "Authorization: Bearer TOKEN"

# ffuf works well once you have a solid parameter wordlist and a baseline size to filter against
ffuf -u "https://target.com/api/users/1?FUZZ=1" -w burp-parameter-names.txt -fs <baseline_size>
```

Don't skip mining the frontend JS bundle for parameter names the developers actually wrote — `fetch()`/`axios` calls in the bundle often reference fields that never made it into the visible form, which is a far more targeted list than a generic wordlist.

## 💥 How To Exploit It

Once a parameter changes the response, the follow-up is systematic: try it in the JSON body as well as the query string (many backends bind the same field from either location), and specifically test privilege-adjacent values — `role`, `admin`, `verified`, `internal` — since these are the ones that turn an information-disclosure bug into a full authorization bypass rather than just extra debug noise in the response.

## 🌍 Where This Actually Bites

In a **banking** engagement, a hidden `?debug=true` parameter on an internal reporting API returned the underlying SQL query and database host alongside the normal response — information that on its own seems academic, until you realize it hands an attacker exactly the internal hostname needed for a follow-up SSRF attempt. In **retail/ecommerce** registration flows, I've found `role` and `accountTier` accepted silently in the JSON body of a signup request that the visible form never exposes — a direct mass-assignment path to a discounted or elevated account tier that costs the business real money per exploited signup, not just a theoretical finding.

## 📋 How To Report It

Grade by the actual consequence, not by the fact that a parameter exists — a `?format=json` toggle that changes nothing sensitive isn't reportable, while a `role` field silently accepted in a registration body is Critical regardless of how it was discovered. Include the exact request/response pair showing the size or content difference that led you to the parameter, and test both GET and POST/JSON paths explicitly before concluding a given field has no effect — inconsistent binding between request types is common enough that testing only one direction misses real findings.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
