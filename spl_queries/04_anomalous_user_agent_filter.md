# Anomalous User-Agent Filtering

## Purpose
Strip out standard browser traffic to expose non-browser (automated/scripted) clients.

## Explanation
Legitimate human browsing almost always presents a recognizable browser user-agent string. Excluding the four major browser signatures (Mozilla, Chrome, Safari, Firefox) leaves behind a much smaller, high-signal set of automated tooling — scripts, scanners, and attack frameworks — that merit closer review.

## Expected Output
A reduced event set consisting almost entirely of non-browser clients (e.g. curl, Wget, python-requests, sqlmap, Havij).

## MITRE Mapping
T1595 – Active Scanning (supports detection of reconnaissance tooling).

## SPL Query
```spl
index=main sourcetype=web_traffic
user_agent!=*Mozilla*
user_agent!=*Chrome*
user_agent!=*Safari*
user_agent!=*Firefox*
```
