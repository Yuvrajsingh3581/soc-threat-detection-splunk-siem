# Baseline Traffic Overview

## Purpose
Establish a baseline understanding of the web traffic index before attempting to identify anomalies.

## Explanation
This is the starting point of any SOC investigation: a broad, unfiltered search against the relevant index and sourcetype to understand event volume, available fields, and general traffic shape. No analyst should jump straight to filtering without first understanding what "normal" looks like in the data.

## Expected Output
Returns the full set of web traffic events with fields such as `client_ip`, `path`, `method`, `status`, `user_agent`, `referer`, and `size_bytes` available for further analysis.

## MITRE Mapping
N/A — preparatory/triage step, not an adversary technique.

## SPL Query
```spl
index=main sourcetype=web_traffic
```
