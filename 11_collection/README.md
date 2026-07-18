---
mitre_tactic: collection
mitre_tactic_id: TA0009
phase: 11
title: Collection Tools
---

# Phase 11: Collection Tools

## Overview

Collection tools in MITRE ATT&CK's TA0009 tactic focus on gathering information that can be used to achieve objectives during an intrusion. These tools enable attackers to intercept, capture, and exfiltrate data from networks, systems, and communications.

**MITRE ATT&CK Tactic:** Collection (TA0009)

**Objective:** Gather target information for exfiltration or use in subsequent attack phases

---

## Tools in This Phase

| Tool | Category | Primary Function | Language |
|------|----------|------------------|----------|
| **mitm6** | IPv6 DNS Spoofing | Exploit Windows IPv6 defaults for DNS takeover | Python |
| **mitmproxy** | HTTP/HTTPS Proxy | Interactive TLS-capable intercepting proxy | Python |
| **ssldump** | SSL/TLS Analyzer | Decode and decrypt SSL/TLS traffic | C |
| **sslsniff** | SSL/TLS MITM | Create SSL/TLS man-in-the-middle attacks | C++ |
| **sslsplit** | Transparent Proxy | Transparent SSL/TLS interception via NAT | C |
| **wifipumpkin3** | Rogue AP Framework | Create rogue access points for wireless MITM | Python |

---

## Tool Categories

### IPv6-Based Attacks

**mitm6** exploits Windows' default preference for IPv6 DNS servers. By responding to DHCPv6 messages and setting itself as the DNS server, it can redirect IPv4 traffic through IPv6 attack vectors.

**Use Cases:**
- Active Directory domain compromise
- WPAD spoofing for credential capture
- Kerberos relay attacks
- Internal network reconnaissance

### HTTP/HTTPS Interception

**mitmproxy** provides a comprehensive proxy solution for intercepting and manipulating HTTP/HTTPS traffic. It supports multiple interfaces (console, web, CLI) and extensive scripting capabilities.

**Use Cases:**
- Web application security testing
- API analysis and debugging
- Certificate pinning bypass
- Mobile application traffic inspection
- Credential extraction

### SSL/TLS Protocol Analysis

**ssldump**, **sslsniff**, and **sslsplit** provide different approaches to SSL/TLS interception:

- **sslsump**: Protocol analyzer for decoding SSL/TLS traffic (requires private keys)
- **sslsniff**: MITM attack tool with dynamic certificate generation
- **sslsplit**: Transparent proxy for large-scale SSL/TLS interception

**Use Cases:**
- Network forensics
- Encrypted traffic analysis
- Credential capture from SSL sessions
- Large-scale traffic monitoring

### Wireless Network Attacks

**wifipumpkin3** creates rogue access points for wireless man-in-the-middle attacks. It includes captive portal attacks, credential harvesting, and traffic interception capabilities.

**Use Cases:**
- Evil twin attacks
- Captive portal phishing
- WiFi credential harvesting
- Wireless traffic interception

---

## Attack Flow Diagrams

### mitm6 Attack Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    mitm6 Attack Flow                             │
├─────────────────────────────────────────────────────────────────┤
│  1. Victim broadcasts DHCPv6 Information-Request                │
│  2. mitm6 responds with DHCPv6 Advertise                        │
│  3. mitm6 sends Router Advertisement (optional)                 │
│  4. Victim configures attacker as DNS server                     │
│  5. DNS queries intercepted and spoofed                         │
│  6. Traffic redirected to attacker-controlled services           │
└─────────────────────────────────────────────────────────────────┘
```

### mitmproxy Attack Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   mitmproxy Attack Flow                          │
├─────────────────────────────────────────────────────────────────┤
│  1. Client sends HTTPS request through proxy                    │
│  2. mitmproxy forwards ClientHello to server                    │
│  3. Server responds with certificate                            │
│  4. mitmproxy generates forged certificate                      │
│  5. Client connects using forged certificate                    │
│  6. mitmproxy decrypts, inspects, modifies traffic              │
│  7. Re-encrypts and forwards to server                          │
└─────────────────────────────────────────────────────────────────┘
```

