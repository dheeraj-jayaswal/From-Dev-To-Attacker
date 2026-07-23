# Day 56 — Kubernetes API Exposure: The Cluster's Front Door Left Open

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

The Kubernetes API server is the control plane for an entire cluster — every deployment, secret, and configuration change goes through it. When it's reachable from outside the intended network without proper authentication (anonymous access left enabled, or an overly permissive RBAC binding for unauthenticated requests), an attacker can enumerate and potentially manipulate everything running in that cluster.

## 🌱 Why It Exists

Kubernetes clusters are complex enough that network exposure often traces back to a specific operational need — a monitoring tool, a CI/CD deployment pipeline — that required API access from outside the cluster's own network, and the load balancer or ingress rule set up to enable that access ends up broader than intended, reaching further than just the specific tool it was built for.

## ⚔️ How To Find It

```bash
curl -k https://target-ip:6443/api/v1/namespaces/default/secrets
curl -k https://target-ip:6443/version
```

A response without authentication confirms anonymous access is enabled — check specifically whether the `/api/v1/namespaces/*/secrets` path is reachable, since secrets are the highest-value resource in the cluster and their exposure alone is usually sufficient to demonstrate critical impact.

## 💥 How To Exploit It

Kubernetes secrets frequently contain database credentials, API keys, and TLS private keys for every service running in the cluster — reading them directly via the exposed API is the fastest path to broad compromise. If write access is also available (not just read), creating a new pod with a privileged security context can achieve node-level, and from there potentially cluster-wide, compromise.

## 🌍 Where This Actually Bites

In an **income tax** platform's cloud infrastructure, an anonymous-accessible Kubernetes API on a staging cluster exposed secrets containing database credentials that turned out to be shared with the production database connection string — a staging-environment misconfiguration with direct production impact, because credential reuse across environments meant the "less important" cluster's exposure was just as severe as if production itself had been exposed. In **education** sector platforms running multi-tenant infrastructure on Kubernetes, an exposed API on one tenant's cluster revealed namespace and secret naming conventions that, while not directly granting cross-tenant access, provided a clear roadmap for what to test next across the shared infrastructure.

## 📋 How To Report It

Grade Critical without qualification for confirmed anonymous access to secrets — this is credential compromise across every service in the cluster, demonstrated with a single unauthenticated request. Recommend disabling anonymous authentication entirely, enforcing RBAC with least-privilege bindings for any legitimately external access, and specifically auditing whether staging/non-production clusters share credentials with production — that credential-reuse pattern is what turns a "just staging" exposure into a genuinely critical finding, and it's worth calling out explicitly since it's easy for a client to under-prioritize a non-production cluster finding otherwise.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
