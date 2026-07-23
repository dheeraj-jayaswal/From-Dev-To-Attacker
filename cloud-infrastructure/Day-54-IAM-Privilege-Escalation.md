# Day 54 — IAM Privilege Escalation: When "Least Privilege" Was Never Actually Enforced

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Cloud IAM privilege escalation happens when an identity with limited, seemingly harmless permissions can leverage a specific permission combination to grant itself broader access — attaching a more permissive policy to its own role, creating a new access key for a more privileged identity, or launching compute resources that assume a higher-privileged role than the one that launched them.

## 🌱 Why It Exists

IAM policies accumulate over time as different features and integrations each request the specific permission they need, and it's genuinely difficult to reason about the *combination* of permissions across dozens of policies attached to one role — a permission that looks harmless in isolation (`iam:PassRole`, `iam:CreateAccessKey`) becomes an escalation path only when combined with certain other permissions, and that combination is rarely reviewed as a whole once each individual grant was approved separately at the time it was requested.

## ⚔️ How To Find It

```bash
# Enumerate the current identity's actual permissions, don't trust the role name
aws iam get-role --role-name target-role
aws iam list-attached-role-policies --role-name target-role
```

Once you have limited credentials (from SSRF-to-metadata, a leaked key, or a legitimately scoped test account), systematically check for known escalation permission combinations — `iam:CreatePolicyVersion` on a policy you're attached to, `iam:PassRole` combined with the ability to launch a compute resource, or `iam:AttachUserPolicy` on your own user.

## 💥 How To Exploit It

Each escalation path follows a predictable pattern: if you can create a new version of a policy already attached to your role, you can define that new version with full administrative permissions; if you can pass a role to a new Lambda function or EC2 instance and also create/invoke it, you inherit whatever permissions that target role has, regardless of your own role's stated limits.

## 🌍 Where This Actually Bites

In a **banking**-adjacent client's cloud environment, a role built for a specific CI/CD deployment pipeline had `iam:PassRole` alongside EC2 launch permissions — enough to launch a new instance assuming a far more privileged role than the pipeline itself was ever meant to have, discovered while testing what should have been a narrowly-scoped deployment credential. In **retail/ecommerce**, a developer-facing role intended only for read access to application logs had accidentally retained `iam:CreateAccessKey` permission on IAM users from an earlier, broader policy that was never fully rolled back — enough to mint new credentials for a far more privileged service account.

## 📋 How To Report It

Grade Critical for any confirmed escalation path, and demonstrate it concretely rather than just citing the theoretical permission combination — actually creating the elevated credential (in an authorized test, cleaned up immediately afterward) is far more persuasive to a client's cloud security team than a policy-analysis argument alone. Recommend a full least-privilege policy audit rather than patching the single escalation path found — these combinations tend to recur across multiple roles built at different times by different teams, and fixing one instance rarely addresses the underlying pattern of permission accumulation across the account.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
