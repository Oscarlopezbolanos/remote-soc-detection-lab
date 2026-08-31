# Firewall Rules

## Overview

pfSense is the primary firewall protecting the lab environment.

All virtual machines communicate through pfSense, allowing centralized control over traffic flowing between systems and to the Internet.

---

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



Gateway

192.168.120.254, 
192.168.130.254, 
192.168.140.254, 

---

# Security Goals

The firewall was configured to:

- Allow Internet access for all lab systems
- Permit communication between monitored systems
- Centralize routing
- Support Wazuh agent communication
- Support Security Onion monitoring
- Minimize unnecessary exposure

---

# Allowed Services

| Source | Destination | Service | Purpose |
|---------|-------------|----------|---------|
| Kali | Windows 11 | SMB | Attack Simulation |
| Kali | Ubuntu | SSH | Attack Simulation |
| Windows | Wazuh | Agent | Endpoint Monitoring |
| Ubuntu | Wazuh | Agent | Endpoint Monitoring |
| Windows | Security Onion | HTTPS | SOC Access |
| Kali | Security Onion | HTTPS | SOC Access |
| Internet | Security Onion (Tailscale Only) | HTTPS | Remote Administration |

---

# Blocked Services

Examples include:

- Unnecessary inbound Internet traffic
- Unauthorized remote administration
- Direct access to internal services without authentication

Future versions will implement VLAN-based restrictions.

---

# Future Improvements

- VLAN segmentation
- Separate attacker network
- Management VLAN
- Server VLAN
- SOC VLAN
- IDS mirror/SPAN interface
- GeoIP filtering
- Threat Intelligence feeds

---

# Lessons Learned

Centralizing all communication through pfSense greatly simplified troubleshooting and more accurately represents enterprise network architecture.
