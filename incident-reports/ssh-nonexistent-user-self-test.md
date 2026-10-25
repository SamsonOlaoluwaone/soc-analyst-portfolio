#Failed SSH Logins Using a Non-Existent User (Loopback)

| Field | Detail |
|---|---|
| **Date / time** | Oct 21, 2026, approx. 16:06 (about 35 seconds) |
| **Affected host** | ubuntu-server (Wazuh agent 001) |
| **Source IP** | 127.0.0.1 (loopback) |
| **Targeted account** | `fakeuser` (does not exist) |
| **Detection** | Wazuh rule **5710**, level 5: "sshd: Attempt to login using a non-existent user" |
| **MITRE ATT&CK** | T1110.001 Brute Force: Password Guessing (Credential Access); T1021.004 Remote Services: SSH (Lateral Movement) |
| **Severity** | Informational |
| **Verdict** | True positive detection; **benign, authorized lab test** |

## Summary
Wazuh raised at least 10 level-5 alerts within about 35 seconds for failed SSH logins as a user that does not exist on the host.

## Investigation
1. **Source:** `127.0.0.1`. The attempts originated on the server itself. A remote attacker would not normally appear as loopback, so a loopback source means a local process or user made the attempts.
2. **Account:** `fakeuser`, a non-existent account, which matches rule 5710.
3. **Speed:** about 10 events in 35 seconds, consistent with repeated quick test attempts.
4. **Outcome:** searched `rule.id: 5715` (SSH authentication success). **No results**, so no successful login.
5. **Confirmation:** I ran these attempts myself as a test.

## Evidence
```
Failed password for invalid user fakeuser from 127.0.0.1 port <port> ssh2
```
*(Replace with the raw `full_log` value and add a sanitized screenshot to `/evidence`.)*

## MITRE note
Wazuh tags are a starting point. The alert maps to T1110.001, but I verify the source and outcome before deciding the behavior is actually malicious. T1021.004 was tagged because SSH is a lateral movement channel; my test was not lateral movement.

## Recommended actions
- **Here:** none. Close as expected activity.
- **If this were a real case:** identify which process or user generated local SSH attempts, check for compromise, and review for any "Accepted" logins.

## Lesson learned
Same detection, different meaning depending on source. A loopback source made this a self-test, and a remote or unknown source would change the severity.
