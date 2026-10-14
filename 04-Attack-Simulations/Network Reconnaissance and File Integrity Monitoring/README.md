# SOC Investigation 001 — Network Reconnaissance and File Integrity Monitoring

**Date:** October 12, 2026  
**Environment:** Isolated, authorized VirtualBox SOC home lab  
**Status:** Completed lab validation; no confirmed compromise

## Objective
Validate that Security Onion detects network reconnaissance and Wazuh detects file integrity changes on a monitored Windows Server target. Practice pivoting from alerts to supporting event details.

## Lab assets
| Role | System | IP |
|---|---|---|
| Attacker / test source | Kali Linux | 192.168.130.105 |
| Target | Metasploitable 3, Windows Server 2008 R2 | 192.168.140.111 |
| Network monitoring | Security Onion | 192.168.120.150 (management UI) |
| Endpoint monitoring | Wazuh | 192.168.120.102 (management UI) |
| Segmentation | pfSense | Routes between lab networks |

## Test A — Network reconnaissance
An authorized Nmap scan was launched from Kali against Metasploitable 3. Security Onion / Suricata generated alerts including:
- `ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine)`
- `ET SCAN Possible Nmap User-Agent Observed`

**Example alert evidence:**
- Source: `192.168.130.105:47514`
- Destination: `192.168.140.111:8383`
- Timestamp: `2026-10-12T22:56:02.232Z` (18:56:02 EDT)
- Rule ID / signature: `2009358`
- Transport: TCP
- Decoded request: `OPTIONS / HTTP/1.1` with an Nmap Scripting Engine User-Agent and `Access-Control-Request-Method: PATCH`

Other alerts showed traffic to destination port 8020. These records confirm observed scanning behavior, not successful exploitation.

## Test B — File integrity monitoring
Wazuh agent `metasploitable3-win2k8` was active. A test directory was configured for file integrity monitoring with real-time monitoring enabled. A test file was created, modified, and deleted.

**Wazuh observations (October 12, 2026):**
| Approx. dashboard time | Rule ID | Level | Alert |
|---|---|---|---|
| 21:50 | 554 | 5 | File added to the system |
| 21:51 | 550 | 7 | Integrity checksum changed |
| 21:52 | 553 | 7 | File deleted |

The displayed alerts validate that the Wazuh agent reported file changes. They do not, by themselves, prove attribution or malicious activity. Exact affected paths and hash values should be copied from each expanded alert before finalizing the evidence record.

## Analysis and disposition
- **Network finding:** Authorized reconnaissance generated observable Suricata alerts.
- **Endpoint finding:** Wazuh FIM reported test-file creation, modification, and deletion.
- **Correlation:** Both activities targeted the same lab server, but no causal relationship between the scan and file modifications has been established.
- **Disposition:** Expected / authorized test activity. No confirmed intrusion.

## Evidence to attach
1. Nmap scan output saved as text or XML.
2. Security Onion alert details showing signature, timestamp, source/destination IP and ports.
3. Security Onion Hunt view showing related events.
4. Wazuh agent status and expanded alerts for rules 554, 550, and 553, including `syscheck.path` and available hashes.
5. A sanitized diagram of the lab network.

## Next steps
1. Expand each Wazuh FIM alert and record `syscheck.path`, `syscheck.event`, and available before/after hashes.
2. Confirm whether directory creation/deletion generates distinct events in this agent/OS configuration.
3. Run a separate, controlled service- or login-related scenario, capture network and endpoint evidence, and write a second investigation.
4. Map observed behaviors to MITRE ATT&CK only where the telemetry supports the mapping (e.g., active scanning as a behavior; do not imply compromise).

## Skills demonstrated
Network traffic analysis, IDS alert triage, endpoint file integrity monitoring, timestamp and IP correlation, event validation, documentation, and evidence-based incident disposition.
