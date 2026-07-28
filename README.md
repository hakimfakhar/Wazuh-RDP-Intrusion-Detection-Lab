# RDP Intrusion Detection Lab — Wazuh + Sysmon + Suricata + Chainsaw

I built this lab to answer one question for myself: **if I set up my own SIEM stack and then attack my own machine, will it actually catch me?**

Short answer: mostly. I got 8 out of 9 attack stages caught in real time. The 9th one — credential enumeration via `cmdkey.exe` — walked straight through my live monitoring without triggering a single alert, and I only found out because I ran an offline threat hunt against my own logs afterward.

That gap is the whole reason this repository exists. Anyone can screenshot a Wazuh alert firing. Finding the one that *didn't* fire, and figuring out why, is the part that actually taught me something.

---

## What I Actually Did

I ran a full RDP intrusion chain against my own Windows 10 VM from a Kali attacker VM within an isolated VirtualBox host-only network used for attack simulation:

```
ICMP sweep → Nmap scan → Hydra brute force → RDP login
→ whoami / systeminfo → net user / net localgroup
→ cmdkey /list → PowerShell payload drop (EICAR)
```

Every stage was monitored by a detection stack that I configured and tuned myself rather than using a pre-built lab image.

- **Wazuh** (Manager + Agent + FIM/syscheck) as the SIEM
- **Sysmon** for process creation telemetry and command-line logging
- **Suricata** for network-based detection
- **VirusTotal integration** for file hash reputation lookups
- **Chainsaw + the public Sigma ruleset** for offline threat hunting

---

## The Detection Gap

At the time I ran `cmdkey /list` to enumerate cached Windows credentials, **zero alerts fired**.

My custom rule set covered `whoami`, `systeminfo`, `net`/`net1`, `tasklist`, `ipconfig`, `route`, and `schtasks` — everything except `cmdkey.exe`.

I didn't notice the gap until I exported the Windows event log and hunted it with Chainsaw using the public Sigma rule pack, which flagged the exact same process under **Local Accounts Discovery (MITRE ATT&CK T1087)** using detection logic that I had not implemented in my live Wazuh rules.

The biggest lesson I took from this project is that my SIEM's detection coverage was never limited by the tooling itself. It was limited by the rules I had personally thought to write. Running a second, independent hunting pass helped identify what my live monitoring missed.

---

## What's in This Repository

| Folder | Contents |
|-------|-------|
| `Configuration/` | `ossec.conf`, `sysmon-integration`, `virus-total-integration` |
| `Rules/` | `local_rules.xml`, `suricata-rules.yaml` |
| `Reports/` | `RDP_Attack_Simulation_and_Detection_using_Wazuh_Report.pdf` |
| `Findings/` | `MITRE_ATTACK_Mapping.md`, `Attack_Timeline.md`, `Chainsaw_Threat_Hunting.md` |
| `SCREENSHOTS_/` |                           |
**Quick links:** [Configuration](Configuration/) · [Rules](Rules/) · [Reports](Reports/) · [Findings](Findings/) . [SCREENSHOTS_](SCREENSHOTS_/) 



The PDF report walks through every stage of the attack with alert screenshots, raw Sysmon event data, and the corresponding Wazuh detection rules. It also includes:

- Lab architecture and network diagram
- Attack timeline reconstructed from Wazuh, Suricata, and FIM telemetry
- MITRE ATT&CK mappings for every observed technique
- Detection engineering decisions and coverage gaps
- Offline threat hunting results using Chainsaw and Sigma rules

The report is intended to serve as both project documentation and an incident analysis report. If you only read one section, I recommend reading the detection gap discussed in **Chapter 6.7** and the Chainsaw hunt in **Chapter 9**.

---

## Why RDP?

RDP brute-forcing against Windows systems remains a common initial access technique. It requires no custom malware and is often followed by legitimate Windows discovery commands that blend into normal administrative activity.

That made it a good scenario for practicing layered detection based on log correlation, host telemetry, and threat hunting rather than relying solely on signature-based detections.

---

## What I'd Do Differently Next Time

- Add persistence techniques such as Scheduled Tasks and Run Registry Keys.
- Run Chainsaw hunts routinely and feed the findings back into `local_rules.xml`.
- Expand the lab to include lateral movement detection across multiple hosts.

---

## Environment

The attack simulation was performed within an isolated VirtualBox host-only network (`192.168.11.0/24`).

All attack traffic remained confined to the lab environment. Outbound Internet connectivity was enabled only where required for external services such as VirusTotal hash lookups and EICAR test file retrieval.

All attack tooling (`Hydra`, `Nmap`, and `FreeRDP`) was executed from a Kali Linux VM against a Windows 10 VM that I configured specifically for this project.
