# Detection Notes

## Wazuh rules I worked with

| Rule ID | Level | Description | Type |
|---|---|---|---|
| 5710 | 5 | sshd: Attempt to login using a non-existent user | Single event |
| 5712 | 10 | sshd: brute force trying to get access to the system. Non existent user. | **Correlation** (repeated failures from one source in a short window) |
| 5715 | 3 | sshd: authentication success | Used to check whether an attack **succeeded** |
| 5760 | 5 | sshd: authentication failed | Failed password for an **existing** account |
| 5763 | 10 | sshd: brute force trying to get access to the system. Authentication failed. | Correlation for existing accounts |
| 5503 | 5 | PAM: User login failed | PAM authentication failure line |
| 5551 | 10 | PAM: Multiple failed logins in a small period of time | Correlation on PAM failures |
| 2502 | 10 | syslog: User missed the password more than one time | `PAM n more authentication failures` line |
| 5402 | 3 | Successful sudo to ROOT executed | Audit trail of privileged commands |


## Field names (check an expanded alert first)
- `data.srcuser`: targeted account when it does **not** exist
- `data.dstuser`: targeted account when it **does** exist
- `location`: where the agent read the event (e.g. `journald`, `/var/log/auth.log`)

**Rule of thumb:** when a filter returns nothing, search by `data.srcip` or `full_log` before concluding nothing happened, and check the time range.

## Searches I used (Wazuh Threat Hunting, DQL)

```
rule.id: 5710 and data.srcip: 192.168.56.106     # single failures from one source
rule.id: 5712                                    # brute-force correlation alerts
rule.id: 5715                                    # successful SSH logins (compromise check)
rule.groups: authentication_failures             # all authentication failure alerts
full_log: *root*                                 # attempts against the root account
```

Columns added for triage: `data.srcip`, `data.srcuser`, `data.dstuser`, `rule.id`, `rule.level`, `full_log`, `location`.

## Linux log checks

```bash
sudo journalctl -u ssh | grep -i "failed"        # failed logins
sudo journalctl -u ssh | grep -i "accepted"      # successful logins
sudo ss -tulnp                                   # listening ports with owning process
sudo sshd -T | grep -i permitrootlogin           # is root password login allowed?
timedatectl                                      # time zone and NTP status
```

## Counting per connection and per account

```bash
# failed passwords per connection (PID)
sudo journalctl -u ssh | grep "Failed password" | awk '{print $5}' | sort | uniq -c
# failed passwords per account
sudo journalctl -u ssh | grep "Failed password" | sed -E 's/.*Failed password for (invalid user )?([^ ]+) from.*/\2/' | sort | uniq -c
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
