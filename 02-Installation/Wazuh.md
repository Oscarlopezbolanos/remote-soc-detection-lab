# Wazuh Deployment

## Overview

Wazuh serves as the Endpoint Detection and Response (EDR) platform for this SOC lab.

Its purpose is to provide centralized log collection, endpoint monitoring, file integrity monitoring, authentication monitoring, and security alerting across Windows and Linux systems.

---

# Purpose

Wazuh provides:

- Endpoint Detection & Response (EDR)
- Security Information & Event Management (SIEM)
- File Integrity Monitoring
- Windows Event Collection
- Linux Log Collection
- Authentication Monitoring
- Custom Rule Support

---

# Virtual Machine Configuration

| Setting | Value |
|----------|-------|
| Operating System | Ubuntu Server |
| Role | SIEM / EDR |
| IP Address | 192.168.120.102 |

---

# Protected Endpoints

The following systems report security events to Wazuh:

- Windows 11
- Kali Linux
- Nextcloud Server

Additional endpoints will be added as the lab expands.

---

# Detection Capabilities

Current monitoring includes:

- Authentication Events
- Failed Logins
- Successful Logins
- File Integrity Monitoring
- Windows Security Events
- Linux Authentication Logs
- SSH Activity

Future improvements include:

- Sysmon
- Custom Detection Rules
- Sigma Rules
- YARA Integration

---

# Challenges Encountered

Several deployment issues were resolved during implementation:

- Agent communication problems
- Version compatibility
- Service troubleshooting
- Port configuration
- Agent registration
- Dashboard accessibility

Resolving these issues provided valuable experience with enterprise SIEM administration.

---

# Lessons Learned

Wazuh provides endpoint visibility that complements Security Onion's network visibility.

Together they create a complete detection environment capable of monitoring both endpoint activity and network traffic, enabling realistic blue team investigations.
