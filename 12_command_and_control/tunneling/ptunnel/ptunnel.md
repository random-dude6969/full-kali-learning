---
title: "ptunnel - ICMP Tunneling"
description: "Tunnel TCP connections through ICMP echo requests and replies"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "ptunnel"
kali_install: "sudo apt install ptunnel"
version: "0.61"
---

# ptunnel

## Overview

ptunnel tunnels TCP connections through ICMP echo (ping) requests and replies. It encapsulates TCP data within ICMP packets, bypassing firewalls that allow ICMP traffic but block TCP. ptunnel is useful in restrictive environments where only ICMP is permitted for outbound connectivity.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install ptunnel
```

### Verify Installation

```bash
ptunnel -h
```

## Chapter 2: Basic Usage

### Start Proxy (Relay)

```bash
sudo ptunnel
```

**Explanation**: Starts ptunnel in proxy mode, listening for ICMP tunnel connections. The proxy relays TCP connections between the client and target.

### Connect Client

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22
```

**Explanation**: `-p` specifies the proxy address, `-lp` sets the local listen port, `-da` specifies the destination address, `-dp` sets the destination port. This creates a local port 8000 that tunnels to the target's SSH port.

### Connect via Tunnel

```bash
ssh -p 8000 user@127.0.0.1
```

**Explanation**: Connects to the local port 8000, which is tunneled through ICMP to the target's SSH service. The SSH connection appears to come from the proxy's network.

## Chapter 3: Authentication

### Password Protection

```bash
# Proxy
sudo ptunnel -pass mypassword

# Client
ptunnel -p PROXY_IP -pass mypassword -lp 8000 -da TARGET:22 -dp 22
```

**Explanation**: The `-pass` flag sets a password for authentication. Both proxy and client must use the same password. This prevents unauthorized use of the tunnel.

## Chapter 4: Tunnel Configuration

### Custom Ports

```bash
ptunnel -p PROXY_IP -lp 9000 -da TARGET_IP -dp 3389
```

**Explanation**: Tunnels RDP traffic (port 3389) through ICMP to the proxy. Local port 9000 forwards to the target's RDP service.

### Multiple Tunnels

```bash
# Terminal 1: SSH tunnel
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22

# Terminal 2: HTTP tunnel
ptunnel -p PROXY_IP -lp 8080 -da TARGET_IP -dp 80

# Terminal 3: RDP tunnel
ptunnel -p PROXY_IP -lp 3389 -da TARGET_IP -dp 3389
```

**Explanation**: Multiple ptunnel instances can run simultaneously, each tunneling a different service. Each instance uses its own ICMP connection.

## Chapter 5: Performance

### Bandwidth Limitations

ICMP tunneling has significant bandwidth limitations:
- ICMP packet size limits (typically 64KB max)
- Network latency for each ping/pong exchange
- ICMP rate limiting on many networks
- Reliability depends on ICMP being permitted

**Explanation**: ICMP tunneling is suitable for lightweight C2 channels but not for bulk data transfers. Expect throughput in the range of 10-100 kbps depending on network conditions.

### MTU Considerations

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -mtu 512
```

**Explanation**: The `-mtu` flag sets the Maximum Transmission Unit for the tunnel. Lower MTU values may improve reliability on networks with strict ICMP size limits.

## Chapter 6: Security

### ICMP Traffic Analysis

ICMP tunneling can be detected by:
- High volume of ICMP traffic
- Unusual ICMP payload sizes
- ICMP traffic to unusual destinations
- Timing analysis of ping/pong patterns

**Explanation**: ICMP traffic is commonly monitored in enterprise environments. Using standard ICMP patterns and limiting tunnel activity can reduce detection risk.

### ICMP Filtering

Many networks restrict ICMP:
- Some block ICMP echo entirely
- Others limit ICMP rate
- Some inspect ICMP payloads
- Firewall rules may filter by ICMP type

**Explanation**: Before using ptunnel, verify that ICMP echo requests are permitted through the network. Test with standard `ping` to confirm connectivity.

## Chapter 7: Advanced Features

### Verbose Mode

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -v
```

**Explanation**: The `-v` flag enables verbose output, showing ICMP packet details and tunnel establishment process. Useful for debugging connection issues.

### Daemon Mode

```bash
sudo ptunnel -daemon
```

**Explanation**: Runs ptunnel as a daemon in the background. Useful for long-running tunnel services.

### Custom ICMP Identifier

```bash
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -id 12345
```

**Explanation**: The `-id` flag sets a custom ICMP identifier field. This can help distinguish tunnel traffic from legitimate ICMP traffic.

## Chapter 8: Practical Scenarios

### SSH Through ICMP

```bash
# Proxy
sudo ptunnel

# Client
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22

# Connect
ssh -p 8000 user@127.0.0.1
```

**Explanation**: The most common use case is tunneling SSH through ICMP. This provides remote access when TCP connections are blocked but ICMP is allowed.

### Pivoting

```bash
# SSH through ICMP
ptunnel -p PROXY_IP -lp 8000 -da PIVOT:22 -dp 22

# SOCKS proxy through SSH
ssh -D 1080 -p 8000 user@127.0.0.1

# Scan internal network
proxychains4 nmap -sT 10.10.10.0/24
```

**Explanation**: ptunnel can be combined with SSH SOCKS proxy for multi-hop pivoting. Traffic flows: client -> ICMP -> proxy -> TCP -> pivot -> SOCKS -> internal network.

## Resources

- [ptunnel Homepage](https://www.kali.org/tools/ptunnel/)
- [ICMP Tunneling Techniques](https://en.wikipedia.org/wiki/ICMP_tunnel)
- [RFC 792 - ICMP](https://tools.ietf.org/html/rfc792)
