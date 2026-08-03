# SOC Incident Report

**Report ID:** IR-2025-1012-WEB01
**Classification:** Internal — Simulated Environment
**Prepared by:** Yuvraj Singh, SOC Analyst (Investigation Lead)
**Report Date:** October 2025
**Status:** Closed — Findings Confirmed

---

## 1. Executive Summary

Between early October 2025 and 2025-10-12, a single external IP address, **198.51.100.55**, conducted a sustained, multi-phase attack campaign against a monitored public-facing web application. The activity progressed from reconnaissance and sensitive-file probing through path traversal and SQL injection exploitation attempts, culminating in evidence of webshell-based command execution and confirmed outbound data transfer.

The investigation was triggered by a statistically significant spike in web traffic volume on 2025-10-12, which prompted a full SIEM-driven review of web access and firewall telemetry. Analysis confirmed **993 automated SQL injection events** attributed to the Havij toolkit, **658 path traversal attempts**, and **126,167 bytes** of data transferred from the internal network to the attacker's infrastructure via firewall-permitted outbound traffic.

This report documents the investigative methodology, evidence collected, confirmed findings, business impact, and recommended remediation actions.

---

## 2. Scope

**In Scope:**
- Web application access logs (`sourcetype=web_traffic`) covering the full observation window
- Firewall/network logs (`sourcetype=firewall_logs`) covering traffic between the internal host `10.10.1.5` and the identified attacker IP
- All activity attributed to source IP `198.51.100.55`

**Out of Scope:**
- Endpoint forensics (no EDR telemetry was available in this environment)
- Any traffic not captured within the ingested log indices
- Attribution of the threat actor's identity or organizational affiliation

---

## 3. Methodology

The investigation followed a standard SOC triage-to-containment approach, executed entirely within Splunk Enterprise using SPL:

1. **Baseline review** — establish normal traffic volume across the monitored index
2. **Anomaly identification** — isolate statistically abnormal traffic days
3. **Behavioral fingerprinting** — filter out legitimate browser user agents to expose automated tooling
4. **Actor isolation** — identify the single IP responsible for the majority of anomalous requests
5. **Technique confirmation** — validate specific attack techniques (recon, path traversal, SQL injection) through targeted searches
6. **Impact verification** — correlate web-layer findings against firewall logs to confirm outbound data movement
7. **Timeline construction** — sequence all confirmed events chronologically
8. **Reporting** — document findings, impact, and remediation guidance

A full breakdown of each investigative step and the reasoning behind it is available in [`investigation_notes.md`](investigation_notes.md).

---

## 4. Investigation

### 4.1 Initial Triage
A baseline search across `index=main sourcetype=web_traffic` returned a high daily event volume. Charting this volume over time surfaced a clear spike on **2025-10-12**, which became the focal point of the investigation.

### 4.2 Behavioral Anomaly Detection
Filtering out standard browser user-agent strings (Mozilla, Chrome, Safari, Firefox) isolated a small set of non-browser clients responsible for a disproportionate share of traffic. Aggregating by source IP identified **198.51.100.55** as the dominant anomalous actor.

### 4.3 Reconnaissance Confirmation
Requests from the identified IP targeted common sensitive-file paths (environment configuration files, PHP diagnostic pages, version control directories), consistent with automated reconnaissance/scanning behavior.

### 4.4 Path Traversal Confirmation
Pattern-matching on directory traversal sequences and redirect-style parameters within request paths from the same source IP confirmed repeated path traversal attempts against the application.

### 4.5 SQL Injection Confirmation
User-agent analysis revealed automated exploitation tooling — specifically **Havij** and **sqlmap** signatures — issuing structured injection payloads against application query parameters.

### 4.6 Post-Exploitation Indicators
Further path analysis from the same source IP surfaced requests consistent with staged archive retrieval and webshell-style command execution parameters, indicating the actor moved beyond exploitation into a post-compromise phase.

### 4.7 Network-Layer Correlation
Firewall logs were queried for allowed outbound traffic between the internal host `10.10.1.5` and the attacker IP. All matching connections were permitted by policy, and aggregate byte-transfer analysis confirmed a measurable data outflow.

---

## 5. Evidence

| Evidence Type | Source | Summary |
|---|---|---|
| Web access logs | `sourcetype=web_traffic` | Full request history for `198.51.100.55` across the observation window |
| Firewall logs | `sourcetype=firewall_logs` | Allowed connections between `10.10.1.5` and `198.51.100.55` |
| SPL query results | Splunk Search & Reporting | Screenshots and exported statistics (see `/images`) |

All supporting SPL queries used to generate this evidence are catalogued individually in [`/spl_queries`](../spl_queries).

---

## 6. Findings

| # | Finding | Confidence |
|---|---|---|
| 1 | Sustained reconnaissance activity from `198.51.100.55` targeting sensitive application paths | High |
| 2 | Confirmed path traversal exploitation attempts (658 instances) | High |
| 3 | Confirmed automated SQL injection activity via Havij (993 events) | High |
| 4 | Evidence of webshell/command execution attempts post-exploitation | Medium |
| 5 | Confirmed outbound data transfer of 126,167 bytes to attacker infrastructure | High |
| 6 | Peak attack activity concentrated on 2025-10-12 | High |

---

## 7. Impact

Based on the evidence gathered, the attacker demonstrated the capability to:

- Enumerate sensitive application file paths
- Exploit SQL injection vulnerabilities against backend data stores
- Potentially achieve remote command execution via a web shell
- Exfiltrate data from the internal environment to external infrastructure

Given the confirmed exfiltration volume and the breadth of attack techniques observed, this incident would be classified as a **high-severity confirmed compromise** in a production environment, warranting immediate containment and forensic follow-up.

---

## 8. Recommendations

1. **Block** the source IP `198.51.100.55` at the perimeter firewall and WAF.
2. **Patch/validate** input sanitization on all application query parameters to remediate the SQL injection vector.
3. **Harden** path handling to prevent directory traversal (canonicalize and validate all file path inputs).
4. **Restrict** access to sensitive configuration and version-control paths at the web server level.
5. **Audit** the web root for unauthorized files (webshells) and rotate any potentially exposed credentials.
6. **Review** outbound firewall rules for the affected host; implement egress filtering and DLP alerting for anomalous outbound transfers.
7. **Enable** continuous SIEM alerting on the user-agent and path-based detection logic developed during this investigation.
8. **Conduct** a full forensic review of host `10.10.1.5` to rule out further compromise.

---

## 9. Conclusion

This investigation successfully identified, validated, and documented a complete attack lifecycle — from initial reconnaissance through confirmed data exfiltration — using only SIEM-based log correlation. The methodology and detection logic developed here (documented in full in the [SPL Query Library](../spl_queries)) provide a reusable detection baseline for identifying similar attack patterns in production environments.

