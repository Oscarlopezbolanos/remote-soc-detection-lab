# pfSense Deployment

## Overview

pfSense serves as the core networking device for my cybersecurity homelab. It functions as the router, firewall, and default gateway for every virtual machine in the environment.

The goal of using pfSense was to simulate an enterprise network where all traffic flows through a dedicated firewall before reaching monitored systems.

This allows realistic testing of firewall rules, routing, attack paths, and network segmentation while supporting future expansion into VLANs and multi-network environments.

---

# Purpose

pfSense provides:

- Router
- Stateful Firewall
- Default Gateway
- DNS Resolver
- NAT
- Traffic Control
- Future VLAN Support

---

# Virtual Machine Configuration

| Setting | Value |
|----------|-------|
| Platform | VirtualBox |
| Operating System | pfSense CE |
| vCPU | 2 |
| Memory | 4 GB |
| Network Adapter 1 | NAT |
| Network Adapter 2 | Host-Only |

---

# Network Configuration

LAN Interface

IP Address

192.168.120.254/24

Default Gateway

Internet (NAT)

DNS Resolver

Enabled

Lab Network

192.168.120.0/24

---

# Firewall Design

The firewall was configured to allow communication between lab systems while maintaining centralized routing.

Example traffic:

- Kali → Windows
- Kali → Ubuntu
- Windows → Wazuh
- Windows → Security Onion
- Ubuntu → Wazuh
- Ubuntu → Security Onion

Future versions of this lab will implement VLAN segmentation and tighter east-west traffic controls.

---

# Why pfSense?

Using pfSense instead of VirtualBox NAT provides experience with enterprise networking concepts including:

- Routing
- Firewall Administration
- NAT
- Network Segmentation
- DNS Management
- Secure Remote Access

---

# Challenges Encountered

During development the following issues were encountered:

- VirtualBox Host-Only adapter conflicts
- DHCP conflicts
- Incorrect gateway assignments
- Loss of Internet connectivity after interface changes
- Routing issues between lab systems
- Static IP planning

Each issue was resolved through systematic troubleshooting and documentation.

---

# Lessons Learned

Deploying pfSense transformed the lab from a collection of virtual machines into a realistic enterprise network.

Rather than relying on VirtualBox networking alone, every system communicates through a centralized firewall, closely mirroring production environments.

This foundation allows realistic testing of attack paths, detection engineering, and incident response scenarios.
