# Network Segmentation Migration & pfSense Interface Recovery

**Date:** August 31, 2026  
**Environment:** VirtualBox / pfSense / Kali / Windows / Ubuntu / Wazuh / Security Onion  
**Objective:** Migrate the SOC lab from a flat network to dedicated Security, Attacker, and Victim network segments.

---

# Overview

The SOC lab originally operated primarily on a flat `192.168.120.0/24` network.

To create a more realistic attack and detection environment, the architecture was redesigned into three isolated network segments:

| Security Zone | Network | pfSense Gateway |
|---|---|---|
| Security / Management | `192.168.120.0/24` | `192.168.120.254` |
| Attacker | `192.168.130.0/24` | `192.168.130.254` |
| Victim | `192.168.140.0/24` | `192.168.140.254` |

This forces traffic between the attacker and victim environments to traverse pfSense.

During the migration, pfSense interface assignments were lost and had to be reconstructed by identifying VirtualBox adapters using their MAC addresses.

The incident provided practical experience troubleshooting:

- VirtualBox networking
- Layer 3 addressing
- pfSense interface assignment
- Static routing
- Firewall policy
- Default gateways
- Linux Netplan
- Windows static addressing
- Network segmentation

---

# Original Architecture

The lab originally used:

```text
192.168.120.0/24

Kali
Windows
Nextcloud Clone
Wazuh
Security Onion
       │
       └── pfSense 192.168.120.254
```

Although functional, this placed attacker, victim, and monitoring infrastructure within the same network.

---

# New Architecture

```text
                         pfSense
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼

 Security/Management      Attacker           Victim
 192.168.120.0/24     192.168.130.0/24   192.168.140.0/24

 Wazuh                Kali                Windows
 Security Onion                           Nextcloud Clone
                                         Metasploitable 3
```

---

# VirtualBox Networks

Dedicated VirtualBox Host-Only adapters were assigned to each security zone.

| VirtualBox Network | Subnet | Purpose |
|---|---|---|
| Host-Only Adapter #4 | `192.168.120.0/24` | Security / Management |
| Host-Only Adapter #6 | `192.168.130.0/24` | Attacker |
| Host-Only Adapter #7 | `192.168.140.0/24` | Victim |

VirtualBox DHCP is disabled because static addressing is used.

---

# pfSense Interface Mapping

The final pfSense configuration is:

| Interface | NIC | Network | Address |
|---|---|---|---|
| WAN | `le0` | VirtualBox NAT | DHCP (`10.0.2.15/24` during testing) |
| LAN | `em0` | Attacker | `192.168.130.254/24` |
| OPT1 | `em1` | Security / Management | `192.168.120.254/24` |
| OPT2 | `le1` | Victim | `192.168.140.254/24` |

---

# Problem 1 — pfSense Interface Assignment

During configuration changes, the pfSense interface assignments were reset.

pfSense displayed available interfaces similar to:

```text
le0
em0
em1
le1
```

Simply identifying interfaces by names such as `em0` and `em1` was not sufficient because the important question was:

> Which VirtualBox adapter is connected to each pfSense interface?

---

# Troubleshooting with MAC Addresses

The pfSense console displayed the MAC address associated with each interface.

Example:

```text
le0  08:00:27:82:F2:21
em0  08:00:27:96:84:95
em1  08:00:27:66:10:E1
le1  08:00:27:29:10:B7
```

These were compared against the MAC addresses configured under:

```text
VirtualBox
→ pfSense
→ Settings
→ Network
→ Adapter
```

This allowed each VirtualBox network adapter to be mapped to the correct pfSense interface.

---

# Restoring pfSense Interfaces

From the pfSense console:

```text
1) Assign Interfaces
```

The interfaces were restored to:

```text
WAN  → le0
LAN  → em0
OPT1 → em1
OPT2 → le1
```

The IP configuration was then restored using:

```text
2) Set interface(s) IP address
```

Final addressing:

```text
WAN
DHCP

LAN
192.168.130.254
/24

OPT1
192.168.120.254
/24

OPT2
192.168.140.254
/24
```

---

# Problem 2 — Kali Could Not Reach pfSense

Kali was configured as:

```text
IP:      192.168.130.105
Gateway: 192.168.130.254
```

Network configuration was verified using:

```bash
ip addr
```

and:

```bash
ip route
```

Expected routing:

```text
default via 192.168.130.254 dev eth0
192.168.130.0/24 dev eth0
```

Connectivity was tested with:

```bash
ping -c 3 192.168.130.254
```

and:

```bash
ping -c 3 8.8.8.8
```

Once the correct pfSense interface was mapped to the Attacker Host-Only network, connectivity was restored.

---

# Problem 3 — Victim Network Migration

Windows and the Nextcloud clone originally used:

```text
Windows:
192.168.120.110

Nextcloud Clone:
192.168.120.103
```

Both systems were moved to:

```text
VirtualBox Host-Only Ethernet Adapter #7
```

representing:

```text
192.168.140.0/24
```

---

# Windows Migration

Windows was changed from:

```text
IP:      192.168.120.110
Gateway: 192.168.120.254
```

to:

```text
IP:      192.168.140.110
Mask:    255.255.255.0
Gateway: 192.168.140.254
```

Configuration was verified with:

```powershell
ipconfig
```

Connectivity tests:

```powershell
ping 192.168.140.254
ping 8.8.8.8
```

