# Sensitive Path Reconnaissance Detection

## Purpose
Confirm whether the identified actor attempted to access known sensitive file paths.

## Explanation
Attackers commonly probe for exposed configuration files, diagnostic pages, and version-control directories before attempting exploitation. Filtering the attacker's requests against a list of well-known sensitive paths confirms this reconnaissance behavior.

## Expected Output
A table of timestamped requests to sensitive paths (`.env`, `phpinfo`, `.git`) along with the user agent and HTTP status returned.

## MITRE Mapping
T1595 – Active Scanning; T1083 – File and Directory Discovery.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND path IN ("/.env","/*phpinfo*","/.git*")
| table _time,path,user_agent,status
```
