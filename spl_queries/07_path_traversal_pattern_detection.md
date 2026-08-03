# Path Traversal Pattern Detection

## Purpose
Identify requests from the attacker IP containing directory traversal or redirect-style payloads.

## Explanation
Path traversal attacks rely on relative path sequences to escape the intended web root. This search pattern-matches for those sequences (and related redirect-parameter abuse) within requests from the attacker IP to confirm the technique was in use.

## Expected Output
A set of matching events containing traversal sequences or redirect parameters in the request path.

## MITRE Mapping
T1083 – File and Directory Discovery; T1190 – Exploit Public-Facing Application.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND path="*..*"
OR path="*redirect*"
```