Initially these tests failed even though the IP configuration was correct.

This led to investigation of pfSense firewall policy.

---

# Problem 4 — OPT2 Firewall Policy

The newly created OPT2/Victim interface did not have an appropriate pass rule.

An initial TCP-only rule was insufficient because ICMP traffic such as `ping` was not permitted.

The temporary lab connectivity rule was changed to:

```text
Action:          Pass
Interface:       OPT2
Address Family:  IPv4
Protocol:        Any

Source:          OPT2 net
Destination:     Any
```

The rule was saved and:

```text
Apply Changes
```

was selected.

After applying the rule:

```powershell
ping 192.168.140.254
```

succeeded.

Internet connectivity was then validated with:

```powershell
ping 8.8.8.8
```

---

# Ubuntu / Nextcloud Clone Migration

The Ubuntu Nextcloud clone was moved from:

```text
192.168.120.103
```

to:

```text
192.168.140.103
```

Its VirtualBox interface was first moved to:

```text
Host-Only Adapter #7
```

The current configuration was inspected using:

```bash
ip addr
```

and:

```bash
ip route
```

Netplan configuration was inspected using:

```bash
sudo cat /etc/netplan/50-cloud-init.yaml
```

The configuration was edited with:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

The static address was changed from:

```yaml
addresses:
  - 192.168.120.103/24
```

to:

```yaml
addresses:
  - 192.168.140.103/24
```

The default route was changed from:

```yaml
routes:
  - to: default
    via: 192.168.120.254
```

to:

```yaml
routes:
  - to: default
    via: 192.168.140.254
```

The new configuration was tested using:

```bash
sudo netplan try
```

After successful validation, network configuration was verified:

```bash
ip addr
ip route
```

Connectivity tests:

```bash
ping -c 3 192.168.140.254
ping -c 3 8.8.8.8
```

Both were successful.

---

# Final Device Addressing

| System | Network | IP |
|---|---|---|
| pfSense Security Gateway | Security | `192.168.120.254` |
| Wazuh | Security | `192.168.120.102` |
| Security Onion | Security | `192.168.120.150` |
| pfSense Attacker Gateway | Attacker | `192.168.130.254` |
| Kali | Attacker | `192.168.130.105` |
| pfSense Victim Gateway | Victim | `192.168.140.254` |
| Windows Victim | Victim | `192.168.140.110` |
| Nextcloud Clone | Victim | `192.168.140.103` |
| Metasploitable 3 | Victim | Planned |

---

# Traffic Flow

The segmentation now forces attack traffic through pfSense:

```text
Kali
192.168.130.105
        │
        ▼
pfSense
192.168.130.254
        │
        │ Firewall / Routing
        ▼
pfSense
192.168.140.254
        │
        ▼
Victim Systems
192.168.140.0/24
```

This architecture provides a more realistic environment for:

- Firewall analysis
- Attack detection
- Endpoint telemetry
- Network telemetry
- Incident investigation
- Containment testing

---

# Validation Commands

## Kali

```bash
ip addr
ip route

ping -c 3 192.168.130.254
ping -c 3 8.8.8.8
```

## Ubuntu Victim

```bash
ip addr
ip route

ping -c 3 192.168.140.254
ping -c 3 8.8.8.8
```

## Windows Victim

```powershell
ipconfig

ping 192.168.140.254
ping 8.8.8.8
```

---

# pfSense Configuration Backup

After restoring and validating the environment, a known-good pfSense configuration backup was created.

Location:

```text
Diagnostics
→ Backup & Restore
→ Backup Configuration
→ All
```

The pfSense XML configuration backup should NOT be committed to a public repository because it may contain sensitive configuration information.

---

# Lessons Learned

## 1. Interface names alone are not enough

`em0`, `em1`, `le0`, and `le1` do not describe which VirtualBox network they are connected to.

MAC addresses provide a reliable way to correlate VirtualBox NICs with pfSense interfaces.

## 2. VirtualBox and guest operating system configuration must agree

Changing a VM from:

```text
192.168.120.x
```

to:

```text
192.168.140.x
```

is not enough.

The VirtualBox NIC must also be connected to the correct Host-Only network.

## 3. Routing and firewall policy are different

A correct route does not automatically mean pfSense will permit the traffic.

Firewall rules must explicitly permit traffic entering an interface.

## 4. Protocol selection matters

A TCP-only firewall rule does not permit ICMP.

This became visible when `ping` continued failing even after the network configuration was correct.

## 5. Troubleshooting should be performed layer by layer

The successful troubleshooting process became:

```text
VirtualBox Adapter
       ↓
Interface / MAC
       ↓
IP Address
       ↓
Subnet
       ↓
Default Gateway
       ↓
pfSense Routing
       ↓
Firewall Policy
       ↓
Internet / Destination
```

## 6. Always create a known-good backup

After major network changes, the pfSense configuration should be exported so the environment can be restored without manually rebuilding every interface.

---

# Result

The lab was successfully migrated from a mostly flat network into three dedicated security zones:

```text
Security / Management
192.168.120.0/24

Attacker
192.168.130.0/24

Victim
192.168.140.0/24
```

The Windows and Linux victim systems were successfully migrated to the new Victim network and regained connectivity through pfSense.

The environment is now ready for the next phase:

**controlled attack → detection → investigation → incident report**
