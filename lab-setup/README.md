# Home SOC Lab

## Architecture

| Host | Role | Address (host-only network) |
|---|---|---|
| Wazuh server | Manager, indexer, dashboard | 192.168.56.108 |
| ubuntu-server | Monitored endpoint (Wazuh agent 001), SSH target | 192.168.56.109 |
| Kali Linux VM | Attacker / analyst workstation, also runs Splunk | 192.168.56.106 |

Platform: VirtualBox. Two network adapters per VM: **NAT** (internet access) and **host-only** (VM-to-VM traffic, 192.168.56.x). Addresses are assigned by DHCP and can change, so I verify with `ip a` before each session.

*(Add a simple diagram image to `/evidence`.)*

## Tools
- **Wazuh:** open-source SIEM/XDR (manager, indexer, dashboard, agent)
- **Splunk:** installed; searches in progress
- **Kali Linux:** attack simulation and analysis
- **Ubuntu Server:** monitored host

## What I did
1. Deployed Wazuh and Splunk.
2. Onboarded `ubuntu-server` as a Wazuh agent.
3. Generated controlled SSH attacks from Kali and from the host itself.
4. Investigated the alerts in the Wazuh dashboard and wrote incident reports.

## Troubleshooting log (real issues I solved)

| Problem | How I diagnosed it | Cause / lesson |
|---|---|---|
| Wazuh search for my failed logins returned **no results** | Checked which host was reporting by looking at the agent name in alerts | The monitored agent was `ubuntu-server`, not the Kali VM I was testing on. **Lesson: confirm the log source is actually sending data.** |
| `ssh: Connection refused` to 192.168.56.108 | Compared "refused" (nothing listening) vs "timed out" (filtered); ran `ip a` and `ss -tlnp` on the target | I was targeting the Wazuh server. The real target was `.109`. **Lesson: verify the IP belongs to the host you think it does; DHCP addresses change.** |
| Unexpected listeners in `netstat` | Matched each port to its process (`ss -tulnp`) | Wazuh, Splunk, SSH and Tor (installed by me) were all legitimate. **Lesson: attack surface review starts with `0.0.0.0` listeners.** |

## Host exposure review
Listeners on `0.0.0.0` were reviewed first because they are reachable from the network (22 sshd, 443 dashboard, 1514 Wazuh agent comms, 55000 Wazuh API, 8000/8089 Splunk). The Tor SOCKS proxy on `127.0.0.1:9050` was traced to the Debian `tor` package that I installed, and it is bound to localhost only.
