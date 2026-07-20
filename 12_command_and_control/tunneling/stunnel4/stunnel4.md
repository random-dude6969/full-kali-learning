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

stunnel works by accepting unencrypted connections on a local port, encrypting the traffic with TLS, and forwarding it to a remote server. On the server side, stunnel accepts encrypted connections, decrypts the traffic, and forwards it to the local service. This process is transparent to both the client and the server.

The tool supports multiple concurrent connections, certificate verification, and various TLS configurations. It can be used to secure protocols like HTTP, SSH, SMTP, and any other TCP-based service.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install stunnel4
```

### From Source

```bash
wget https://www.stunnel.org/downloads/stunnel-5.73.tar.gz
tar xzf stunnel-5.73.tar.gz
cd stunnel-5.73
./configure
make
sudo make install
```

### Verify Installation

```bash
stunnel -version
```

### Check Configuration

```bash
# Default configuration location
ls -la /etc/stunnel/

# Check if service is enabled
systemctl status stunnel4
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

### Start with systemd

```bash
sudo systemctl start stunnel4
sudo systemctl enable stunnel4
```

**Explanation**: stunnel can be managed as a systemd service. The service reads configuration from `/etc/stunnel/stunnel.conf`.

## Chapter 3: Certificate Management

### Generate Self-Signed Certificate

```bash
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
cat cert.pem key.pem > /etc/stunnel/stunnel.pem
```

**Explanation**: Creates a self-signed certificate for the stunnel server. The certificate and key are combined into a single PEM file for stunnel.

### Generate CA-Signed Certificate

```bash
# Generate CA key and certificate
openssl req -newkey rsa:2048 -nodes -keyout ca.key -x509 -days 365 -out ca.crt

# Generate server key and CSR
openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.csr

# Sign server certificate
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

# Combine for stunnel
cat server.crt server.key > /etc/stunnel/cert.pem
```

**Explanation**: CA-signed certificates provide proper authentication and are trusted by default. Use this for production deployments.

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

### IMAP over SSL

```
[imap-tls]
client = no
accept = 993
connect = 143
protocol = imap
```

**Explanation**: Terminates TLS for an IMAP server. The `protocol = imap` option enables IMAP-specific TLS negotiation.

### POP3 over SSL

```
[pop3-tls]
client = no
accept = 995
connect = 110
protocol = pop3
```

**Explanation**: Terminates TLS for a POP3 server. The `protocol = pop3` option enables POP3-specific TLS negotiation.

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

### Multiple Services

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

### Delayed Retry

```
[service]
client = yes
connect = TARGET:22
retry = yes
delay = 30
```

**Explanation**: The `delay` option sets a delay in seconds between reconnection attempts. This prevents overwhelming the target with reconnection attempts.

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

### Session Caching

```
[service]
sessionCacheSize = 1000
sessionTimeout = 300
```

**Explanation**: Session caching improves performance by reusing TLS sessions. The cache size and timeout control how long sessions are cached.

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

# Check status
sudo stunnel -check
```

**Explanation**: stunnel responds to SIGHUP for configuration reload and SIGTERM for shutdown. Use the PID file for process management.

### Chroot

```
[global]
chroot = /var/lib/stunnel4
```

**Explanation**: The chroot option restricts stunnel to a specific directory, limiting the impact of potential vulnerabilities.

### Daemon Mode

```
[global]
foreground = no
pid = /var/run/stunnel4/stunnel.pid
```

**Explanation**: stunnel runs as a daemon by default. The `foreground = yes` option keeps it in the foreground for debugging.

### Resource Limits

```
[global]
setuid = stunnel4
setgid = stunnel4
```

**Explanation**: Running stunnel as a non-root user limits the impact of potential vulnerabilities. The user must have access to the certificate and key files.

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

### Database Encryption

```
[mysql-tls]
client = yes
accept = 127.0.0.1:3307
connect = db-server:3306
```

**Explanation**: Encrypts MySQL connections. The client connects to localhost:3307, and stunnel encrypts the traffic before forwarding to the database server.

### RDP over SSL

```
[rdp-tls]
client = yes
accept = 127.0.0.1:3390
connect = rdp-server:3389
```

**Explanation**: Tunnels RDP connections through SSL. The RDP client connects to localhost:3390, and stunnel encrypts the traffic before forwarding.

### Multi-Hop Tunnel

```
# First hop
[first-hop]
client = yes
accept = 127.0.0.1:2222
connect = HOP1:443
protocol = socks

# Second hop
[second-hop]
client = yes
accept = 127.0.0.1:2223
connect = HOP2:443
protocol = socks
```

**Explanation**: Chain multiple stunnel instances for multi-hop tunneling. Each hop adds a layer of encryption and routing.

## Chapter 9: Detection and Defense

### Network Signatures

- TLS connections on non-standard ports
- Self-signed certificates
- Unusual cipher suites
- Long-lived TLS sessions

### IDS/IPS Detection

```
# Suricata rule for stunnel
alert tcp any any -> any any (msg:"SSL on Non-Standard Port"; \
  flow:to_server,established; \
  content:"|16 03|"; \
  threshold:type both,track by_src, count 5, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor stunnel logs
tail -f /var/log/stunnel.log

# Check for TLS connections
ss -tnp | grep -E "ESTAB.*:[0-9]{4,5}"

# Review certificate usage
openssl x509 -in /etc/stunnel/cert.pem -text -noout
```

### Defense Recommendations

1. Monitor for TLS connections on non-standard ports
2. Implement certificate pinning
3. Use CA-signed certificates
4. Monitor for self-signed certificates
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check stunnel is running
ps aux | grep stunnel

# Check port is listening
ss -tlnp | grep 4443

# Check firewall rules
iptables -L -n | grep 443
```

### Certificate Issues

```bash
# Verify certificate
openssl x509 -in cert.pem -text -noout

# Test SSL connection
openssl s_client -connect TARGET_IP:4443

# Check certificate expiration
openssl x509 -in cert.pem -noout -dates
```

### Configuration Errors

```bash
# Test configuration
stunnel -check /etc/stunnel/stunnel.conf

# View detailed logs
stunnel -debug 7 /etc/stunnel/stunnel.conf
```

### Performance Issues

```bash
# Enable session caching
# In stunnel.conf
sessionCacheSize = 1000
sessionTimeout = 300

# Disable compression if CPU-bound
# In stunnel.conf
compression = no
```

### Permission Issues

```bash
# Check certificate permissions
ls -la /etc/stunnel/cert.pem /etc/stunnel/key.pem

# Fix permissions
sudo chmod 600 /etc/stunnel/key.pem
sudo chmod 644 /etc/stunnel/cert.pem
```

## Resources

- [stunnel Homepage](https://www.stunnel.org/)
- [stunnel man page](https://www.stunnel.org/static/stunnel.html)
- [Kali Linux stunnel4 Page](https://www.kali.org/tools/stunnel4/)
- [stunnel Configuration Examples](https://www.stunnel.org/examples.html)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
