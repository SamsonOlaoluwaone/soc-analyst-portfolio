# FINDING-001: Duplicate Log Ingestion Inflating Alert Counts (~2x)

| Field | Detail |
|---|---|
| **Type** | SIEM / log-source quality finding |
| **Affected agent** | ubuntu-server (Wazuh agent 001) |
| **Status** | **Resolved and verified** (three tests) |
| **Impact** | Alert counts doubled; a harmless test was escalated to a level-10 alert |

## Summary
Every SSH log event on ubuntu-server reached Wazuh twice, once from the systemd journal and once from `/var/log/auth.log`. Each event produced duplicate alerts. After removing the redundant source, each event produces exactly one alert.

## How I found it
1. For the IR-002 attack, the server logged 24 relevant lines for non-existent-user attempts, but Wazuh showed 48 alerts (45 rule 5710 + 3 rule 5712).
2. Identical log lines produced two alerts about 0.5 seconds apart.
3. The agent config (`/var/ossec/etc/ossec.conf`) listed both sources (`journald` at line 191 and `/var/log/auth.log` at line 215).
4. Adding the `location` field to alerts showed a 7/7 split between the two sources on an earlier test run, with the same raw line appearing once under each.

## Reproduction (baseline, before the fix)
One command, three wrong passwords as a non-existent user `dupetest`:

| Measure | Result |
|---|---|
| Log lines | 4 (1 `Invalid user` + 3 `Failed password`) |
| Wazuh alerts | **8** (4 journald + 4 auth.log) |
| High-severity alert | **Rule 5712 (level 10) fired** |

A normal login mistake was escalated to "brute force trying to get access to the system".

## Fix applied
1. Backed up the config: `sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak`
2. Removed the `auth.log` `<localfile>` block (4 lines); kept `journald`.
3. Restarted the agent: `sudo systemctl restart wazuh-agent`

Rationale for keeping `journald`: the `auth.log` copy contained summary lines (`message repeated 2 times`), which hide individual events, and the journal records each event separately.

```
$ sudo diff /var/ossec/etc/ossec.conf.bak /var/ossec/etc/ossec.conf
217,220d216
<     <location>/var/log/auth.log</location>
<   </localfile>
<   <localfile>
<     <log_format>syslog</log_format>
```
`diff` reports the smallest set of changed lines, and the neighbouring `<localfile>` blocks are nearly identical, so it aligns the removal differently from how I edited. Direct checks confirm the result: `sed -n '205,225p'` shows well-formed blocks (`dpkg.log` followed by `custom-web/access.log`) and `grep -c "auth.log"` returns 0.

## Verification (after the fix)

| Test | Log lines | Wazuh alerts | `location` | Rule 5712 |
|---|---|---|---|---|
| Baseline `dupetest` (1 connection) | 4 | **8** | journald + auth.log | Fired (8th alert) |
| After fix: `dupetest2` (1 connection) | 4 | **4** | journald only | Did not fire |
| After fix: `dupetest3` (3 connections, burst) | 12 | **12** (11 x 5710, 1 x 5712) | journald only | Fired once, on the 8th event |

**Detection still works:** the burst test raised a level-10 correlation alert, so the fix did not weaken detection.

**No loss of authentication visibility:** after removing `auth.log`, `sudo` activity still reached Wazuh via `journald` (rule 5402 "Successful sudo to ROOT executed", plus PAM session rules 5501 and 5502). The audit trail also recorded the commands I ran during this change.

## Interpretation and limits
- Rule 5712 fired on the 8th matching event in the burst test, and on the 8th alert in the duplicated baseline and in the two Oct 29 connections. This suggests a frequency threshold of about 8 events, so duplication made the rule trigger after about 4 real log lines. I have not read the rule's frequency and timeframe from its definition, so this is an inference.
- Tests were run on a single lab agent. A longer observation period would confirm no other event types were lost.

## Impact if left unfixed
- Alert volumes overstated by about 2x.
- Event-count correlation rules can trigger too early (false escalations).
- Wasted analyst time and storage.

## Rollback
```
sudo cp /var/ossec/etc/ossec.conf.bak /var/ossec/etc/ossec.conf
sudo systemctl restart wazuh-agent
```

## Related finding
Timestamps in Wazuh differed from the agent's raw log times by varying amounts on different days. This is analysed in FINDING-002 (clock synchronization). Correlation in this repository relies on source ports and PIDs rather than timestamps.

## Lessons learned
1. A SIEM is only as reliable as its log sources. Reconcile alert counts against raw logs.
2. Duplicates can change severity, not just volume.
3. Prove a fix with a before-and-after test, and record the limits of the test.
