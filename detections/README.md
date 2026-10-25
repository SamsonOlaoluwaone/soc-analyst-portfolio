# Detection Notes

## Wazuh rules I worked with

| Rule ID | Level | Description | Type |
|---|---|---|---|
| 5710 | 5 | sshd: Attempt to login using a non-existent user | Single event |
| 5712 | 10 | sshd: brute force trying to get access to the system | **Correlation** (repeated failures from one source in a short window) |
| 5715 | - | sshd: authentication success | Used to check whether an attack **succeeded** |

*(Add the level for 5715 from your own dashboard under Rules.)*

## Searches I used (Wazuh Threat Hunting, DQL)

```
rule.id: 5710 and data.srcip: 192.168.56.106     # single failures from one source
rule.id: 5712                                    # brute-force correlation alerts
rule.id: 5715                                    # successful SSH logins (compromise check)
rule.groups: authentication_failures             # all authentication failure alerts
```

Columns added for triage: `data.srcip`, `data.srcuser`, `rule.id`, `rule.level`.

## Linux log checks

```bash
sudo journalctl -u ssh | grep -i "failed"        # failed logins
sudo journalctl -u ssh | grep -i "accepted"      # successful logins
sudo ss -tulnp                                   # listening ports with owning process
```

## Reading an SSH log line
`Failed password for invalid user admin from 192.168.56.106 port 53664 ssh2`
Who = `admin`, what = failed password, from where = source IP, session = source port.

## Brute force vs. password spraying

| | Brute force | Password spraying |
|---|---|---|
| Accounts | One or few | Many |
| Attempts per account | Many | Few |
| MITRE | T1110.001 | T1110.003 |
