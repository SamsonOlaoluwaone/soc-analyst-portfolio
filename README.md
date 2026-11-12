AKINRINADE SAMSON OLAOLUWA | SOC Analyst Portfolio

Entry-level SOC analyst candidate with an HND in Statistics, building hands-on detection and investigation skills in a home SIEM lab. I enjoy turning large volumes of log data into clear, evidence-based incident decisions.

📍 OYO STATE, NIGERIA | 📧 samson.shegun736@gmail.com | 🔗 https://www.linkedin.com/in/samson-akinrinade-ab255a433/

## About me

My statistics background trains me to separate **signal from noise**, spot anomalies against a baseline, and treat false positives as a measurable cost. I apply the same thinking to security alerts: verify the source, check the outcome, scope the impact, then document and escalate.

---

## Featured projects

| # | Project | Skills shown | Status |
|---|---------|--------------|--------|
| 1 | [Home SOC lab (Wazuh + Splunk + Kali)](lab-setup/README.md) | SIEM deployment, agent onboarding, troubleshooting log sources | ✅ Complete |
| 2 | [IR-001: SSH brute force from internal host (revised)](incident-reports/IR-002-ssh-brute-force.md) | Correlation rules, timeline building, brute force vs spraying, reconciling alert counts with raw logs | ✅ Complete |
| 3 | [FINDING-001: Duplicate log ingestion](incident-reports/FINDING-001-duplicate-log-ingestion.md) | Log-source validation, change management (backup, fix, verify, rollback) | ✅ Resolved and verified |
| 4 | [FINDING-002: Clock synchronization failure](incident-reports/FINDING-002-clock-synchronization.md) | Time-sync diagnosis, evidence integrity, measuring offsets across systems | ✅ Resolved and verified |
| 5 | [Detection notes and queries](detections/README.md) | Wazuh queries, rule 5710/5712/5760/5763 logic, field names | ✅ Complete |
| 6 | [LAB-001: Packet-level analysis of SSH](labs/LAB-001-ssh-packet-analysis.md) | Wireshark, TCP handshake, open vs. closed ports, correlating packets with logs and SIEM | ✅ Complete (screenshots pending) |
| 7 | Splunk searches and dashboard | SPL, stats, timecharts | 🔄 In progress |

---

## Skills

**SIEM:** Wazuh (manager, dashboard, agent, rule analysis), Splunk (installed; searches in progress)
**Operating systems:** Linux (Kali, Ubuntu Server), command line, `journalctl`, `ss`/`netstat`, `grep`/`awk`/`sed` log processing, service management
**Networking:** TCP/IP, ports and protocols, DNS, firewall log reading, VirtualBox host-only and NAT networks
**Network tools:** Wireshark (capture, display filters, TCP flags, conversations)
**Detection and frameworks:** MITRE ATT&CK (T1110.001, T1021.004), alert triage, true/false positive classification
**Investigation:** brute force analysis, log correlation, timeline building, host and process verification, time-zone and clock checks
**Documentation:** structured incident reports, severity rating, escalation recommendations, change records
**Learning in progress:** Splunk SPL, EDR concepts, SOAR concepts, phishing and malware analysis

---

## How I investigate (my triage workflow)

1. **Detect:** What rule fired? What is its level and meaning?
2. **Validate:** Is the activity real? Check the raw log (`full_log`), not just the dashboard summary.
3. **Scope:** Source IP, targeted accounts, number of events, time window, affected hosts. Search by source IP, not by one rule.
4. **Outcome:** Did anything succeed? (e.g. look for rule 5715 / "Accepted password" after failures.)
5. **Contain and recommend:** Block, harden, monitor.
6. **Document and escalate:** Write the report with evidence (UTC times), verdict, and next steps.


---

## Certifications and learning path

- [ ] CompTIA Security+ (target)
- [ ] TryHackMe SOC Level 1 path (in progress)
- [ ] LetsDefend SOC analyst alerts (in progress)

*All activity in this repository was performed in my own isolated lab. No real systems or third parties were involved.*
