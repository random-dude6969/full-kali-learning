---
title: "udptunnel - UDP Encapsulation over TCP"
description: "Tunnel UDP packets through TCP connections for bypassing UDP-blocking firewalls"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "udptunnel"
kali_install: "sudo apt install udptunnel"
version: "1.1"
---

# udptunnel

## Overview

udptunnel encapsulates UDP packets within TCP connections, allowing UDP-based services to traverse networks that block UDP traffic. It creates a TCP tunnel that carries UDP packets, enabling services like DNS, SIP, and gaming protocols to work through TCP-only firewalls.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install udptunnel
```

### Verify Installation

```bash
udptunnel -h
```

## Chapter 2: Basic Usage

### Start Server

```bash
udptunnel -s -p 5000
```

**Explanation**: Starts the udptunnel server listening on TCP port 5000. The server receives encapsulated UDP packets and forwards them to their destinations.

### Start Client

```bash
udptunnel -c SERVER_IP:5000 -l 5353 -d TARGET_IP:53
```

**Explanation**: `-c` specifies the server address, `-l` sets the local UDP listen port, `-d` specifies the destination. UDP packets sent to localhost:5353 are tunneled through TCP to the target's DNS port.

### Test Tunnel

```bash
dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: Sends a DNS query through the tunnel. The query is encapsulated in TCP, sent to the udptunnel server, and forwarded as UDP to the target DNS server.

## Chapter 3: DNS Tunneling

### DNS over TCP

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 5353 -d DNS_SERVER:53

# Query
dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: DNS queries are typically UDP, but some networks block UDP DNS. udptunnel carries DNS queries over TCP, bypassing UDP restrictions.

### SIP over TCP

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 5060 -d SIP_SERVER:5060
```

**Explanation**: SIP signaling typically uses UDP port 5060. udptunnel encapsulates SIP packets in TCP, allowing VoIP traffic through TCP-only networks.

## Chapter 4: Configuration

### Command Line Options

```bash
udptunnel -s -p 5000 -b 1000
```

**Explanation**: `-s` starts server mode, `-p` sets the listen port, `-b` sets the bandwidth limit in bytes per second.

### Client Options

```bash
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060 -b 5000
```

**Explanation**: The `-b` flag limits bandwidth to prevent saturating the TCP connection. This is useful for maintaining tunnel stability.

## Chapter 5: Performance

### Bandwidth Limitations

TCP encapsulation of UDP introduces:
- TCP overhead (headers, acknowledgments)
- Head-of-line blocking
- Congestion control
- Retransmission delays

**Explanation**: UDP's real-time properties are degraded when tunneled through TCP. Time-sensitive applications may experience latency and jitter.

### Latency

```bash
# Measure tunnel latency
time dig @127.0.0.1 -p 5353 example.com
```

**Explanation**: TCP encapsulation adds latency due to connection management and congestion control. The additional latency depends on network conditions and tunnel configuration.

## Chapter 6: Security

### Traffic Analysis

udptunnel traffic appears as:
- TCP connections on the tunnel port
- Regular data flow patterns
- No UDP signatures visible to network monitors

**Explanation**: The TCP encapsulation hides the original UDP traffic. Network monitors see only TCP connections, not the underlying UDP protocols.

### Firewall Evasion

```bash
# Bypass UDP blocking
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060
```

**Explanation**: Networks that block UDP but permit TCP can be traversed using udptunnel. The TCP tunnel carries UDP packets through the firewall.

## Chapter 7: Advanced Features

### Multiple Tunnels

```bash
# Terminal 1: DNS tunnel
udptunnel -c SERVER:5000 -l 5353 -d DNS:53

# Terminal 2: SIP tunnel
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060

# Terminal 3: NTP tunnel
udptunnel -c SERVER:5000 -l 123 -d NTP:123
```

**Explanation**: Multiple udptunnel instances can run simultaneously, each tunneling a different UDP service through TCP.

### Bandwidth Control

```bash
udptunnel -c SERVER:5000 -l 5353 -d DNS:53 -b 10000
```

**Explanation**: Bandwidth limiting prevents the tunnel from consuming all available bandwidth. This is important for maintaining tunnel stability and not impacting other network services.

## Chapter 8: Practical Scenarios

### VoIP Through Firewall

```bash
# Server on public IP
udptunnel -s -p 5000

# Client behind firewall
udptunnel -c PUBLIC_IP:5000 -l 5060 -d SIP_SERVER:5060

# Configure SIP client to use 127.0.0.1:5060
```

**Explanation**: SIP and RTP traffic can be tunneled through TCP-only firewalls, enabling VoIP connectivity in restricted environments.

### Gaming Protocol

```bash
# Server
udptunnel -s -p 5000

# Client
udptunnel -c SERVER:5000 -l 27015 -d GAME_SERVER:27015
```

**Explanation**: Many game protocols use UDP for low-latency communication. udptunnel carries game traffic through TCP when UDP is blocked.

## Resources

- [udptunnel Homepage](https://www.kali.org/tools/udptunnel/)
- [UDP Protocol](https://tools.ietf.org/html/rfc768)
- [TCP Protocol](https://tools.ietf.org/html/rfc793)
