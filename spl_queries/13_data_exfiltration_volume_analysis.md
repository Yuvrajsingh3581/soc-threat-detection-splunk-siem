# Data Exfiltration Volume Analysis

## Purpose
Quantify the total volume of data transferred from the internal host to the attacker's infrastructure.

## Explanation
Summing the `bytes_transferred` field across all allowed connections between the internal host and the attacker IP provides a concrete, defensible figure for the incident report's impact assessment.

## Expected Output
A single aggregate value showing total bytes transferred, confirmed at 126,167 bytes from `10.10.1.5`.

## MITRE Mapping
T1041 – Exfiltration Over C2 Channel.

## SPL Query
```spl
sourcetype=firewall_logs
src_ip="10.10.1.5"
AND dest_ip="198.51.100.55"
AND action="ALLOWED"
| stats sum(bytes_transferred) by src_ip
```
