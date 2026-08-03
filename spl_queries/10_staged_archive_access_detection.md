# Staged Archive Access Detection

## Purpose
Identify requests suggesting the attacker was staging data for exfiltration.

## Explanation
References to compressed archive filenames (backup and log archives) in requests from the attacker IP are a strong indicator of a data-staging step, where an attacker consolidates target data before moving it off the network.

## Expected Output
A table of timestamped requests referencing archive-style filenames, along with the associated user agent.

## MITRE Mapping
T1074 – Data Staged.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND path IN ("*backup.zip*","*logs.tar.gz*")
| table _time,path,user_agent
```
