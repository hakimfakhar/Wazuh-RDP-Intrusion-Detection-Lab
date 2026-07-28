# Consolidated Attack Timeline

All times shown as displayed on the Wazuh dashboard, in the victim's local time zone (UTC+01:00, Casablanca — confirmed via `systeminfo`).

Full detail available in **Chapter 10** of the [full report](Reports/RDP_Attack_Simulation_and_Detection_using_Wazuh_Report.pdf).

| Time | Source | Event |
|---|---|---|
| 19:11:44 – 19:11:50 | Suricata | ICMP ping sweep detected (7× GPL ICMP PING *NIX, rule 86601) |
| 19:13 (host time) | Attacker (Nmap) | Scan confirms TCP/3389 (RDP) open on 192.168.11.135 |
| 19:13:21 – 19:13:27 | Suricata | Nmap scan triggers ET SCAN alerts on MySQL, VNC, Oracle, PostgreSQL, MSSQL ports |
| 19:15:45 – 19:15:56 | Attacker (Hydra) | First brute-force run (4 tasks); credential validated by Windows auth but RDP session not established |
| 19:16:00 | Wazuh | Successful Remote Logon Detected (NTLM), Special privileges assigned (rules 92657 / 67028) |
| 19:16:03 | Suricata | ET SCAN – Unusually fast Terminal Server traffic (behavioral) |
| 19:20:21 – 19:20:24 | Attacker (Hydra) | Second brute-force run (1 task); credential pair `sylix:secret` confirmed with full session establishment |
| 19:20:26 – 19:21:20 | Wazuh / Suricata | Repeated successful-logon and special-privilege events; tail-end logon failures and RDP response alerts |
| 19:22:48 – 19:22:54 | Wazuh | Non-network/service local logon; RDP connection from 192.168.11.137 confirmed for `WORKGROUP\sylix` (rule 92653) |
| 19:22:49 | Attacker (FreeRDP) | Interactive RDP desktop session opened |
| 19:24:32 | Wazuh | `whoami` executed – user account discovery (rule 100509) |
| 19:38:09 | Wazuh | `systeminfo` executed – system information discovery (rule 100512) |
| 19:40:26 | Wazuh | `net user` / `net localgroup` executed – discovery activity (rule 92031) |
| ~19:39 – 19:42 (UTC) | Chainsaw (offline) | `net.exe` and `cmdkey.exe` executions matched to Sigma rules T1018 / T1087 |
| 19:49:00 – 19:50:33 | FIM | `the_bear_ep02s01.ps1` written to Downloads then Public (rule 554) |
| 19:56:48 – 19:58:57 | FIM | `eicar.com` added to Downloads, modified under Public (rules 554 / 550) |
| 19:59 | Wazuh (VT integration) | VirusTotal: 62 engines detect `eicar.com`; PowerShell-created-executable alert (rules 87105 / 92203) |
| 20:01:52 | FIM | Final registry key integrity checksum change recorded (rule 594) |
| 20:13 | Analyst (Chainsaw) | Offline Sigma hunt executed against exported `.evtx`; confirms `cmdkey.exe` detection gap |

---

**Note on the Chainsaw timestamp offset:** Chainsaw parses raw `.evtx` timestamps in UTC, while the Wazuh dashboard renders alert times in the host's local zone (UTC+01:00, Casablanca). Once that one-hour offset is accounted for, the Chainsaw-derived timestamps align closely with the corresponding `net`/`cmdkey` activity observed live in Wazuh — confirming both tools were looking at the same underlying events. The difference was detection coverage, not timing.