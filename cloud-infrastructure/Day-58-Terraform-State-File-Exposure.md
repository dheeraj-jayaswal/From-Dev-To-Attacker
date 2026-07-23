# Day 58 — Terraform State File Exposure: Your Entire Infrastructure, in One JSON File

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Terraform tracks every resource it manages in a state file — and by design, that file frequently contains resource attributes in plaintext, including database passwords, private keys, and connection strings generated during provisioning, since Terraform needs to know these values to manage the resources going forward. When that state file ends up in a publicly readable location (a misconfigured S3 bucket, a public repository), it's a direct, structured dump of the infrastructure's secrets.

## 🌱 Why It Exists

Local state storage is Terraform's default, and teams migrating to shared/remote state (an S3 backend, for team collaboration) have to deliberately configure that backend's access controls — the migration itself is often driven purely by "we need to share this file across the team," without an equal focus on "and this file needs the same access restriction as the credentials inside it." Accidentally committing a state file to version control follows the same pattern as any other credential-in-git mistake — `.gitignore` has to explicitly include it, and that's an easy line to miss when Terraform is first set up.

## ⚔️ How To Find It

```bash
# Check any discovered S3 bucket for terraform state naming patterns
aws s3 ls s3://target-terraform-state --no-sign-request

# GitHub dorking for accidentally committed state files
org:target filename:terraform.tfstate
```

Terraform state buckets are commonly named predictably (`company-terraform-state`, `company-tfstate`) — the same bucket-name-guessing approach used for general S3 discovery applies directly here, and is worth trying as a dedicated check whenever S3 bucket enumeration is already part of the engagement.

## 💥 How To Exploit It

Reading the state file directly reveals resource attributes as plain JSON — searching for `password`, `secret`, `private_key`, and connection-string patterns within it surfaces credentials for potentially every resource Terraform has ever provisioned for that project, not just the one application you started the engagement testing.

## 🌍 Where This Actually Bites

In a **banking**-adjacent client's infrastructure, a Terraform state file left in a public S3 bucket contained the initial admin password for a database resource, generated at provisioning time and stored in the state as plaintext — a credential that, per client confirmation, had never actually been rotated since that initial provisioning months earlier. In **freight logistics**, an accidentally-committed `.tfstate` file in a public repository revealed the full private key for a resource meant to be internal-only, along with enough resource-naming detail to map the client's entire cloud architecture without a single active scan.

## 📋 How To Report It

Grade Critical without qualification — a readable Terraform state file is functionally equivalent to a structured credential dump for the infrastructure it manages, and should be reported with that framing. Recommend remote state storage with strict access controls (private bucket, encryption at rest, versioning with access logging) as the baseline, and specifically recommend rotating every credential found in the exposed state — not just the ones confirmed still live — since Terraform state accumulates historical values across multiple applies and a full rotation is the only way to be certain nothing exploitable remains.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
