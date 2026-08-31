# Remote SOC Detection Lab: Segmented EDR/NDR & Zero Trust Architecture

### Attack Simulation • Detection Engineering • Threat Hunting • Incident Response

![Status](https://img.shields.io/badge/Status-Active-success)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![Firewall](https://img.shields.io/badge/Firewall-pfSense-red)
![Security](https://img.shields.io/badge/Focus-Blue%20Team-blue)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-red)

---

# Overview

This project documents my cybersecurity homelab designed to practice Security Operations Center (SOC) workflows, Detection Engineering, Threat Hunting, Incident Response, network segmentation, and MITRE ATT&CK-based attack simulation.

The environment is built in VirtualBox and uses pfSense as the central firewall and router between multiple isolated network segments.

Rather than operating as a flat network, the lab separates security infrastructure, attacker systems, and victim systems into dedicated subnets.

The current architecture consists of:

- Security / Management Network — `192.168.120.0/24`
- Attacker Network — `192.168.130.0/24`
- Victim Network — `192.168.140.0/24`

Traffic between these networks must traverse pfSense, allowing firewall policies, routing controls, segmentation, logging, and containment to be tested.

Wazuh provides centralized endpoint and security telemetry while Security Onion provides network security monitoring capabilities.

Remote administration is performed using Tailscale without exposing management services directly to the public Internet.

The primary goal of the lab is no longer simply to execute attacks.

The goal is to follow the complete SOC investigation lifecycle:

**Attack → Detect → Investigate → Correlate → Contain → Report**

---

# Objectives

- Build a segmented SOC detection and investigation environment
- Separate attacker, victim, and security infrastructure
- Route inter-network traffic through pfSense
- Implement firewall policies between security zones
- Implement Zero Trust-style remote administration using Tailscale
- Centralize endpoint telemetry and alerts with Wazuh
- Deploy Security Onion for network security monitoring
- Generate controlled attack traffic from Kali Linux
- Deploy intentionally vulnerable systems for exploitation testing
- Test MITRE ATT&CK techniques
- Investigate alerts instead of only generating them
- Correlate endpoint, network, and firewall evidence
- Develop detection rules
- Practice threat hunting
- Create incident timelines
- Write structured incident reports
- Document lessons learned from each investigation

---

# Lab Architecture

```text
                              Internet
                                 │
                           VirtualBox NAT
                                 │
                         ┌───────────────┐
                         │    pfSense    │
                         │ Firewall /    │
                         │    Router     │
                         └───────┬───────┘
                                 │
             ┌───────────────────┼───────────────────┐
             │                   │                   │
             │                   │                   │
    SECURITY / MANAGEMENT      ATTACKER            VICTIMS
      192.168.120.0/24      192.168.130.0/24    192.168.140.0/24
             │                   │                   │
      pfSense .120.254     pfSense .130.254    pfSense .140.254
             │                   │                   │
       ┌─────┴─────┐             │             ┌─────┴───────────┐
       │           │             │             │                 │
     Wazuh     Security Onion   Kali         Windows        Nextcloud
   .120.102      .120.150     .130.105      .140.110       .140.103
                                                  │
                                                  │
                                           Metasploitable 3
                                              Planned Target
```

---

# Network Segmentation

The lab uses three isolated VirtualBox Host-Only networks.

## Security / Management Network

**Network:** `192.168.120.0/24`

**pfSense Gateway:** `192.168.120.254`

This network contains security monitoring and management infrastructure.

| Device | Role | IP Address |
|---|---|---|
| pfSense | Gateway / Firewall | `192.168.120.254` |
| Wazuh | SIEM / Endpoint Security Monitoring | `192.168.120.102` |
| Security Onion | Network Security Monitoring / NDR | `192.168.120.150` |

---

## Attacker Network

**Network:** `192.168.130.0/24`

**pfSense Gateway:** `192.168.130.254`

This network contains systems used to generate controlled malicious traffic.

| Device | Role | IP Address |
|---|---|---|
| pfSense | Gateway / Firewall | `192.168.130.254` |
| Kali Linux | Attack Platform | `192.168.130.105` |

Kali cannot communicate with victim systems at Layer 2.

Traffic destined for the victim network must be routed through pfSense.

---

## Victim Network

**Network:** `192.168.140.0/24`

**pfSense Gateway:** `192.168.140.254`

This network contains systems used for attack simulation and investigation.

| Device | Role | IP Address |
|---|---|---|
| pfSense | Gateway / Firewall | `192.168.140.254` |
| Windows | Windows Victim Workstation | `192.168.140.110` |
| Nextcloud Clone | Ubuntu / Linux Target | `192.168.140.103` |
| Metasploitable 3 | Vulnerable Attack Target | Planned |

The Nextcloud system used in this segment is a lab clone and not the primary production instance.

---

# pfSense Interface Architecture

pfSense acts as the routing and security boundary between all lab networks.

| pfSense Interface | Virtual Network | Address | Purpose |
|---|---|---|---|
| WAN (`le0`) | VirtualBox NAT | DHCP | Internet Access |
| LAN (`em0`) | Attacker Network | `192.168.130.254/24` | Kali / Attack Infrastructure |
| OPT1 (`em1`) | Security Network | `192.168.120.254/24` | Security & Management |
| OPT2 (`le1`) | Victim Network | `192.168.140.254/24` | Vulnerable / Victim Systems |

This architecture forces traffic between the attacker and victim networks through pfSense.

Example attack path:

```text
Kali
192.168.130.105
        │
        │ Attack Traffic
        ▼
pfSense
192.168.130.254
        │
        │ Routing + Firewall Policy
        ▼
pfSense
192.168.140.254
        │
        ▼
Victim
192.168.140.x
```

This allows network access policies, logging, segmentation, and containment controls to be tested during attack simulations.

---

# Firewall Segmentation

pfSense controls communication between the three security zones.

The victim network currently contains an outbound rule allowing lab systems to communicate through pfSense while connectivity and detection capabilities are being developed.

Future firewall policies will further restrict communication between:

```text
Attacker → Victim
Victim → Security Infrastructure
Victim → Internet
Attacker → Security Infrastructure
```

This will allow attack scenarios to test both successful exploitation and firewall-based containment.

---

# Virtual Machines

| Machine | Purpose |
|---|---|
| pfSense | Firewall, Routing, Segmentation and Network Policy |
| Kali Linux | Attack Platform |
| Windows | Endpoint Detection / Victim Workstation |
| Nextcloud Clone | Linux Server / Victim Target |
| Metasploitable 3 | Intentionally Vulnerable Attack Target |
| Wazuh | SIEM, Endpoint Telemetry and Security Monitoring |
| Security Onion | Network Security Monitoring, IDS and Threat Hunting |

---

# Detection Architecture

The lab is designed to investigate attacks from multiple telemetry perspectives.

## Endpoint / Host Telemetry

Wazuh collects and analyzes endpoint telemetry including:

- Windows Event Logs
- Linux Logs
- Authentication Events
- File Integrity Monitoring
- Security Events
- Sysmon Telemetry
- Process Activity
- Endpoint Alerts

Windows Sysmon provides additional visibility into activity such as:

- Process Creation
- Network Connections
- Command Lines
- Parent / Child Process Relationships
- File Activity
- Process Hashes

---

## Network Telemetry

Security Onion provides network-focused security monitoring capabilities including:

- Zeek
- Suricata
- IDS Alerts
- Network Metadata
- Elastic Stack
- Threat Hunting
- PCAP Analysis
- Network Visibility

Security Onion monitoring placement and visibility across the segmented attacker and victim networks is currently being validated as part of the lab development process.

---

## Firewall Telemetry

pfSense provides another investigation perspective by recording traffic crossing security boundaries.

This creates three potential evidence sources during an investigation:

```text
                    ATTACK
                      │
            ┌─────────┼─────────┐
            │         │         │
            ▼         ▼         ▼

          Wazuh    pfSense   Security Onion

        Endpoint   Firewall      Network
        Evidence   Evidence      Evidence
```

The objective is to correlate these data sources and reconstruct attacker activity.

---

# Investigation Methodology

Each attack scenario will be treated as a simulated security incident.

Instead of stopping after successful exploitation, the activity will be investigated from the defender's perspective.

The workflow is:

```text
1. Generate controlled attack activity

              ↓

2. Detect suspicious behavior

              ↓

3. Investigate endpoint telemetry

              ↓

4. Investigate network telemetry

              ↓

5. Review firewall evidence

              ↓

6. Correlate timestamps and events

              ↓

7. Identify Indicators of Compromise

              ↓

8. Map activity to MITRE ATT&CK

              ↓

9. Determine containment/remediation

              ↓

10. Write Incident Report
```

---

# MITRE ATT&CK Testing

Attack scenarios will be mapped to the MITRE ATT&CK framework.

Planned and tested techniques may include:

```text
T1046 - Network Service Discovery

T1110 - Brute Force

T1021 - Remote Services

T1059 - Command and Scripting Interpreter

T1078 - Valid Accounts

T1003 - OS Credential Dumping

T1566 - Phishing
```

Each scenario may include:

- Attack objective
- Attack execution
- Initial detection
- Wazuh evidence
- Security Onion evidence
- pfSense evidence
- Screenshots
- Logs
- PCAP data when available
- Indicators of Compromise
- MITRE ATT&CK mapping
- Investigation timeline
- Containment recommendations
- Incident report
- Lessons learned

---

# Incident Response

Attack scenarios will be documented using a structured Incident Response methodology.

Reports may contain:

- Executive Summary
- Incident Scope
- Initial Detection
- Investigation
- Timeline
- Indicators of Compromise
- MITRE ATT&CK Mapping
- Containment
- Eradication
- Recovery
- Recommendations
- Lessons Learned

The goal is to practice explaining not only **what attack was executed**, but also:

> **What evidence proves what the attacker did?**

---

# Zero Trust Remote Administration

Remote management of the lab is performed using Tailscale.

Instead of exposing administrative interfaces directly to the public Internet, Tailscale provides encrypted authenticated connectivity between trusted devices and the lab environment.

Remote administration capabilities include access to:

- pfSense
- Wazuh
- Security Onion

Benefits include:

- Encrypted connectivity
- No direct public exposure of management interfaces
- Identity-based access
- Reduced attack surface
- Remote SOC administration

---

# Technologies Used

## Virtualization

- VirtualBox
- Host-Only Networks
- NAT
- Snapshots

## Network Security

- pfSense
- Network Segmentation
- Static IPv4 Addressing
- Inter-Subnet Routing
- Firewall Policy
- NAT
- DNS
- HTTPS/TLS

## Detection & Monitoring

- Wazuh
- Security Onion
- Sysmon
- Zeek
- Suricata
- Elastic Stack
- PCAP Analysis

## Remote Access

- Tailscale

## Offensive Security

- Kali Linux
- Nmap
- Metasploit Framework
- Hydra
- SMB Enumeration
- SSH
- PowerShell
- Wireshark
- Metasploitable

---

# Repository Structure

```text
remote-soc-detection-lab/

│
├── README.md
│
├── 01-Architecture/
│   ├── Network-Diagram.md
│   ├── IP-Addressing.md
│   └── VM-Layout.md
│
├── 02-Installation/
│   ├── pfSense.md
│   ├── Wazuh.md
│   ├── Security-Onion.md
│   └── Tailscale.md
│
├── 03-Configuration/
│   ├── Firewall-Rules.md
│   ├── Routing.md
│   ├── SSL.md
│   └── Troubleshooting.md
│
├── 04-MITRE-ATTACK/
│
├── 05-Incident-Reports/
│
├── 06-Detection-Rules/
│
├── 07-PCAPs/
│
├── 08-Screenshots/
│
└── 09-Lessons-Learned/
```

---

# Skills Demonstrated

- SOC Operations
- Network Security
- Network Segmentation
- Firewall Administration
- Routing
- Zero Trust Remote Access
- Detection Engineering
- Endpoint Telemetry Analysis
- Network Security Monitoring
- Threat Hunting
- Incident Response
- Log Analysis
- Sysmon Analysis
- Linux Administration
- Windows Administration
- Docker
- Virtualization
- Troubleshooting
- MITRE ATT&CK Mapping
- Offensive / Defensive Security Correlation

---

# Current Development Phase

The infrastructure and segmentation phase of the lab is largely complete.

Current development is shifting toward:

**Detection → Investigation → Incident Response**

The next phase will focus on introducing intentionally vulnerable systems and generating controlled attack activity from the attacker network.

The objective will be to determine how effectively the attack can be reconstructed using endpoint, network, and firewall telemetry.

---

# Future Roadmap

- Deploy Metasploitable 3
- Install Wazuh Agent on compatible victim systems
- Expand Sysmon telemetry
- Validate Security Onion visibility between attacker and victim networks
- Perform controlled exploitation scenarios
- Create investigation timelines
- Develop custom Wazuh detections
- Develop Sigma rules
- Develop YARA rules
- Develop custom Suricata rules
- Create threat hunting playbooks
- Test Atomic Red Team
- Explore Velociraptor
- Explore osquery
- Purple Team exercises
- Active Directory attack/detection lab
- Build a library of incident reports

---

# Author

**Oscar Lopez-Bolanos**

Cybersecurity Student | SOC Analyst Candidate

Columbus State University — Cybersecurity Nexus Program

**Certifications:**
- CompTIA Network+
- eJPT

**Currently Studying:**
- CompTIA Security+
- AWS Cloud Practitioner

---

# Project Philosophy

Building the infrastructure is only the beginning.

The purpose of this lab is to develop the ability to answer the questions a SOC analyst must answer during a real investigation:

**What happened?**

**When did it happen?**

**Which systems were affected?**

**What evidence supports the conclusion?**

**How should the incident be contained?**

**How can it be detected faster next time?**
