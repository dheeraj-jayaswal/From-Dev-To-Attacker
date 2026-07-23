# Day 55 — Exposed Docker API: Container Management With No Door

**Severity:** Critical | **Category:** Cloud & Infrastructure | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Docker's management API, when bound to a TCP port (commonly 2375) without authentication or TLS, gives whoever can reach that port complete control over every container on the host — the ability to list, start, stop, and critically, create *new* containers with arbitrary configuration, including mounting the host's own filesystem into a new container.

## 🌱 Why It Exists

Docker's default configuration listens only on a local Unix socket, which is safe — this exposure requires someone to have deliberately enabled the TCP API, almost always to allow remote management from a CI/CD pipeline or a separate orchestration tool, and then either forgot to add TLS/authentication or assumed the network itself (a VPC, a "private" subnet) provided sufficient isolation on its own.

## ⚔️ How To Find It

```bash
curl http://target-ip:2375/version
curl http://target-ip:2375/containers/json
```

A response to either of these without any authentication challenge confirms the API is open — no credentials needed to reach this point, since the vulnerability is precisely that no credentials are being checked.

## 💥 How To Exploit It

The most direct path to host compromise is creating a new container with the host's root filesystem mounted as a volume, then executing commands inside that new container to read or write files on the underlying host directly — this converts container-API access into full host compromise in a single API call, not a multi-step chain.

```bash
curl -X POST http://target-ip:2375/containers/create -H "Content-Type: application/json" \
  -d '{"Image":"alpine","Cmd":["sh"],"HostConfig":{"Binds":["/:/host"]}}'
```

## 🌍 Where This Actually Bites

In a **freight logistics** client's staging environment, an exposed Docker API on a host that was meant to be internal-only (but reachable due to an overly permissive security-group rule) allowed full host filesystem access via the volume-mount technique — from a "just a staging server" starting point to reading production-adjacent configuration and credential files stored on that host. In **retail/ecommerce**, a CI/CD build agent's Docker API was reachable from a wider network segment than intended, and the host it ran on had cached credentials for the container registry — meaning compromise of that single Docker API led directly to the ability to push malicious images that would later be pulled and deployed by the legitimate pipeline.

## 📋 How To Report It

Grade Critical without qualification — an exposed, unauthenticated Docker API is equivalent to unrestricted root access on the host, and the report should state that plainly rather than softening it as a "container security" issue. Recommend binding the Docker API to the local Unix socket only wherever remote access genuinely isn't required, and where remote management is a real operational need, enforcing mutual TLS authentication on the TCP endpoint — network-level isolation (a private subnet, a security group rule) alone is not a sufficient control on its own, since network boundaries shift over time in ways application-layer authentication doesn't depend on.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
