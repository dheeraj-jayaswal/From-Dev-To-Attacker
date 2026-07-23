# Day 30 — Dependency Confusion: When Your Private Package Name Isn't Actually Private

**Severity:** High → Critical | **Category:** Supply Chain Security | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Dependency confusion exploits how package managers (npm, pip) resolve a package name when both a private/internal registry and the public registry could satisfy it: many configurations check the public registry as well, and if an attacker publishes a same-named package with a higher version number on the public registry, the build tooling installs the attacker's package instead of the internal one — including whatever install-time script the attacker chose to ship with it.

## 🌱 Why It Exists

This is a build-tooling default, not a coding mistake: package managers were designed around a single global namespace, and the private/internal registry model was retrofitted on top later. Unless a project explicitly scopes its package manager configuration to prefer (or exclusively use) the internal registry for internally-named packages, the public registry remains a valid resolution target by default — and most teams never realize this until it's demonstrated against them directly.

## ⚔️ How To Find It

```bash
# Internal package names leak through JS bundles, public repos, and error messages
grep -oE '@[a-z0-9-]+/[a-z0-9-]+' app.bundle.js | sort -u

# Confirm the name is unclaimed on the public registry
npm info @target/internal-auth
```

The `confused` tool automates this check directly against a `package.json` or `requirements.txt`, flagging every internal-looking dependency that resolves to nothing on the public registry — the fastest way to scan an entire dependency tree rather than checking names one at a time.

## 💥 How To Exploit It

For reporting purposes, proving claimability is sufficient — a 404 on the public registry for a package name that's actively referenced in the target's own code, combined with an explanation of how npm/pip's resolution order would prefer a higher-versioned public package, is a complete finding. Actually registering the package and demonstrating callback execution is the kind of step that needs explicit client authorization before attempting, given the potential blast radius (any developer or CI/CD pipeline running an install).

## 🌍 Where This Actually Bites

In a **freight logistics** engagement, JS bundle mining revealed an internal npm package name used for the client's carrier-rate integration logic — unclaimed on the public registry, and referenced directly in code that ran inside the CI/CD pipeline building production deployments, meaning a successful attack here wouldn't just compromise a developer's laptop, it would compromise the build artifact shipped to production. This is the detail worth emphasizing in any report on this class of finding: the blast radius isn't "one developer's machine gets popped," it's "anything downstream of that developer's `npm install` — including the CI/CD pipeline itself — is compromised."

## 📋 How To Report It

Grade High to Critical based on where the vulnerable package is actually consumed — a package only ever installed on individual developer machines is serious; the same package referenced in a CI/CD build pipeline is Critical, since a single successful attack compromises every subsequent production deployment built through that pipeline. Recommend registering placeholder packages under the internal names on the public registry as an immediate mitigation, alongside the proper long-term fix: explicitly scoping the package manager configuration (`.npmrc` registry scoping, or pip's `--index-url` restricted to the internal registry) so public-registry resolution for internally-named packages is never attempted in the first place.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
