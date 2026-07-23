# Day 65 — Workflow Step Bypass: Skipping Straight to the End of a Multi-Step Process

**Severity:** High → Critical (depends on the skipped step) | **Category:** Business Logic | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Many application flows are built as a sequence of steps — verify identity, then upload documents, then submit for approval — with the assumption baked into the frontend that a user progresses through them in order. Workflow step bypass happens when the backend doesn't independently enforce that sequence, so a later-step endpoint can be called directly without ever having completed the earlier steps it was meant to depend on.

## 🌱 Why It Exists

Multi-step flows are usually built with the frontend controlling navigation between steps — a wizard-style UI that only shows "step 3" after "step 2" completes — and it's a natural but incomplete assumption that this UI-level sequencing is sufficient, without the backend independently tracking and verifying which steps a given session or record has actually completed before allowing the next one.

## ⚔️ How To Find It

Map the full multi-step flow first (registration → document upload → verification review → account activation, for example), then attempt to call a later-step endpoint directly, with a fresh session or record that has never touched the earlier steps at all.

```bash
curl -X POST https://target.com/api/onboarding/activate-account \
  -H "Authorization: Bearer $FRESH_SESSION_TOKEN" -d '{"userId": 4821}'
```

If the later step succeeds without any server-side check confirming the prerequisite steps were actually completed for that specific user/record, the bypass is confirmed.

## 💥 How To Exploit It

The impact is entirely dependent on what the skipped steps were meant to enforce: skipping an identity-verification step and reaching account activation directly means an account gets activated without ever actually being verified; skipping a required approval step in an order-fulfillment flow means an order can be marked fulfilled without the checks that step was meant to apply.

## 🌍 Where This Actually Bites

In an **income tax** platform's return-filing flow, the sequence required document verification before final submission — but the submission endpoint itself didn't check whether verification had actually completed for that specific filing, meaning a return could be submitted and accepted as final without the mandatory verification step ever occurring. In an **education** platform's admissions workflow, the "final decision" endpoint could be called directly on an application record that had never passed through the required document-review step — a candidate's admission could be finalized without the review that step existed specifically to enforce.

## 📋 How To Report It

Grade by what the skipped step was actually protecting against — skipping a cosmetic "are you sure" confirmation is low-impact; skipping identity verification, document review, or an approval gate on a financial or compliance-relevant flow is Critical, since it means the entire process can complete without ever satisfying a requirement the business (and often a regulator) depends on. Recommend the backend track workflow state explicitly per record (a status field verified server-side at each step) rather than relying on the frontend's own step-by-step navigation as the enforcement mechanism.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
