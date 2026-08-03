# Investigation Notes

Analyst-style working notes documenting the reasoning behind each investigative step, in the order they were performed. These notes reflect my own decision-making process while working through the SIEM data — not a prescribed script.

---

## Phase 1 — Environment Familiarization

Before writing any detection logic, I ran a broad baseline search (`index=main sourcetype=web_traffic`) to understand what fields were available and get a feel for normal traffic volume and shape. Key fields identified: `client_ip`, `path`, `method`, `status`, `user_agent`, `referer`, `size_bytes`.

**Why this matters:** you can't detect an anomaly without first understanding the baseline. Skipping this step is a common mistake that leads analysts to chase false positives.

## Phase 2 — Volume-Based Anomaly Detection

Using `timechart span=1d count`, I plotted daily event volume across the full dataset and sorted it to find the highest-traffic day. One date stood out well above the rest of the distribution: **2025-10-12**. This became my anchor point for the rest of the investigation — everything downstream was cross-referenced against this date.

**Analyst takeaway:** volume spikes alone aren't proof of malicious activity, but they're an efficient way to narrow a large dataset down to a window worth investigating further.

## Phase 3 — Behavioral Fingerprinting via User-Agent Analysis

Legitimate browser traffic almost always presents a recognizable user-agent string (Mozilla, Chrome, Safari, Firefox). I excluded all four of these substrings from the search and reviewed what remained. The result was a small, concentrated set of non-browser clients — scripts, tools, and command-line utilities — that stood out clearly against the noise of normal traffic.

**Why this technique works:** attackers using automated tooling rarely bother spoofing a realistic browser user-agent unless they're specifically trying to evade detection. This makes user-agent filtering a fast, high-signal triage technique.

## Phase 4 — Actor Isolation

Aggregating the filtered results with `stats count by client_ip` and sorting descending immediately surfaced one IP address responsible for the overwhelming majority of anomalous requests: **198.51.100.55**. From this point forward, the investigation was scoped specifically to activity originating from this address.

## Phase 5 — Reconnaissance Detection

I searched for requests from the identified IP targeting common sensitive paths — environment files, diagnostic/info pages, and version-control directories. Multiple hits confirmed the actor was performing automated reconnaissance, likely using a path/file discovery tool, before attempting exploitation.

## Phase 6 — Path Traversal Detection

Pattern-matching for directory traversal sequences and redirect-style parameters within the `path` field, scoped to the attacker IP, confirmed repeated path traversal attempts. Reviewing the top values for the `path` field made the traversal payloads (and their frequency) immediately visible.

## Phase 7 — Path Traversal Frequency Analysis

Aggregating the traversal-pattern matches by path (`stats count by path`) quantified the scale of this behavior: **658 distinct path traversal attempts** were recorded from the attacker IP.

## Phase 8 — SQL Injection Detection

Filtering requests from the attacker IP where the user-agent matched known SQL injection tooling signatures (Havij, sqlmap) returned a large, structured set of results. The `user_agent` field breakdown (Top Values) confirmed **993 requests** carrying the Havij automated SQL injection signature, alongside a smaller volume attributed to sqlmap.

## Phase 9 — Post-Exploitation / Staging Detection

I searched for requests referencing archive-style filenames (backup/log archives) and webshell-style command parameters, scoped to the attacker IP. Matches here indicated the actor had moved past exploitation and was attempting to stage data and/or execute commands via an uploaded web shell.

## Phase 10 — Firewall Correlation

To validate whether any of this activity resulted in actual data movement, I pivoted to `sourcetype=firewall_logs` and searched for allowed connections between the internal host (`10.10.1.5`) and the attacker IP. All matching connections were permitted (`action=ALLOWED`), which meant no network-layer control had stopped the traffic.

## Phase 11 — Exfiltration Volume Confirmation

Running `stats sum(bytes_transferred) by src_ip` against the allowed firewall connections returned a total of **126,167 bytes** transferred from the internal host to the attacker's infrastructure. This provided concrete, quantifiable confirmation that data left the environment.

## Phase 12 — Timeline & Reporting

With every technique confirmed and quantified, I reconstructed the full sequence of events chronologically (see [`timeline.md`](timeline.md)) and compiled the findings into a formal incident report (see [`incident_report.md`](incident_report.md)) with MITRE ATT&CK mapping and remediation recommendations.

---

## Reflection

The value of this exercise wasn't just running queries — it was learning to let each result inform the next query. Starting from a volume anomaly and narrowing step-by-step to a fully confirmed, quantified compromise is the core analytical skill a SOC analyst needs, regardless of which SIEM platform they're using.
