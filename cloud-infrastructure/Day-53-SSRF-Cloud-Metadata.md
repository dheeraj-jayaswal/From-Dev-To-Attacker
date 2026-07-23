# Day 53 — SSRF to Cloud Metadata: One URL Parameter, Full Cloud Account Compromise

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Server-Side Request Forgery lets an attacker make the server itself issue an HTTP request to a destination they control — including internal-only addresses the server can reach but an external attacker normally couldn't. Every major cloud provider exposes a metadata service at a fixed internal address (`169.254.169.254`) that any workload can query for its own configuration, and critically, its temporary IAM credentials — meaning SSRF to that specific address can hand an attacker the cloud account permissions of the server itself.

## 🌱 Why It Exists

Any feature that fetches a URL on the user's behalf — a webhook validator, an image-fetch-from-URL feature, a PDF generator rendering a remote page, a "test your API endpoint" tool — creates this risk whenever the destination isn't restricted, because the developer's mental model is "fetch whatever the user gives us," without considering that "whatever" includes the server's own internal-only addresses.

## ⚔️ How To Find It

```bash
# Any feature accepting a URL parameter is a candidate
curl "https://target.com/api/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
```

Test the metadata address for AWS, GCP, and Azure specifically, since the path structure differs — AWS uses IMDSv1/v2 at the same base address with different header requirements, GCP requires a specific metadata-flavor header, and Azure has its own equivalent path — a blind SSRF finding is worth testing against all three even if you don't know the hosting provider in advance.

## 💥 How To Exploit It

Once the metadata endpoint responds, retrieving the IAM role's temporary credentials is the direct path to full cloud API access using the server's own permissions — from there, standard cloud enumeration (`aws sts get-caller-identity`, then listing accessible resources) reveals exactly how far that access reaches, which is frequently far beyond what the vulnerable application itself needed.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** platform, a "product image from URL" import feature was vulnerable to SSRF — reaching AWS's metadata endpoint and retrieving IAM credentials that turned out to have broad S3 access across the client's entire infrastructure, including buckets completely unrelated to the vulnerable image-import service. In **freight logistics**, a webhook-testing tool meant to let partners validate their own callback endpoint accepted any URL including the internal metadata address, and the retrieved credentials had permissions to the client's shipment-database backup bucket — a blast radius the original feature's developers never anticipated when building a simple webhook validator.

## 📋 How To Report It

Grade Critical without qualification the moment metadata credentials are retrieved — this is functionally full cloud account compromise scoped to whatever the instance role permits, and the severity doesn't depend on how "minor" the original vulnerable feature seemed. Demonstrate the retrieved credentials' actual scope (which resources they can access) as the concrete evidence of impact, and recommend both the immediate fix (URL allow-listing, blocking internal/link-local address ranges at the application layer) and the infrastructure-level mitigation (enforcing IMDSv2 with its required token header, which meaningfully raises the bar against basic SSRF-to-metadata attacks even if the application-layer fix is incomplete).

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
