# Threat Hunting with Chainsaw

To validate live Wazuh detection coverage against an independent rule set, I exported the Windows Security/Sysmon event log covering the incident window and hunted it offline with **Chainsaw**, using the full public Sigma rule collection.

## Command Used

```bash
.\chainsaw.exe hunt "C:\Program Files\chainsaw\sya\LogvV3.evtx" \
  -s "C:\Program Files\chainsaw\sigma\rules" \
  --mapping "C:\Program Files\chainsaw\mappings\sigma-event-logs-all.yml" \
  --output "C:\Program Files\chainsaw\chainsaw_report.txt"
```

## Hunt Summary

| Metric | Value |
|---|---|
| Sigma rules loaded | 2,912 (231 rules failed to load) |
| Artifact hunted | Exported `.evtx` (4.1 MiB) |
| Total detections | 1,185, across 795 documents |
| Relevant match reviewed | `cmdkey.exe` (T1087) |
| New coverage gap confirmed | `cmdkey.exe` execution — no live Wazuh rule |

## The Finding

Chainsaw matched the `cmdkey.exe` process-creation event against the **Local Accounts Discovery** Sigma rule, correctly classifying it under **T1087 (Account Discovery)** — a technique my live Wazuh rule set had no coverage for at the time.

This confirmed what I'd already found in the main report: the `cmdkey /list` execution generated **zero** alerts in Wazuh when it ran, because I hadn't written a rule for it. Chainsaw's Sigma-based hunt caught it retroactively by matching the process image name against a public detection rule that my live rule set simply didn't include.

**Why this matters:** live SIEM coverage only reflects the sum of the rules I've actually written. A second, independently maintained rule source — Sigma, via a hunting tool like Chainsaw — is a genuinely practical, low-cost way to catch what live coverage misses, provided the hunt is run regularly and its findings are fed back into the live rule set.

With 1,185 total detections against only 795 documents, the hunt surfaced a lot of routine administrative and system activity alongside this finding — expected behavior for a broad, un-tuned public Sigma pack run against a full event log. Review here was scoped to the one match directly tied to the confirmed attack chain.

Full detail available in **Chapter 9** of the [full report](../Reports/RDP_Attack_Simulation_and_Detection_using_Wazuh_Report.pdf).