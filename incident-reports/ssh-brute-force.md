# SSH Brute-Force Attempt Against ubuntu-server

| Field | Detail |
|---|---|
| **Date / time** | Oct 22, 2026, 19:02 to 19:07 (about 5 minutes) |
| **Affected host** | ubuntu-server, 192.168.56.109 (Wazuh agent 001) |
| **Source IP** | 192.168.56.106 (internal lab host, Kali VM) |
| **Targeted accounts** | `admin`, `test`, `oracle` (none exist on the host) |
| **Detections** | Rule **5710** (level 5) x 45 alerts; correlation rule **5712** (level 10) x 3 alerts |
| **MITRE ATT&CK** | T1110.001 Brute Force: Password Guessing (Credential Access) |
| **Severity** | Low (no successful login; internal source) |
| **Verdict** | **True positive detection; benign authorized security test** |

## Summary
A single internal host made repeated failed SSH login attempts against three non-existent usernames. Wazuh raised 45 level-5 alerts and three level-10 brute-force correlation alerts. No successful login followed.

## Timeline

| Username | Burst window | Rule 5710 alerts | Rule 5712 fired |
|---|---|---|---|
| admin | 19:02:33 to 19:03:18 | 15 | 19:02:54 |
| test | 19:05:39 to 19:06:29 | 15 | 19:05:57 |
| oracle | 19:06:57 to 19:07:47 | 15 | 19:07:21 |

## Analysis
- **Rate:** 45 alerts in about 5 minutes, roughly 9 per minute. This pace points to automation or rapid repeated attempts.
- **Pattern:** attempts went through **one username at a time** (about 15 alerts each). This is consistent with **brute force (T1110.001)** rather than password spraying (T1110.003), which would rotate usernames while trying few passwords per account.
- **Why only 3 level-10 alerts:** rule 5712 is a correlation rule. It fires once per burst after enough 5710 events occur from the same source in a short window, so 45 single failures produced 3 correlated alerts.
- **Alert count vs. attempts:** events appeared in pairs less than a second apart, so the real attempt count may be lower than 45. *(To verify: count "Failed password" lines in the host's auth log.)*
- **Outcome:** searched `rule.id: 5715` from this source: **no results**. No successful authentication, no evidence of compromise.
- **Attribution:** the source is my own Kali VM, run as an authorized lab test.
- Wazuh dashboard displayed times one hour ahead of the server logs (browser UTC+1 vs server UTC). All times in this report are UTC.”

## Evidence
```
Oct 22 18:07:46 ubuntuserver sshd[63541]: Failed password for invalid user oracle from 192.168.56.106 port 44786 ssh2
```
*(Add the raw `full_log` line and sanitized screenshots to `/evidence`.)*

![evidence](evidence/rawlog.png.png)

## Recommended actions (if this were a real external attacker)

**Immediate**
1. Block the source IP at the firewall.
2. Search for any successful login (rule 5715 / "Accepted password") from that IP over the last 24 hours.
3. Check whether other hosts saw the same source.

**Preventive**
4. Disable SSH password authentication and use key-based authentication.
5. Install and configure fail2ban.
6. Restrict port 22 to known management IPs using `ufw`.

**Escalation**
7. In production, escalate to the server owner if the source is external or if any successful login is found.

## True positive vs. benign
*The detection was correct* (true positive), but *the activity was not malicious* (authorized test). Both facts go in the ticket: the first confirms the rule works, and the second determines the response.

## Lessons learned
- Repetition changes meaning: single failures are noise, patterns are detections.
- Always read rule ID and level from the same alert row to avoid reporting errors.
- Counts need filters: a number is only meaningful with its source, rule, and time range.
- Confirmed the server time zone with timedatectl before building the timeline, since mismatched time zones can misorder events across systems
