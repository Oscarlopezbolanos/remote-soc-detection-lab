# SSL Configuration

## Overview

Security Onion provides secure HTTPS access to the SOC interface.

All administration is performed over encrypted TLS connections.

---

# HTTPS Access

Primary Interface

https://192.168.120.150

Remote Interface

https://<tailscale-ip>

---

# Certificate

Security Onion uses a self-signed certificate by default.

Because the certificate is not signed by a trusted Certificate Authority, browsers display a warning.

This behavior is expected inside a lab environment.

---

# Browser Warning

Users may receive:

NET::ERR_CERT_AUTHORITY_INVALID

This is normal for self-signed certificates.

After verifying the server identity, the connection can safely proceed.

---

# Troubleshooting

During deployment the management IP changed from:

192.168.120.130

to

192.168.120.150

The Security Onion web interface continued redirecting users to the old address.

The issue was resolved using:

```bash
sudo so-ip-update
```

followed by a reboot.

---

# Verification

HTTPS connectivity was verified using:

```bash
curl -vk https://192.168.120.150
```

The response returned:

HTTP/1.1 302

confirming nginx was operating correctly.

---

# Future Improvements

Future versions of the lab may implement:

- Internal Certificate Authority
- Let's Encrypt
- Automatic certificate renewal
- Enterprise PKI integration

---

# Lessons Learned

Understanding HTTPS troubleshooting, certificates, redirects, and encrypted communications is essential for administering modern security platforms.
