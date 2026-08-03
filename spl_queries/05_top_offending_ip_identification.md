# Top Offending IP Identification

## Purpose
Determine which source IP is responsible for the majority of anomalous, non-browser traffic.

## Explanation
Aggregating the filtered anomalous traffic by `client_ip`, sorting descending, and taking the top results quickly isolates the primary actor of interest. This step scoped the remainder of the investigation to a single IP address.

## Expected Output
A ranked table of client IPs by event count, with `198.51.100.55` clearly dominant.

## MITRE Mapping
T1595 – Active Scanning.

## SPL Query
```spl
sourcetype=web_traffic
user_agent!=*Mozilla*
user_agent!=*Chrome*
user_agent!=*Safari*
user_agent!=*Firefox*
| stats count by client_ip
| sort -count
| head 5
```