### SSLsplit Attack Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   SSLsplit Attack Flow                           │
├─────────────────────────────────────────────────────────────────┤
│  1. iptables/nftables redirects traffic to sslsplit             │
│  2. sslsplit terminates incoming SSL/TLS connection              │
│  3. sslsplit generates forged certificate on-the-fly            │
│  4. New SSL/TLS connection to original destination              │
│  5. All data logged in plaintext                                 │
│  6. Traffic forwarded between client and server                 │
└─────────────────────────────────────────────────────────────────┘
```

### wifipumpkin3 Attack Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                 wifipumpkin3 Attack Flow                         │
├─────────────────────────────────────────────────────────────────┤
│  1. Wireless adapter creates rogue access point                 │
│  2. Victim connects to rogue AP                                  │
│  3. DHCP assigns IP address                                     │
│  4. DNS requests intercepted                                    │
│  5. Captive portal presented to victim                          │
│  6. Credentials harvested                                       │
│  7. All traffic intercepted via MITM                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tool Selection Guide

### When to Use mitm6

- **Target:** Active Directory environments
- **Goal:** Credential capture, domain compromise
- **Prerequisites:** Network access, IPv6 enabled on targets
- **Best For:** WPAD spoofing, NTLM relay, Kerberos relay

### When to Use mitmproxy

- **Target:** HTTP/HTTPS traffic
- **Goal:** Traffic inspection, modification, credential extraction
- **Prerequisites:** Proxy configuration on target
- **Best For:** Web app testing, API analysis, scripting

### When to Use ssldump

- **Target:** SSL/TLS traffic with available private keys
- **Goal:** Protocol analysis, traffic decryption
- **Prerequisites:** RSA private key or SSLKEYLOGFILE
- **Best For:** Forensics, debugging, protocol analysis

### When to Use sslsniff

- **Target:** SSL/TLS connections
- **Goal:** Dynamic certificate generation, MITM
- **Prerequisites:** Network positioning, CA certificate
- **Best For:** Browser-based attacks, OCSP attacks

### When to Use sslsplit

- **Target:** Large-scale SSL/TLS interception
- **Goal:** Transparent proxy, traffic logging
- **Prerequisites:** NAT engine, CA certificate
- **Best For:** Network forensics, large-scale monitoring

### When to Use wifipumpkin3

- **Target:** Wireless networks
- **Goal:** Rogue AP, credential harvesting
- **Prerequisites:** Wireless adapter in AP mode
- **Best For:** Evil twin, captive portal, phishing

---

## Integration Patterns

### mitm6 + ntlmrelayx

```bash
# Terminal 1: Start ntlmrelayx
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc.target.local

# Terminal 2: Start mitm6
mitm6 -d target.local
```

### mitm6 + krbrelayx

```bash
# Terminal 1: Start krbrelayx
krbrelayx.py -t dc.target.local

# Terminal 2: Start mitm6
mitm6 --relay dc.target.local -d target.local
```

### mitmproxy + Wireshark

```bash
# Start mitmproxy with PCAP export
mitmdump -w traffic.pcap

# Analyze in Wireshark
wireshark traffic.pcap
```

### sslsplit + Wireshark

```bash
# Start sslsplit with master key logging
sslsplit -M masterkeys.log -k ca.key -c ca.pem https 0.0.0.0 8443

# Import master keys in Wireshark
# Edit > Preferences > Protocols > TLS > Master Secret Log
```

### wifipumpkin3 + aircrack-ng

```bash
# Start rogue AP
wifipumpkin3 -i wlan0

# In separate terminal, deauthenticate clients
aireplay-ng --deauth 10 -a [AP_MAC] wlan0mon
```

---

## Detection Signatures

### Network-Based Detection

- **Rogue DHCPv6:** Monitor for unexpected DHCPv6 responses
- **SSL/TLS Certificates:** Track certificate issuances
- **DNS Anomalies:** Detect DNS spoofing patterns
- **Rogue APs:** Identify unauthorized access points

### Host-Based Detection

- **Certificate Pinning:** Verify against pinned certificates
- **OCSP Stapling:** Check for valid OCSP responses
- **HSTS Preloading:** Validate HSTS headers
- **Network Configuration:** Monitor DNS server changes

### Log Analysis

- **Authentication Failures:** Monitor NTLM/Kerberos failures
- **Certificate Errors:** Track SSL/TLS validation errors
- **DNS Query Logs:** Analyze DNS request patterns
- **Connection Logs:** Review proxy and connection attempts

---

## Legal and Ethical Considerations

### Authorization Requirements

- **Written Permission:** Obtain explicit authorization before testing
- **Scope Definition:** Clearly define testing boundaries
- **Time Limits:** Specify testing duration
- **Data Handling:** Establish data retention policies

### Responsible Disclosure

- **Report Findings:** Document and report vulnerabilities
- **Protect Data:** Secure captured credentials and data
- **Comply with Laws:** Follow local regulations (CFAA, GDPR, etc.)

### Testing Best Practices

- **Isolated Networks:** Use lab environments when possible
- **Monitor Impact:** Watch for unintended consequences
- **Clean Up:** Remove all artifacts after testing
- **Document Everything:** Maintain detailed logs

---

## Additional Resources

### Documentation

- [MITRE ATT&CK: Collection](https://attack.mitre.org/tactics/TA0009/)
- [Kali Linux Tools](https://www.kali.org/tools/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

### Related Phases

- **Phase 10: Credential Access** - Steal account credentials
- **Phase 12: Exfiltration** - Extract collected data
- **Phase 13: Command and Control** - Maintain persistence

### Practice Resources

- **HackTheBox:** Practice MITM attacks in safe environment
- **VulnHub:** Download vulnerable VMs for testing
- **OWASP WebGoat:** Practice web application attacks

---

## Quick Reference

### Installation Commands

```bash
# mitm6
sudo apt install mitm6

# mitmproxy
sudo apt install mitmproxy

# ssldump
sudo apt install ssldump

# sslsniff
sudo apt install sslsniff

# sslsplit
sudo apt install sslsplit

# wifipumpkin3
sudo apt install wifipumpkin3
```

### Essential Commands

```bash
# mitm6
mitm6 -d target.local

# mitmproxy
mitmproxy -p 8080

# ssldump
ssldump -i eth0 -k server.key

# sslsniff
sslsniff -a -c ca.crt -s 443 -w output.log

# sslsplit
sslsplit -k ca.key -c ca.pem https 0.0.0.0 8443

# wifipumpkin3
wifipumpkin3 -i wlan0 -iNet eth0
```
