<div align="center">

# 🛡️ From Dev To Attacker

**A field journal on why vulnerabilities exist — not just how to exploit them.**

*Written from 5+ years of hands-on Web App & API penetration testing across
Income Tax, Banking, Retail, E-commerce, Freight Logistics, and Education platforms —
by someone who spent years writing the code attackers now target.*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Dheeraj%20Kumar%20Jayaswal-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/dheerajkumarjayaswal)
[![GitHub](https://img.shields.io/badge/GitHub-dheeraj--jayaswal-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/dheeraj-jayaswal)
[![Location](https://img.shields.io/badge/Location-Pune%2C%20India-FF6B6B?style=for-the-badge&logo=googlemaps&logoColor=white)](https://github.com/dheeraj-jayaswal)

</div>

---

## 📌 What Is This Repository?

Most AppSec content teaches you **how** to exploit a vulnerability. This journal spends
equal time on **why the vulnerability exists in the first place** — because I didn't
start my career as an attacker. I spent years as a full-stack developer (ASP.NET / SQL
Server) before moving into offensive security, and that background is the entire premise
of this repository: I've written the kind of code that gets exploited, so I know exactly
which shortcuts, deadline pressures, and assumptions lead to it.

I'm currently Technology Lead – Offensive Security at Infosys, with 15+ years in IT and
5+ years focused entirely on Web Application & API Penetration Testing. Every write-up
here reflects real patterns seen across professional engagements (sanitised and
anonymised — no client-identifying detail is ever included).

Each entry follows the same structure:

| Section | What you'll find |
|---|---|
| 🔍 **What it is** | Plain-English explanation — no jargon wall |
| 🌱 **Why it exists** | The actual developer/architectural mistake behind it — the "dev" lens |
| ⚔️ **How to find it** | Recon and detection methodology, real tool commands |
| 💥 **How to exploit it** | Step-by-step exploitation, PoC-level detail — the "attacker" lens |
| 🌍 **Where this actually bites** | Real business impact, broken down by industry |
| 📋 **How to report it** | Severity rationale, business framing, remediation guidance |

---

## 🏢 Why the Domain Angle Matters

A vulnerability class doesn't carry the same weight everywhere. Having tested
applications across **Income Tax platforms, Banking systems, Retail & E-commerce
storefronts, Freight Logistics platforms, and Education portals**, I've seen the exact
same bug produce a shrug in one industry and a regulatory incident in another. The
**"Where this actually bites"** section in every write-up exists specifically to carry
that context — generically, without any real client data — so severity reasoning
reflects actual business risk, not just a CVSS score in isolation.

---

## 🗂️ Categories

This journal now spans the full breadth of enterprise Web App & API pentesting — 67 entries across 11 categories. New entries will continue to be added over time within these categories.

- **🔍 Recon & Fingerprinting** — subdomain enumeration, DNS zone transfer, wildcard detection, subdomain takeover, dangling CNAMEs, reverse DNS, ASN/IP range mapping, certificate transparency, JS endpoint extraction, Wayback Machine harvesting, directory/file bruteforce, hidden parameter discovery, API endpoint discovery, robots.txt/sitemap leaks, Shodan/Censys exposed services, server banner & framework version disclosure, admin panel discovery, Cloudflare origin IP leaks
- **🛡️ Access Control** — IDOR, BOLA, privilege escalation, mass assignment
- **🔑 Authentication & Session** — JWT attacks, auth bypass, MFA bypass, password reset flaws, brute force, username enumeration (response & timing), account lockout bypass
- **🗃️ Sensitive Data Exposure** — pastebin/gist leaks, backup file discovery, source map exposure, GitHub dorking for API keys, hardcoded credentials in git history, exposed S3 buckets
- **🔗 Supply Chain Security** — dependency confusion (npm/PyPI namespace squatting)
- **⚙️ Security Misconfiguration** — debug mode in production, error stack traces, phpinfo/debug endpoints, security header analysis
- **💉 Injection** — SQL injection, NoSQL injection, OS command injection, SSTI, XSS, CSV/formula injection
- **🌐 API Security** — BOLA, mass assignment, excessive data exposure, lack of rate limiting, GraphQL security, BFLA
- **☁️ Cloud & Infrastructure** — SSRF to cloud metadata, IAM privilege escalation, exposed Docker/Kubernetes APIs, CI/CD secrets exposure, Terraform state exposure
- **📁 File Upload & Path Traversal** — unrestricted file upload, path traversal/LFI, ZIP Slip, remote file inclusion
- **💼 Business Logic** — race conditions, price manipulation, workflow step bypass, coupon/discount abuse, negative quantity/value manipulation

---

## 📖 How To Use This

**New to AppSec?** Start with Recon & Fingerprinting — every entry builds up from first
principles before commands appear.

**Experienced tester or consultant?** Jump to any topic directly. The domain-impact
sections and reporting guidance are written to be adaptable for your own client-facing
reports.

**Developer reading this to secure your own code?** The "Why it exists" section in every
entry is written specifically for you — it names the actual coding/architecture mistake,
not just the attack.

---

## 👤 About Me

- **Name** — Dheeraj Kumar Jayaswal
- **Role** — Technology Lead – Offensive Security, Infosys Limited
- **Focus** — Web Application & API Penetration Testing
- **Experience** — 15+ years in IT · 5+ years in Offensive Security
- **Edge** — Former full-stack developer (ASP.NET / SQL Server) — I think like a developer, attack like a hacker
- **Domains tested** — Income Tax · Banking · Retail · E-commerce · Freight Logistics · Education
- **Certifications** — CEH (2021) · AWS Certified Solutions Architect – Associate · AWS Certified Cloud Practitioner · Pursuing OSCP & IIT Kanpur Executive Cert in Cyber Security

---

## 📂 Other Repositories in This Series

| Repository | What's in it |
|---|---|
| [AppSec-From-The-Trenches](https://github.com/dheeraj-jayaswal/AppSec-From-The-Trenches) | The core enterprise web app & API vulnerability knowledge base — full write-ups across OWASP Top 10, access control, injection, and more |
| [API-From-The-Trenches](https://github.com/dheeraj-jayaswal/API-From-The-Trenches) | Dedicated API security series — BOLA, JWT attacks, GraphQL, mass assignment, OWASP API Top 10 coverage |
| [DarkWeb-From-The-Trenches](https://github.com/dheeraj-jayaswal/DarkWeb-From-The-Trenches) | Threat intelligence & dark web OSINT methodology — credential leak monitoring, ransomware tracking, pre-engagement TI |
| [Notes-From-The-Trenches](https://github.com/dheeraj-jayaswal/Notes-From-The-Trenches) | Hands-on cybersecurity notes and practical learning collected from real-world pentesting |
| [Bug-Bounty-Hunting-Companion](https://github.com/dheeraj-jayaswal/Bug-Bounty-Hunting-Companion) | Personal bug bounty methodology, recon-to-report checklists, and testing playbooks |
| [VDP-Dorks](https://github.com/dheeraj-jayaswal/VDP-Dorks) | Curated Google dorks for authorized Vulnerability Disclosure Program recon |
| [.pcap-Arsenal](https://github.com/dheeraj-jayaswal/.pcap-Arsenal) | Packet captures organized by protocol, for Web/API/Network-layer analysis and learning |

**From-Dev-To-Attacker** (this repo) is the newest addition — the "why it breaks" companion to the rest of the series.

---

## 🤝 Connect

[LinkedIn](https://linkedin.com/in/dheerajkumarjayaswal) — open to consulting, collaboration, and security discussions.

---

<div align="center">

*Every vulnerability has two stories: how it was written, and how it was broken.*
*This journal tells both.*

**#FromDevToAttacker · #AppSec · #PenTest · #WebSecurity · #APISecurity · #OffensiveSecurity**

</div>
