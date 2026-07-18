---
title: freeradius
category: credential_access
subcategory: wifi_credential
phase: 08
mitre_attack: TA0006
tags:
  - radius
  - enterprise
  - authentication
  - 802.1x
  - eap
  - testing
difficulty: advanced
os: linux
install: sudo apt install freeradius
github: https://github.com/FreeRADIUS/freeradius-server
related:
  - hostapd-wpe
  - radtest
  - eapmd5pass
---

# freeradius

## Chapter 1: Overview

**FreeRADIUS** is a high-performance RADIUS server used for authentication, authorization, and accounting (AAA). In security testing, it's used to test WPA-Enterprise implementations and audit RADIUS configurations.

### Key Features

- **Multi-protocol support**: RADIUS, DHCP, DNS
- **EAP methods**: PEAP, TLS, TTLS, MD5
- **Backend integration**: MySQL, PostgreSQL, LDAP, Kerberos
- **Policy engine**: Complex authentication policies
- **Accounting**: Session tracking and logging

### Use Cases

- Test WPA-Enterprise implementations
- Audit RADIUS configurations
- Capture enterprise credentials
- Validate EAP configurations
- Stress test authentication systems

---

## Chapter 2: Installation & Setup

```bash
sudo apt update && sudo apt install freeradius
```

### Additional Packages

```bash
sudo apt install freeradius-utils freeradius-mysql freeradius-ldap
```

### Verify Installation

```bash
freeradius -v
```

---

## Chapter 3: Basic Configuration

### Configuration Directory

```bash
ls /etc/freeradius/3.0/
```

### Main Config Files

| File | Purpose |
|------|---------|
| `radiusd.conf` | Main configuration |
| `clients.conf` | RADIUS client definitions |
| `users` | User database |
| `eap.conf` | EAP configuration |
| `sites-enabled/` | Virtual servers |

### Test Configuration

```bash
freeradius -XC
```

---

## Chapter 4: Running the Server

### Debug Mode

```bash
freeradius -X
```

### Start Service

```bash
sudo systemctl start freeradius
```

### Check Status

```bash
sudo systemctl status freeradius
```

### Stop Service

```bash
sudo systemctl stop freeradius
```

---

## Chapter 5: Client Utilities

### radtest

```bash
radtest username password 127.0.0.1 0 testing123
```

### radclient

```bash
# Authentication request
echo "User-Name = test" | radclient 127.0.0.1 auth testing123

# Status check
radclient 127.0.0.1 status testing123
```

### radsniff

```bash
# Capture RADIUS traffic
radsniff -i eth0 -p 1812

# Write to file
radsniff -i eth0 -w radius_capture.pcap
```

### radwho

```bash
# Show online users
radwho
```

---

## Chapter 6: EAP Configuration

### Enable EAP Methods

Edit `/etc/freeradius/3.0/mods-enabled/eap`:

```
eap {
    default_eap_type = peap
    
    peap {
        default_eap_type = mschapv2
    }
    
    ttls {
        default_eap_type = mschapv2
    }
}
```

### EAP-TLS (Strongest)

```
eap {
    tls {
        certfile = /etc/freeradius/certs/server.pem
        private_key_file = /etc/freeradius/certs/server.key
    }
}
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Test RADIUS Server

```bash
# Start debug mode
freeradius -X

# In another terminal, test authentication
radtest testuser testpass 127.0.0.1 0 testing123
```

### Scenario 2: Capture Enterprise Credentials

```bash
# Start RADIUS server
freeradius -X

# Configure as man-in-the-middle
# Capture authentication attempts
```

### Scenario 3: Audit EAP Configuration

```bash
# Test EAP-MD5
radeapclient 127.0.0.1 auth testing123

# Check for vulnerabilities
freeradius -XC
```

---

## Chapter 8: Mitigation & Defense

**For network administrators:**

- Use EAP-TLS (certificate-based)
- Avoid EAP-MD5 (weak)
- Implement RADIUS logging
- Regular credential rotation
- Monitor authentication failures
- Use strong RADIUS secrets
- Network segmentation
- Audit RADIUS configurations

### Secure EAP Configuration

| Method | Security | Use Case |
|--------|----------|----------|
| EAP-TLS | Strongest | Certificate environments |
| EAP-TTLS | Strong | Password-based with tunnel |
| EAP-PEAP | Good | Common enterprise |
| EAP-MD5 | Weak | Avoid |

---

## Resources

- [FreeRADIUS Documentation](https://freeradius.org/documentation/)
- [RADIUS RFC](https://tools.ietf.org/html/rfc2865)
- [EAP RFC](https://tools.ietf.org/html/rfc3748)
- [OffSec PEN-210 Course](https://www.offsec.com/courses/pen-210/)
- [WiFi Security](https://www.wi-fi.org/discover-wi-fi/wpa3)

---

*Generated for educational purposes in authorized security testing environments only.*
