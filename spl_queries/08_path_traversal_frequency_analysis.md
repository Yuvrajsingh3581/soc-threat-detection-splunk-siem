# Path Traversal Frequency Analysis

## Purpose
Quantify how many distinct path traversal attempts were made and identify the specific payloads used.

## Explanation
Aggregating the traversal-matching results by `path` provides a frequency count per unique payload, turning a qualitative confirmation into a quantified finding suitable for the incident report.

## Expected Output
A ranked table of traversal-related paths and their occurrence counts, totaling 658 attempts.

## MITRE Mapping
T1083 – File and Directory Discovery; T1190 – Exploit Public-Facing Application.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND path="*..\/..\/*"
OR path="*redirect*"
| stats count by path
```
