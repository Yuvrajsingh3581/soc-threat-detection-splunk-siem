# MITRE ATT&CK Mapping

This document maps the observed adversary behavior from this investigation to the [MITRE ATT&CK](https://attack.mitre.org/) framework (Enterprise Matrix). Each technique is paired with the specific evidence that supports the mapping.

---

## Summary Table

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Reconnaissance | Active Scanning | T1595 | Sequential requests to sensitive/diagnostic paths (`.env`, `.git`, `phpinfo`) from `198.51.100.55` |
| Initial Access | Exploit Public-Facing Application | T1190 | Confirmed SQL injection (993 events) and path traversal (658 attempts) against the web application |
| Execution | Command and Scripting Interpreter | T1059 | Requests carrying command-execution parameters consistent with webshell interaction |
| Persistence | Server Software Component: Web Shell | T1505.003 | Upload/execution requests paired with a non-standard "Webshell Runner" user agent |
| Discovery | File and Directory Discovery | T1083 | Path traversal payloads targeting system files (e.g. `/etc/passwd`) |
| Collection | Data Staged | T1074 | Requests referencing archive filenames (`backup.zip`, `logs.tar.gz`) prior to exfiltration |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | Firewall-confirmed outbound transfer of 126,167 bytes to attacker-controlled IP |
| Command and Control | Application Layer Protocol | T1071 | Outbound HTTP/HTTPS-permitted traffic from internal host to attacker infrastructure |

---

## Detailed Mapping

### T1595 — Active Scanning (Reconnaissance)
The attacker issued repeated requests targeting well-known sensitive file locations before any exploitation attempt occurred. This automated, sequential probing pattern is consistent with a reconnaissance/scanning tool cataloging the application's attack surface.

### T1190 — Exploit Public-Facing Application (Initial Access)
Both SQL injection and path traversal payloads were confirmed against live application parameters. The volume (993 SQLi events, 658 traversal attempts) and toolkit signatures (Havij, sqlmap) indicate deliberate, automated exploitation rather than incidental scanning noise.

### T1059 — Command and Scripting Interpreter (Execution)
Requests containing command-parameter-style query strings, associated with a distinctive scripted user agent, suggest the attacker achieved a level of remote command execution against the compromised application.

### T1505.003 — Server Software Component: Web Shell (Persistence)
The pairing of an upload endpoint with a non-browser, script-labeled user agent strongly suggests a web shell was placed on the server to provide persistent remote access independent of the original vulnerability.

### T1083 — File and Directory Discovery (Discovery)
Path traversal payloads explicitly targeting operating-system files indicate the attacker was attempting to enumerate and read files outside the intended web root.

### T1074 — Data Staged (Collection)
Requests referencing compressed archive filenames indicate the attacker (or a script running on the compromised server) was consolidating data into a transferable package prior to exfiltration.

### T1041 — Exfiltration Over C2 Channel (Exfiltration)
Firewall logs confirmed permitted outbound traffic from the internal host to the attacker's IP, with an aggregate transfer of 126,167 bytes — providing network-layer confirmation that data left the environment over the same channel used for command and control.

### T1071 — Application Layer Protocol (Command and Control)
All observed outbound communication used standard web protocols, allowing the traffic to blend in with legitimate application traffic and evade simple protocol-based detection.

---

## Notes for Detection Engineering

This mapping can be used directly to prioritize detection rule development: reconnaissance (T1595) and exploitation (T1190) techniques offer the earliest opportunity for detection and containment, while exfiltration (T1041) represents the last line of defense before impact is realized. SOC playbooks should prioritize alerting at the earliest tactics in this chain.
