# Day 5 — DNS Zone Transfer: The One Command That Prints the Whole Blueprint

**Severity:** High | **Category:** Recon & Fingerprinting | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

A DNS Zone Transfer (AXFR) is a legitimate mechanism that lets a secondary DNS server pull a complete copy of a domain's records from the primary server — every subdomain, every internal IP, every mail server entry, in one request. When the primary server is misconfigured to answer AXFR requests from *anyone* rather than only from approved secondary servers, an attacker gets that entire blueprint in a single command.

## 🌱 Why It Exists

Zone transfer is meant to be restricted to a short allow-list of secondary nameserver IPs. In practice, this restriction gets missed for a very ordinary reason: DNS configuration is usually set up once, early in a project, by whoever stood up the infrastructure — and it's rarely revisited unless someone is specifically auditing DNS hygiene. Teams focus their security review on the application layer; the nameserver config sits quietly in the background, correct on day one and never checked again years later.

## ⚔️ How To Find It

```bash
# Step 1 — identify the nameservers
dig NS target.com +short

# Step 2 — attempt AXFR against each one
dig AXFR target.com @ns1.target.com
dig AXFR target.com @ns2.target.com
```

A vulnerable server returns the full record set ending in "Transfer complete." A properly configured one returns "Transfer failed." Automate the sweep across all discovered nameservers with `dnsrecon -d target.com -t axfr` or nmap's `dns-zone-transfer` script for a quick network-wide check.

## 💥 How To Exploit It

The zone dump itself is the payoff — internal hostnames like `db-master.target.com` or `vault.internal.target.com` map out infrastructure that no amount of subdomain brute-forcing would ever surface. From there it chains directly: internal IPs feed SSRF payloads, old/undocumented API subdomains become BFLA targets, and dev/staging hosts revealed in the dump often turn out to be running in debug mode.

## 🌍 Where This Actually Bites

In **banking** environments, a successful zone transfer routinely surfaces internal hostnames for core banking components that were never meant to be internet-discoverable — the kind of naming convention leak that turns a generic pentest into a targeted attack plan overnight. In **income tax** platforms, exposed mail server records make convincing phishing infrastructure trivial to build, since the attacker now knows exactly which mail relay to spoof. I've also seen this in **freight logistics** engagements where a secondary/DR nameserver was left with the default "allow all transfers" configuration long after the primary was hardened — nobody applied the same fix twice.

## 📋 How To Report It

Attach the full `dig AXFR` output as evidence, but redact or summarize the internal IP list rather than dumping the raw addresses verbatim into a shareable report. Score it around CVSS 7.5 (network-exploitable, no privileges, no user interaction, high confidentiality impact) and frame the remediation precisely: configure `allow-transfer` to an explicit list of secondary server IPs in the zone file — a two-minute BIND config change that closes the issue permanently. Recommend re-testing every nameserver individually, since it's common for only one of several to be misconfigured.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
