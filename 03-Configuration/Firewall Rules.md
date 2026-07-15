# Firewall Rules

## Overview

pfSense is the primary firewall protecting the lab environment.

All virtual machines communicate through pfSense, allowing centralized control over traffic flowing between systems and to the Internet.

---

# Network

Lab Network

192.168.120.0/24

Gateway

192.168.120.254

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
