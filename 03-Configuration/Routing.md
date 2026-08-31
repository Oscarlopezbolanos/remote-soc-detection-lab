# Routing

## Overview

All systems in the lab communicate through pfSense.

Rather than relying on VirtualBox networking alone, every virtual machine uses pfSense as its default gateway.

This creates a realistic enterprise routing environment.

---

# Network Topology

# Network

Lab Network

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
| Metasploitable 3 | Vulnerable Attack Target | `192.168.140.111` |

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



Gateway

192.168.120.254, 
192.168.130.254, 
192.168.140.254, 

---

# Device Routing

| Device | IP Address | Gateway |
|----------|------------|----------|
| Kali | 192.168.130.105 | 192.168.120.254 |
| Windows 11 | 192.168.140.110 | 192.168.120.254 |
| Nextcloud Ubuntu | 192.168.140.103 | 192.168.120.254 |
| Wazuh | 192.168.120.102 | 192.168.120.254 |
| Security Onion | 192.168.120.150 | 192.168.120.254 |
| Metasploitable 3 | 192.168.140.111 | 192.168.140.254 |

---

# Internet Access

Internet connectivity is provided through the VirtualBox NAT adapter connected to pfSense.

Traffic Flow

Internet

↓

VirtualBox NAT

↓

pfSense

↓

Lab Systems

---

# Internal Communication

Every system communicates over the internal 192.168.120.0/24 network.

Examples include:

Kali → Windows

Windows → Wazuh

Ubuntu → Wazuh

Kali → Security Onion

Windows → Security Onion

---

# Remote Access

Remote management is performed through Tailscale.

Traffic Flow

Laptop

↓

Tailscale

↓

Security Onion

↓

Lab Network

No public port forwarding is required.

---

# Lessons Learned

Using pfSense as the centralized router significantly improved network visibility while closely matching enterprise infrastructure.
