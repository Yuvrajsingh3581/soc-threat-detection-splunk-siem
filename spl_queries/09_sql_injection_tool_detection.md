# SQL Injection Tool Detection

## Purpose
Confirm and quantify automated SQL injection activity from the attacker IP.

## Explanation
Known SQL injection frameworks (Havij, sqlmap) often leave a distinctive user-agent fingerprint. Filtering the attacker's traffic against these signatures isolates confirmed exploitation attempts from general reconnaissance noise.

## Expected Output
A timestamped table of SQL injection requests and their resulting HTTP status, confirming 993 Havij-attributed events.

## MITRE Mapping
T1190 – Exploit Public-Facing Application.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND user_agent IN ("*sqlmap*","*Havij*")
| table _time,path,status
```
