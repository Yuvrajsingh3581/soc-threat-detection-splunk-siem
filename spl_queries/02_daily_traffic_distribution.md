# Daily Traffic Distribution

## Purpose
Visualize how web traffic volume is distributed across each day in the observation window.

## Explanation
Charting event count over a daily span (`timechart span=1d count`) converts a large, flat dataset into a visual/tabular trend that makes volume anomalies easy to spot. This is a standard first analytical step when hunting for a spike tied to malicious activity.

## Expected Output
A time-indexed table/chart showing event counts per day, with one day standing out as significantly higher than the rest.

## MITRE Mapping
N/A — detection/analysis technique, not an adversary technique.

## SPL Query
```spl
index=main sourcetype=web_traffic
| timechart span=1d count
```
