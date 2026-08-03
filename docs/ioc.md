# Indicators of Compromise (IOC)

The following indicators were identified during the investigation and should be used to detect or block related activity in monitoring tools, firewalls, and WAF rule sets.

---

## IP Addresses

| IOC | Type | Description | Confidence |
|---|---|---|---|
| `198.51.100.55` | External IP | Source of all reconnaissance, path traversal, SQL injection, and post-exploitation activity | High |
| `10.10.1.5` | Internal Host | Internal system observed transferring data to the attacker IP | High |

---

## User Agents

| User Agent (Observed Pattern) | Associated Tooling | Behavior |
|---|---|---|
| `Havij/1.17 (Automated SQL Injection)` | Havij | Automated SQL injection attack tool |
| `sqlmap/1.7.9#stable (http://sqlmap.org)` | sqlmap | Automated SQL injection attack tool |
| `Ruby/2.7.0 (Webshell Runner)` | Custom/scripted client | Associated with webshell execution requests |
| Non-standard scripting clients (e.g. `curl`, `Wget`, `python-requests`, `Go-http-client`, `zgrab`) | Various | Automated scanning/reconnaissance clients inconsistent with normal browser traffic |

---

## Suspicious Request Paths

| Path Pattern | Category | Notes |
|---|---|---|
| `/.env` | Sensitive file exposure | Environment configuration disclosure attempt |
| `/*phpinfo*` | Sensitive file exposure | PHP diagnostic information disclosure attempt |
| `/.git*` | Sensitive file exposure | Version control directory exposure attempt |
| `*../..*` (traversal sequences) | Path traversal | Directory traversal exploitation attempts |
| `/download?file=../../etc/passwd` | Path traversal | Confirmed traversal payload targeting system files |
| `/item.php?id=1 AND SLEEP(5)--` | SQL injection | Time-based blind SQL injection payload |
| `/upload.php` (paired with webshell-style user agent) | Post-exploitation | Suspected webshell upload/execution endpoint |
| `*backup.zip*`, `*logs.tar.gz*` | Data staging | Requests consistent with staging archives for exfiltration |
| `*shell.php?cmd=*` | Command execution | Webshell command execution parameter pattern |

---

## Observed Behaviors (TTP Summary)

| Behavior | Description |
|---|---|
| Automated reconnaissance | Rapid, sequential requests to known sensitive file paths |
| Directory/path traversal | Repeated attempts to escape the web root via relative path sequences |
| SQL injection (automated) | High-volume structured injection payloads via known toolkits |
| Post-exploitation staging | Requests referencing archive files consistent with data-staging behavior |
| Suspected webshell execution | Requests carrying command-execution style parameters |
| Confirmed data exfiltration | Firewall-permitted outbound transfer totaling 126,167 bytes |

---

## Recommended Detection Rules

- Alert on any inbound request where `user_agent` matches known SQLi/scanning tool signatures
- Alert on requests to sensitive path patterns (`.env`, `.git`, `phpinfo`) from external sources
- Alert on directory traversal sequences (`../`, encoded equivalents) in request paths
- Alert on outbound firewall-allowed connections to `198.51.100.55` or its associated netblock
- Baseline and alert on unusual outbound byte-transfer volume per host
