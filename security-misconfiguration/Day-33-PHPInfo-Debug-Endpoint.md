# Day 33 — phpinfo() and Debug Endpoints: One Diagnostic File, Full Server Blueprint

**Severity:** High | **Category:** Security Misconfiguration | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

`phpinfo()` is a single function call that dumps every PHP configuration setting, every loaded extension, every environment variable, and every relevant file system path as a formatted page — a genuinely useful diagnostic tool during setup and troubleshooting that, left reachable in production, hands an unauthenticated visitor a complete server configuration snapshot in one request.

## 🌱 Why It Exists

The file usually gets created deliberately during initial server setup to confirm PHP is configured correctly, and the removal step afterward is exactly the kind of one-time cleanup task that gets forgotten once the immediate setup problem is solved and attention moves elsewhere — the file keeps working indefinitely because nothing about the server naturally expires it.

## ⚔️ How To Find It

```bash
for path in phpinfo.php info.php test.php debug.php; do
  curl -s -o /dev/null -w "%{http_code} $path\n" "https://target.com/$path"
done
```

Framework-specific debug tooling deserves the same check — Laravel's `/_debugbar` and `/telescope`, Spring Boot's `/actuator/env` — since these are functionally equivalent diagnostic-dump endpoints for their respective stacks, often left reachable for the same "forgot to remove after setup" reason.

## 💥 How To Exploit It

Read directly for environment variables containing credentials — database passwords, API keys, and signing secrets frequently sit in plaintext in the dumped environment. Two specific config values are worth checking in a `phpinfo()` dump beyond the obvious credential search: `allow_url_include` set to On opens a remote-file-inclusion path, and an empty `disable_functions` list means `exec()`-family functions are available if any other vulnerability provides a foothold to use them.

## 🌍 Where This Actually Bites

In an **education** sector engagement running a legacy PHP-based portal, a `phpinfo.php` file left over from initial server setup years earlier exposed the production database password directly in the dumped environment variables — a single unauthenticated request handed over full database access. In a **banking**-adjacent client running Spring Boot microservices, an exposed `/actuator/env` endpoint on an internal-facing service dumped every environment variable including the JWT signing secret used across the service mesh — turning one forgotten diagnostic endpoint into a path to forge valid tokens for the entire internal service network.

## 📋 How To Report It

Grade High as a baseline, and escalate to Critical the moment any live credential is confirmed in the dumped output — quote the specific config values found (redacted appropriately) rather than a generic "sensitive information was exposed," since the specific `allow_url_include`/`disable_functions` state or the exact credential type found changes what the client needs to prioritize first. Recommend deleting the specific file and adding a deployment-checklist item to prevent recurrence, since this exact finding tends to reappear on the next server provisioned the same way if the root process gap isn't addressed.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
