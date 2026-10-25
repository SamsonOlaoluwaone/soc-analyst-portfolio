# [Your Name] | SOC Analyst Portfolio

Entry-level SOC analyst candidate with an HND in Statistics, building hands-on detection and investigation skills in a home SIEM lab. I enjoy turning large volumes of log data into clear, evidence-based incident decisions.

📍 [Your City, Country] | 📧 [your.email@example.com] | 🔗 [LinkedIn URL]

---

## About me

My statistics background trains me to separate **signal from noise**, spot anomalies against a baseline, and treat false positives as a measurable cost. I apply the same thinking to security alerts: verify the source, check the outcome, scope the impact, then document and escalate.

---

## Featured projects

| # | Project | Skills shown | Status |
|---|---------|--------------|--------|
| 1 | [Home SOC lab (Wazuh + Splunk + Kali)](lab-setup/README.md) | SIEM deployment, agent onboarding, troubleshooting log sources | ✅ Complete |
| 2 | [IR-001: Failed SSH logins from loopback](incident-reports/IR-001-ssh-nonexistent-user-self-test.md) | Alert triage, log reading, MITRE mapping, benign closure | ✅ Complete |
| 3 | [IR-002: SSH brute force from internal host](incident-reports/IR-002-ssh-brute-force.md) | Correlation rules, timeline building, brute force vs spraying, true/false positive reasoning | ✅ Complete |
| 4 | [Detection notes and queries](detections/README.md) | Wazuh search queries, rule 5710/5712/5715 logic | ✅ Complete |
| 5 | Wireshark: SSH handshake and traffic analysis | TCP three-way handshake, packet-level evidence | 🔄 In progress |
| 6 | Splunk searches and dashboard | SPL, stats, timecharts | 🔄 In progress |

---

## Skills

**SIEM:** Wazuh (manager, dashboard, agent, rule analysis), Splunk (installed; searches in progress)
**Operating systems:** Linux (Kali, Ubuntu Server), command line, `journalctl`, `ss`/`netstat`, service management
**Networking:** TCP/IP, ports and protocols, DNS, firewall log reading, VirtualBox host-only and NAT networks
**Detection and frameworks:** MITRE ATT&CK (T1110.001, T1021.004), alert triage, true/false positive classification
**Investigation:** brute force analysis, log correlation, timeline building, host and process verification
**Documentation:** structured incident reports, severity rating, escalation recommendations
**Learning in progress:** Wireshark, Splunk SPL, EDR concepts, SOAR concepts, phishing and malware analysis

---

## How I investigate (my triage workflow)

1. **Detect:** What rule fired? What is its level and meaning?
2. **Validate:** Is the activity real? Check the raw log (`full_log`), not just the dashboard summary.
3. **Scope:** Source IP, targeted accounts, number of events, time window, affected hosts.
4. **Outcome:** Did anything succeed? (e.g. look for rule 5715 / "Accepted password" after failures.)
5. **Contain and recommend:** Block, harden, monitor.
6. **Document and escalate:** Write the report with evidence, verdict, and next steps.

---

## Repository structure

```
├── incident-reports/   Written investigations (IR-001, IR-002)
├── lab-setup/          Architecture, setup and troubleshooting notes
├── detections/         Queries and rule logic I used
├── templates/          Incident report template
└── evidence/           Sanitized screenshots and log excerpts
```

---

## Certifications and learning path

- [ ] CompTIA Security+ (target)
- [ ] TryHackMe SOC Level 1 path (in progress)
- [ ] LetsDefend SOC analyst alerts (in progress)

*All activity in this repository was performed in my own isolated lab. No real systems or third parties were involved.*
