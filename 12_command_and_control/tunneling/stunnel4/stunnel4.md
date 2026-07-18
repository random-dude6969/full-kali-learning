---
title: "stunnel4 - Universal SSL Tunnel"
description: "SSL/TLS encryption wrapper for adding encryption to non-SSL aware network services"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "stunnel4"
kali_install: "sudo apt install stunnel4"
version: "5.73"
website: "https://www.stunnel.org/"
---

# stunnel4

## Overview

stunnel is a proxy designed to add TLS/SSL encryption to client-server applications without modifying the application code. It acts as an encryption wrapper between remote clients and local servers, providing SSL/TLS functionality to services that do not natively support it. stunnel is commonly used for encrypting legacy protocols and creating secure tunnels.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install stunnel4
```

### Verify Installation

```bash
stunnel -version
```

## Chapter 2: Basic Usage

### Client Mode

```bash
# /etc/stunnel/stunnel.conf
[ssh-tunnel]
client = yes
accept = 127.0.0.1:2222
connect = TARGET_IP:22
```

**Explanation**: Creates a local port 2222 that connects to the target's SSH port via SSL. The `client = yes` setting indicates this is the client side of the tunnel.

### Server Mode

```bash
# /etc/stunnel/stunnel.conf
[ssh-server]
client = no
accept = 443
connect = 127.0.0.1:22
cert = /etc/stunnel/cert.pem
key = /etc/stunnel/key.pem
```

**Explanation**: Listens on port 443 for SSL connections and forwards them to the local SSH daemon on port 22. The server presents its certificate to connecting clients.

### Start stunnel

```bash
sudo stunnel /etc/stunnel/stunnel.conf
```

**Explanation**: Starts stunnel with the specified configuration file. stunnel runs in the background by default.

## Chapter 3: Certificate Management

### Generate Self-Signed Certificate

```bash
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
cat cert.pem key.pem > /etc/stunnel/stunnel.pem
```

**Explanation**: Creates a self-signed certificate for the stunnel server. The certificate and key are combined into a single PEM file for stunnel.

### Use Existing Certificate

```bash
# In stunnel.conf
[service]
cert = /path/to/cert.pem
key = /path/to/key.pem
```

**Explanation**: stunnel can use certificates from Let's Encrypt, internal CAs, or any other certificate authority. Specify the certificate and key files separately.

### Certificate Verification

```bash
[service]
client = yes
verify = 2
CAfile = /etc/stunnel/ca-cert.pem
```

**Explanation**: The `verify` option controls certificate verification. Level 2 verifies the peer certificate chain against the CA file, preventing man-in-the-middle attacks.

## Chapter 4: Protocol Support

### HTTP Proxy

```
[https]
client = no
accept = 443
connect = 127.0.0.1:80
cert = /etc/stunnel/cert.pem
```

**Explanation**: Terminates TLS for an HTTP server. Clients connect via HTTPS on port 443, and stunnel forwards the decrypted traffic to the HTTP server on port 80.

### SSH over SSL

```
[ssh]
client = yes
accept = 127.0.0.1:2222
connect = TARGET:22
```

**Explanation**: Tunnels SSH through SSL. The SSH client connects to localhost:2222, and the traffic is encrypted with SSL before reaching the target.

### SMTP over SSL

```
[smtp-tls]
client = yes
accept = 127.0.0.1:2525
connect = mail.example.com:587
```

**Explanation**: Provides SSL encryption for SMTP connections. The mail client connects to localhost:2525, and stunnel encrypts the traffic before forwarding.

## Chapter 5: Advanced Features

### Protocol Negotiation

```
[imap-tls]
client = no
accept = 993
connect = 143
protocol = imap
```

**Explanation**: The `protocol` option enables application-specific TLS negotiation. stunnel performs the necessary protocol-specific handshake (IMAP, POP3, SMTP, NNTP) before establishing TLS.

### SOCKS Proxy

```
[socks-tls]
client = no
accept = 443
connect = 0
protocol = socks
```

**Explanation**: stunnel can function as a SOCKS proxy over TLS. Clients connect via SSL and use the SOCKS protocol to reach remote services.

### Connection Retry

```
[service]
client = yes
connect = TARGET:22
retry = yes
```

**Explanation**: The `retry` option causes stunnel to automatically reconnect if the connection drops. This provides resilience for long-lived tunnels.

## Chapter 6: Security

### Security Levels

```
[service]
securityLevel = 2
```

**Explanation**: Security levels control minimum cipher strength:
- Level 0: Everything permitted
- Level 1: 80-bit security minimum
- Level 2: 112-bit security, no RC4, no compression
- Level 3: 128-bit security, forward secrecy required

### Protocol Version

```
[service]
sslVersion = TLSv1.2
```

**Explanation**: Forces a specific TLS version. Use TLSv1.2 or TLSv1.3 for modern security. Avoid SSLv2, SSLv3, TLSv1.0, and TLSv1.1.

### Cipher Configuration

```
[service]
ciphers = HIGH:!aNULL:!MD5
```

**Explanation**: The `ciphers` option controls which TLS ciphers are permitted. Use strong ciphers and exclude weak ones.

## Chapter 7: Operational Considerations

### Logging

```
[service]
debug = 7
output = /var/log/stunnel.log
```

**Explanation**: Debug levels range from 0 (emergency) to 7 (debug). Higher levels provide more detailed logging. The `output` option specifies the log file.

### Process Management

```bash
# Start
sudo stunnel /etc/stunnel/stunnel.conf

# Reload configuration
sudo kill -HUP $(cat /var/run/stunnel4/stunnel.pid)

# Stop
sudo kill $(cat /var/run/stunnel4/stunnel.pid)
```

**Explanation**: stunnel responds to SIGHUP for configuration reload and SIGTERM for shutdown. Use the PID file for process management.

### Chroot

```
[global]
chroot = /var/lib/stunnel4
```

**Explanation**: The chroot option restricts stunnel to a specific directory, limiting the impact of potential vulnerabilities.

## Chapter 8: Practical Scenarios

### Encrypt Legacy Services

```
[legacy-app]
client = no
accept = 8443
connect = 127.0.0.1:8080
cert = /etc/stunnel/cert.pem
```

**Explanation**: Wraps a legacy HTTP application with TLS. Clients connect via HTTPS on port 8443, providing encryption for an application that only supports HTTP.

### VPN-like Tunnel

```
[tunnel]
client = yes
accept = 127.0.0.1:1080
connect = VPS:443
protocol = socks
```

**Explanation**: Creates a SOCKS proxy over SSL. Applications configured to use this SOCKS proxy can access remote networks through the encrypted tunnel.

### Multi-Service

```
[ssh]
client = yes
accept = 127.0.0.1:2222
connect = TARGET:22

[web]
client = yes
accept = 127.0.0.1:8080
connect = TARGET:80

[rdp]
client = yes
accept = 127.0.0.1:3389
connect = TARGET:3389
```

**Explanation**: Multiple services can be tunneled through stunnel simultaneously. Each service block defines its own accept/connect pair.

## Resources

- [stunnel Homepage](https://www.stunnel.org/)
- [stunnel man page](https://www.stunnel.org/static/stunnel.html)
- [Kali Linux stunnel4 Page](https://www.kali.org/tools/stunnel4/)
- [stunnel Configuration Examples](https://www.stunnel.org/examples.html)
