# Day 24 — Shodan & Censys: Finding Exposed Services Without Touching the Target

**Severity:** High | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Shodan and Censys continuously scan the entire public IPv4 address space, grabbing service banners from every open port, and make that data searchable by organization, ASN, hostname, or SSL certificate. Rather than scanning a target yourself, you're reading a scan someone else already ran — searching by a company's name or IP range surfaces every exposed service, often including databases and admin tools nobody intended to be internet-facing.

## 🌱 Why It Exists

The exposure itself is almost always a network-segmentation gap rather than an application bug: a database or admin tool gets deployed with a default "listen on all interfaces" configuration during setup, intended to be reachable only from an internal network — but the firewall rule restricting that access never gets added, or gets removed later during unrelated infrastructure changes. Shodan doesn't create the exposure; it just makes an existing one trivially discoverable.

## ⚔️ How To Find It

```bash
shodan search --fields ip_str,port,product 'org:"Target Inc" port:27017,6379,9200,3389'
shodan domain target.com
```

Cross-reference with Censys for coverage Shodan misses — the two platforms run different scanning infrastructure and rarely return identical result sets. Once you have a candidate, verify manually rather than trusting the banner's age: `redis-cli -h <ip> CONFIG GET requirepass` returning empty confirms no auth on Redis; `curl http://<ip>:9200/_cat/indices` returning data confirms open Elasticsearch.

## 💥 How To Exploit It

For databases with no authentication (MongoDB, Redis, Elasticsearch by default), the "exploit" is simply connecting and reading — no payload required. The severity comes entirely from what's actually inside once you connect, which is why verification always has to go one step further than the banner: a banner confirms the service exists, not that it's exploitable.

## 🌍 Where This Actually Bites

In a **retail/ecommerce** engagement, a Shodan sweep on the client's ASN surfaced an Elasticsearch instance with no authentication, indexing what turned out to be the full product catalog and — more seriously — a customer order index that should never have been on a search cluster reachable from the public internet. In **freight logistics**, I've found exposed Redis instances used as a caching layer for a partner integration, with `requirepass` never set — meaning session data and cached API tokens for that integration were readable by anyone who found the IP.

## 📋 How To Report It

Grade by the sensitivity of what's actually accessible once connected — an empty or clearly non-production database is Low/Informational, while a live, populated production dataset with no auth is Critical regardless of which specific database engine is involved. Document the exact verification command and its (redacted) output as evidence, and always note the scan/banner date from Shodan alongside your own live re-verification, since a stale banner claiming "no auth" that's since been fixed is a false positive you don't want in a report.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
