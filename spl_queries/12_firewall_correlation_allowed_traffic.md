# Firewall Correlation — Allowed Outbound Traffic

## Purpose
Correlate web-layer findings with network-layer data to confirm whether outbound connections to the attacker were permitted.

## Explanation
Pivoting from web logs to firewall logs validates whether any network control intervened. Filtering for allowed connections between the internal host and the attacker IP confirms that outbound traffic was not blocked.

## Expected Output
A table of allowed firewall events showing protocol, source/destination IP, destination port, and the policy reason for allowing the connection.

## MITRE Mapping
T1071 – Application Layer Protocol; T1041 – Exfiltration Over C2 Channel.

## SPL Query
```spl
sourcetype=firewall_logs
src_ip="10.10.1.5"
AND dest_ip="198.51.100.55"
AND action="ALLOWED"
| table _time,action,protocol,src_ip,dest_ip,dest_port,reason
```
