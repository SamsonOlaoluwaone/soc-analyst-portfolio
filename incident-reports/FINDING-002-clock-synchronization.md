# FINDING-002: Clock Synchronization Failure Between SIEM Manager and Agent

| Field | Detail |
|---|---|
| **Type** | SIEM / log-source quality finding |
| **Affected systems** | Wazuh manager host (primary), Kali VM (clock accurate but not NTP-locked) |
| **Status** | **Resolved and verified** |
| **Impact** | Alert timestamps differed from the agent's log times by seconds, minutes, and once by about 18 hours, making multi-system timelines unreliable |

## Summary
Comparing raw log times on `ubuntu-server` (the agent) with the timestamps Wazuh stamped on the same events showed offsets that changed from day to day. The manager host had no working time synchronization, and at times no internet connection, so its clock was not trustworthy. After restoring connectivity and NTP, the manager and the other lab hosts agree to within about one second.

## How I found it
While matching Wireshark captures, server logs, and Wazuh alerts for LAB-001, the same events carried very different times in the logs and in the dashboard. All times below are agent log time minus manager alert time, after removing the dashboard's UTC+1 display offset.

| Date | Offset (agent minus manager) |
|---|---|
| Oct 21 | agent about **21 minutes** ahead |
| Oct 22 | within about 1 second |
| Oct 29 | agent about **18 hours 3 minutes** behind |
| Nov 4 | agent about 3 seconds ahead |
| Nov 8 | agent about 5 seconds ahead |

The spacing between alerts matched the spacing between log lines, so the cause was a constant clock difference and not delayed delivery.

## Measurements
Clock state of each host:

| Host | Date | NTP / synchronized | Observation |
|---|---|---|---|
| ubuntu-server (agent) | Nov 7 to 10 | active / **yes** | Within 1 second of Kali on Nov 10 |
| Wazuh manager host | Nov 7 | inactive / **no** | Hardware clock and system clock 25 s apart |
| Wazuh manager host | Nov 8 | active / **no** | `Packet count: 0`; every NTP request to `91.189.91.157:123` timed out |
| Wazuh manager host | Nov 10 | active / no (no internet) | `ping 1.1.1.1` 100% loss, DNS failed; clock unsynchronized after a reboot |
| Kali VM | Nov 8 | active / no | Time zone New York; hardware clock about 3 days behind the system clock; system clock accurate within 1 s of web time on Nov 10 |

The manager's service log also showed several reboots (Nov 5, Nov 8, and two on Nov 10). After each boot a host starts from its hardware clock until time synchronization succeeds.

## Fix applied
1. Enabled NTP: `sudo timedatectl set-ntp true`.
2. Restored the manager host's internet connection.
3. Configured preferred time servers in `/etc/systemd/timesyncd.conf` (backup kept as `timesyncd.conf.bak`). On the first successful sync the manager used the fallback server `ntp.ubuntu.com`.

## Verification
| Test | Result |
|---|---|
| `timedatectl` on the manager (Nov 10, 11:30 UTC) | `System clock synchronized: yes` |
| One-line comparison from Kali against the manager's dashboard response header | **11:32:31 / 11:32:31 / 11:32:31** (agreement within 1 s) |
| Controlled alert test `timetest2` (Nov 11): log time vs alert time | Alert 0.3 s and 1.4 s **after** the log line (dashboard UTC+1 removed) |

### Before and after (controlled SSH test, one wrong password)

| | Before fix (`timetest1`, Nov 8) | After fix (`timetest2`, Nov 11) |
|---|---|---|
| Agent log time (UTC) | 21:12:23 | 23:15:38 and 23:15:43 |
| Alert time (UTC) | 21:12:17.8 | 23:15:38.3 and 23:15:44.4 |
| Alert minus log | **-5.2 s** (alert earlier than its own log line) | **+0.3 s and +1.4 s** (normal delivery delay) |

An alert stamped before its source log line cannot happen when clocks agree, so the earlier result showed a clock offset. After the fix the alert falls just after the log line. Both alerts show `location: journald` (no duplicates, FINDING-001 fix still holding), and rule 5712 did not fire, as expected for a single attempt.

## Interpretation and limits
- The leading hypothesis is that the manager's clock ran unsynchronized across reboots, so its offset relative to the synchronized agent wandered. I cannot prove which clock was wrong on Oct 21 or Oct 29, and the sign of the offset changed between those dates.
- The web `date:` header has one-second resolution, so agreement can only be shown to about a second.
- After the fix the alert time falls 0.3 to 1.4 s after the log time. Below about one second this method cannot separate a small clock error from normal delivery delay.
- The 18-hour offset is not fully explained.
- The cause of the manager host's temporary loss of internet was not recorded.

## Impact if left unfixed
- Timelines built across systems can place events hours out of order.
- Time-window correlation rules can fire early, late, or not at all.
- Evidence timestamps are harder to defend in an incident review.

## Recommendations
1. Keep NTP enabled and monitored on every host, and alert when a host reports `synchronized: no`.
2. Document the time zone of every log source, and report incident times in UTC.
3. Set the dashboard's display time zone to UTC (Dashboards Management, Advanced settings) so analysts see the same hours as the logs.
4. Correlate evidence on connection identifiers (source port, PID) as a second key, not on timestamps alone.

## Lessons learned
1. Check clock synchronization before building any multi-system timeline.
2. "Synchronized: no" does not mean the clock is wrong, and "NTP active" does not mean it is working. Measure the offset.
3. Correct my own earlier assumptions: I first suspected UDP 123 was blocked; the service log showed NTP worked at one boot, and the real problem was loss of connectivity.
