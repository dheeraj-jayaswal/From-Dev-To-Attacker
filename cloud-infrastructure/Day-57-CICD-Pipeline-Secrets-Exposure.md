# Day 57 — CI/CD Pipeline Secrets Exposure: The Build System That Knows Everything

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

CI/CD systems (Jenkins, GitHub Actions, GitLab CI) hold the credentials needed to deploy an application — cloud API keys, database passwords, signing certificates — and are trusted with broad access almost by definition, since their entire job is to build and deploy production systems. When the pipeline itself is exposed (an unauthenticated Jenkins instance, an overly permissive Actions workflow, a build log that echoes secret values), that trust becomes directly reachable.

## 🌱 Why It Exists

CI/CD tooling is frequently treated as internal infrastructure rather than as an application requiring the same access-control scrutiny as anything customer-facing — a Jenkins instance stood up for a specific project rarely gets the same security review as the application it deploys, despite holding credentials with equal or greater reach. Secret leakage into build logs happens because logging is verbose by default and developers debugging a failing pipeline will often print environment variables to diagnose an issue, forgetting the log itself may be broadly readable.

## ⚔️ How To Find It

```bash
curl http://target.com:8080/api/json  # unauthenticated Jenkins API check
```

An unauthenticated Jenkins instance typically confirms itself immediately via its API or login page. For GitHub Actions/GitLab CI specifically, check public repository workflow run logs directly — even without infrastructure access, a public repo's build logs are sometimes readable by anyone and occasionally contain echoed secret values from a debugging step a developer forgot to remove.

## 💥 How To Exploit It

An unauthenticated Jenkins instance typically allows creating and running an arbitrary build job — and since Jenkins jobs execute with the credentials configured for that pipeline, a crafted job that simply prints all environment variables extracts every secret the pipeline has access to in one step. For leaked build-log secrets, the exploitation is just reading the log; no further action needed.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** client's infrastructure, an internet-reachable Jenkins instance with no authentication allowed creating a new build job that printed the full credential set used for production deployments — cloud API keys, database passwords, and a container registry credential, all extracted from a single crafted job run. In **freight logistics**, a public GitHub repository's Actions workflow logs contained an accidentally-echoed API key for a partner integration from an earlier debugging commit — the specific log line had been added temporarily to troubleshoot a failing build and simply never removed, remaining readable in the log history indefinitely.

## 📋 How To Report It

Grade Critical for any confirmed unauthenticated access to a CI/CD system with deployment credentials, or any live secret found in build logs. Recommend authentication and network restriction on all CI/CD tooling as a baseline (treat it as production infrastructure, not internal convenience tooling), and specifically flag build-log secret leakage as needing both immediate credential rotation and a scan of historical logs — a single fixed debugging line doesn't retroactively remove the secret from logs already generated and potentially already cached or archived elsewhere.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
