# Routing

## Overview

All systems in the lab communicate through pfSense.

Rather than relying on VirtualBox networking alone, every virtual machine uses pfSense as its default gateway.

This creates a realistic enterprise routing environment.

---

# Network Topology

Network

192.168.120.0/24

Gateway

192.168.120.254

---

# Device Routing

| Device | IP Address | Gateway |
|----------|------------|----------|
| Kali | 192.168.120.105 | 192.168.120.254 |
| Windows 11 | 192.168.120.110 | 192.168.120.254 |
| Nextcloud Ubuntu | 192.168.120.103 | 192.168.120.254 |
| Wazuh | 192.168.120.102 | 192.168.120.254 |
| Security Onion | 192.168.120.150 | 192.168.120.254 |

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
