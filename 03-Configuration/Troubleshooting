# Troubleshooting

## Overview

This document records the major technical issues encountered while building the SOC lab along with the steps used to resolve them.

---

# Security Onion Redirect Loop

Problem

Security Onion continued redirecting users to the old management IP.

Old IP

192.168.120.130

New IP

192.168.120.150

Resolution

```bash
sudo so-ip-update
```

Reboot the server.

---

# HTTPS Timeout from Windows

Problem

Ping worked but HTTPS timed out.

Investigation

- Verified nginx
- Verified Docker
- Verified HTTPS listener
- Verified firewall
- Used tcpdump
- Used curl
- Compared Kali vs Windows behavior

Resolution

The issue was caused by the Windows Host-Only adapter configuration rather than Security Onion.

After correcting the VirtualBox Host-Only networking configuration, HTTPS became accessible.

---

# Highstate Warning

Problem

"No highstate has completed since reboot."

Resolution

Wait for Docker containers to initialize completely.

Verify with:

```bash
sudo so-status
```

---

# Docker Containers

Verification

```bash
sudo so-status
```

Expected Result

All containers show:

Running

Healthy

---

# HTTPS Verification

Linux

```bash
curl -vk https://192.168.120.150
```

Windows

```powershell
curl -vk https://192.168.120.150
```

Expected

HTTP/302

---

# Packet Capture

Traffic verification

```bash
sudo tcpdump -ni any host <IP>
```

This was used extensively to verify:

- HTTPS requests
- Routing
- Host connectivity
- Tailscale traffic
- Network troubleshooting

---

# Lessons Learned

Building the lab required troubleshooting across multiple technologies simultaneously:

- VirtualBox
- Linux
- Docker
- pfSense
- Security Onion
- HTTPS
- SSL
- Routing
- Firewall Rules
- Tailscale
- Windows Networking

Resolving these issues provided practical experience that closely resembles real-world enterprise infrastructure administration.
