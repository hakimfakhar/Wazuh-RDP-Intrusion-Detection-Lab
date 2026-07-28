# MITRE ATT&CK Mapping

This table maps every stage of the attack chain to its corresponding MITRE ATT&CK tactic and technique, and records what actually detected it (or, for the one gap found, what eventually caught it).

Full detail available in **Chapter 8** of the [full report](Reports/RDP_Attack_Simulation_and_Detection_using_Wazuh_Report.pdf).

| Tactic | Technique | ID | Observed Action | Detected By |
|---|---|---|---|---|
| Reconnaissance | Active Scanning | T1595 | ICMP ping sweep | Suricata (86601) |
| Reconnaissance | Active Scanning: Scanning IP Blocks | T1595.001 | Nmap port scan | Suricata (86601, ET SCAN) |
| Discovery | Remote System Discovery | T1018 | Nmap host/port enumeration | Suricata |
| Credential Access | Brute Force: Password Guessing | T1110.001 | Hydra RDP brute force | Wazuh (60122, 100200, 100501) |
| Initial Access | External Remote Services | T1133 | RDP authentication with valid creds | Wazuh (92657, 67028) |
| Lateral Movement / C2 | Remote Services: RDP | T1021.001 | FreeRDP interactive session | Wazuh (92653), Suricata |
| Discovery | System Owner/User Discovery | T1033 | `whoami` | Wazuh (100509) |
| Discovery | System Information Discovery | T1082 | `systeminfo` | Wazuh (100512) |
| Discovery | Account Discovery: Local Account | T1087.001 | `net user` | Wazuh (92031) |
| Discovery | Permission Groups Discovery: Local | T1069.001 | `net localgroup` | Wazuh (92031) |
| **Credential Access** | **Account Discovery / Credential Manager** | **T1087** | **`cmdkey /list`** | ** Chainsaw / Sigma only — no live Wazuh rule** |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | `the_bear_ep02s01.ps1` execution | Wazuh (92203) |
| Defense Evasion / Testing | File / AV Signature Validation | — | EICAR test file delivery | Wazuh FIM (554/550) + VirusTotal (87105) |

> **Note:** `the_bear_ep02s01.ps1` was deliberately named to mimic a legitimate media-file naming convention (masquerading, MITRE ATT&CK T1036) — its actual function was to retrieve the EICAR antivirus test file.

---

Of the thirteen mapped technique instances, twelve were caught by live detection rules. The one highlighted above — Credential Manager enumeration via `cmdkey.exe`, mapped to T1087 — is the finding that pushed the offline threat-hunting pass with Chainsaw described in [Chapter 9](Reports/RDP_Attack_Simulation_and_Detection_using_Wazuh_Report.pdf).