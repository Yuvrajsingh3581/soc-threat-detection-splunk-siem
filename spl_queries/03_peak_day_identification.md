# Peak Traffic Day Identification

## Purpose
Identify the single highest-volume day within the observation window to anchor the rest of the investigation.

## Explanation
Building on the daily distribution, sorting the timechart results by count isolates the peak day precisely. This became the investigation's anchor date, and all further attacker-specific searches were cross-referenced against it.

## Expected Output
A ranked list of dates by event count, with 2025-10-12 identified as the peak-activity day.

## MITRE Mapping
N/A — detection/analysis technique.

## SPL Query
```spl
index=main sourcetype=web_traffic
| timechart span=1d count
| sort - count
```
