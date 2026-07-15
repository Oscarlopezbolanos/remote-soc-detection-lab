# Remote SOC Lab: Zero Trust Detection Engineering with Wazuh & Security Onion

### Detection Engineering with Wazuh & Security Onion

![Status](https://img.shields.io/badge/Status-Active-success)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![OS](https://img.shields.io/badge/OS-Windows%2011-lightgrey)
![Security](https://img.shields.io/badge/Focus-Blue%20Team-blue)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-red)

---

# Overview

This project documents my enterprise-style cybersecurity homelab designed for Detection Engineering, Threat Hunting, Incident Response, and MITRE ATT&CK emulation.

The lab is built using VirtualBox and consists of multiple virtual machines connected through a segmented network protected by pfSense. Remote administration is performed securely using Tailscale, allowing management of the environment from anywhere without exposing services directly to the Internet.

The primary goal of this project is to simulate real-world cyber attacks and validate detections using both Endpoint Detection and Response (EDR) and Network Detection and Response (NDR) technologies.

---

# Objectives

- Build an enterprise-style Security Operations Center (SOC) lab
- Implement Zero Trust remote access using Tailscale
- Deploy enterprise firewall segmentation using pfSense
- Centralize log collection and endpoint monitoring with Wazuh
- Deploy Security Onion for network visibility and threat detection
- Generate realistic attack traffic
- Test MITRE ATT&CK techniques
- Develop detection rules
- Practice Incident Response and Threat Hunting
- Document every attack scenario

---

# Lab Architecture

```
                        Internet
                            │
                     Encrypted Tunnel
                        (Tailscale)
                            │
                  ┌───────────────────┐
                  │      pfSense      │
                  │ Firewall / Router │
                  └─────────┬─────────┘
                            │
                  192.168.120.0/24
                            │
 ┌──────────┬────────────┬────────────┬────────────┬─────────────┐
 │          │            │            │             │
 │ Kali     │ Windows    │ Ubuntu     │ Wazuh       │ Security Onion
 │ Attacker │ Victim      │ Linux      │ EDR / SIEM  │ IDS / NDR
 │          │             │ Target     │             │
 └──────────┴────────────┴────────────┴─────────────┴────────────┘
```

---

# Lab Network

Network: **192.168.120.0/24**

| Device | Hostname | Operating System | Role | IP Address |
|----------|----------|------------------|------|------------|
| pfSense | pfsense | pfSense CE | Router / Firewall | 192.168.120.254 |
| Kali Linux | kali | Kali Linux | Attacker | 192.168.120.105 |
| Windows 11 | win11 | Windows 11 Pro | Victim Workstation | 192.168.120.110 |
| Nextcloud | Nextcloud Clone | Ubuntu Server | Linux Target | 192.168.120.103 |
| Wazuh | wazuh | Ubuntu Server | SIEM / EDR | 192.168.120.102 |
| Security Onion | securityonion | Oracle Linux | NDR / IDS | 192.168.120.150 |

---

# Virtual Machines

| Machine | Purpose |
|----------|----------|
| pfSense | Firewall, Router, Network Segmentation |
| Kali Linux | Attack Platform |
| Windows 11 | Victim Workstation |
| Ubuntu | Linux Target |
| Wazuh | Endpoint Detection & Response (EDR) |
| Security Onion | Network Detection & Response (NDR) |

---

# Technologies Used

## Virtualization

- VirtualBox
- VirtualBox Host-Only Networking
- NAT Networking
- Snapshots

## Networking

- pfSense
- VLAN Concepts
- Static Routing
- Firewall Rules
- DNS
- HTTPS
- SSL/TLS

## Detection

- Wazuh
- Security Onion
- Elastic Stack
- Fleet
- Zeek
- Suricata
- Elasticsearch
- Kibana

## Remote Access

- Tailscale
- Zero Trust Networking

## Offensive Tools

- Kali Linux
- Nmap
- Hydra
- Metasploit
- SMB Enumeration
- SSH
- PowerShell
- Wireshark

---

# Zero Trust Remote Access

One of the primary objectives of this project was securely managing the lab remotely.

Instead of exposing services through public port forwarding, remote access is achieved using Tailscale.

Advantages include:

- Encrypted communication
- No inbound router ports exposed
- Secure access from anywhere
- Identity-based authentication
- Reduced attack surface

This allows me to securely access:

- pfSense
- Wazuh
- Security Onion

from any trusted device.

---

# Detection Stack

## Wazuh

Provides:

- Endpoint Detection and Response
- File Integrity Monitoring
- Log Collection
- Windows Event Monitoring
- Linux Monitoring
- Sysmon Integration
- Security Alerts

---

## Security Onion

Provides:

- Network Detection & Response
- Zeek
- Suricata IDS
- Elastic Stack
- Fleet Management
- PCAP Analysis
- Threat Hunting
- Network Visibility

---

# MITRE ATT&CK Testing

This repository will contain individual attack scenarios mapped directly to the MITRE ATT&CK Framework.

Each technique will include:

- Objective
- Attack Execution
- Detection
- Evidence
- Screenshots
- Logs
- PCAP Files
- Wazuh Alerts
- Security Onion Alerts
- MITRE Mapping
- Incident Report
- Lessons Learned

Example:

```
MITRE ATT&CK/

T1046-Network-Discovery/

T1110-Brute-Force/

T1021-Remote-Services/

T1003-Credential-Dumping/

T1059-PowerShell/

T1078-Valid-Accounts/

T1566-Phishing/
```

---

# Incident Response

Each attack will be documented using a NIST-style Incident Response workflow.

Example documentation includes:

- Executive Summary
- Initial Detection
- Timeline
- Indicators of Compromise
- MITRE ATT&CK Mapping
- Containment
- Eradication
- Recovery
- Lessons Learned

---

# Repository Structure

```
remote-soc-detection-lab/

│
├── README.md
│
├── 01-Architecture/
│     ├── Network Diagram
│     ├── IP Addressing
│     ├── VM Layout
│
├── 02-Installation/
│     ├── pfSense
│     ├── Wazuh
│     ├── Security Onion
│     ├── Tailscale
│
├── 03-Configuration/
│     ├── Firewall Rules
│     ├── Routing
│     ├── SSL
│     ├── Troubleshooting
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

- Network Security
- Firewall Administration
- Zero Trust Architecture
- Detection Engineering
- Endpoint Security
- Network Security Monitoring
- Threat Hunting
- Incident Response
- Log Analysis
- Linux Administration
- Windows Administration
- Docker
- Virtualization
- Troubleshooting
- MITRE ATT&CK Mapping

---

# Future Roadmap

- Deploy Active Directory
- Sysmon Detection Engineering
- Sigma Rules
- YARA Rules
- Malware Analysis
- Atomic Red Team Testing
- Caldera Automation
- Velociraptor
- osquery
- Custom Wazuh Rules
- Custom Suricata Rules
- Threat Hunting Playbooks
- Purple Team Exercises

---

# Author

Oscar Lopez-Bolanos

Cybersecurity Student | SOC Analyst Candidate

Columbus State University – Cybersecurity Nexus Program

CompTIA Network+

eJPT (In Progress)

Security+ (Planned)

AWS Cloud Practitioner (Planned)

---

# Project Status

🚧 Active Development

This project is continuously expanded with new attack simulations, detection rules, incident reports, and threat hunting exercises as I continue learning and improving my blue team capabilities.
