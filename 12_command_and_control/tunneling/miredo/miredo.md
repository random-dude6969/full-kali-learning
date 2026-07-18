---
title: "miredo - Teredo IPv6 Tunneling"
description: "Tunnel IPv6 packets through IPv4 networks using Teredo relay and tunnel broker"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "miredo"
kali_install: "sudo apt install miredo"
version: "1.2.6"
---

# miredo

## Overview

miredo is a Teredo tunneling implementation for Linux. It tunnels IPv6 packets through IPv4 networks using the Teredo protocol, which encapsulates IPv6 packets within UDP datagrams. Teredo is useful for providing IPv6 connectivity to IPv4-only networks and for creating covert channels that appear as normal UDP traffic.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install miredo
```

### Verify Installation

```bash
miredo --version
```

## Chapter 2: Basic Usage

### Start as Client

```bash
sudo miredo
```

**Explanation**: Starts miredo as a Teredo client, connecting to the default Teredo relay (teredo.remlab.net). The client obtains a Teredo address and creates a `teredo` tunnel interface.

### Start as Relay

```bash
sudo miredo -r
```

**Explanation**: `-r` starts miredo in relay mode. The relay forwards Teredo packets between Teredo clients and the native IPv6 internet.

### Start as Server

```bash
sudo miredo -s
```

**Explanation**: `-s` starts miredo as a Teredo server. The server assigns Teredo addresses to clients and assists with NAT traversal.

## Chapter 3: Configuration

### Configuration File

```
# /etc/miredo.conf
InterfaceName teredo
ServerAddress 193.219.129.1
TunnelMTU 1280
TunnelIPv6 yes
```

**Explanation**: The configuration file controls the tunnel interface name, server address, MTU size, and IPv6 settings. The default configuration works for most use cases.

### Custom Server

```bash
sudo miredo -c /etc/miredo.conf
```

**Explanation**: The `-c` flag specifies a custom configuration file. Use this to configure a custom Teredo server or tunnel parameters.

### Specify Interface

```
# /etc/miredo.conf
InterfaceName mytun
```

**Explanation**: Changes the tunnel interface name from the default `teredo` to a custom name. Useful when running multiple tunnel instances.

## Chapter 4: Teredo Protocol

### How Teredo Works

Teredo encapsulates IPv6 packets within UDP datagrams to traverse IPv4 NAT devices. The protocol assigns each client a Teredo address derived from the server's IP address and the client's NAT mapping.

**Explanation**: Teredo clients communicate through a Teredo server that helps with NAT traversal. Once a tunnel is established, IPv6 packets are encapsulated in UDP and sent through the IPv4 network.

### Teredo Address Format

```
2001:0000:4136:9060:0000:0000:0000:0000
```

**Explanation**: A Teredo address consists of the Teredo prefix (2001:0000:), the server's IP address, flags, the client's mapped port, and the client's mapped IP address. This encoding allows direct peer-to-peer communication.

### NAT Traversal

```bash
# Check Teredo address
ip -6 addr show teredo

# Check Teredo status
miredo status
```

**Explanation**: The Teredo address indicates successful NAT traversal. The `miredo status` command shows the current tunnel state and connection information.

## Chapter 5: Networking

### Verify Tunnel Interface

```bash
ip -6 addr show teredo
ip -6 route show
```

**Explanation**: The `teredo` interface should have an IPv6 address. Routes should include the Teredo prefix and any learned IPv6 routes.

### Ping Through Tunnel

```bash
ping6 2001:4860:4860::8888
```

**Explanation**: Pings a public IPv6 address (Google DNS) through the Teredo tunnel. This verifies that the tunnel is working and IPv6 connectivity is available.

### DNS Resolution

```bash
# Test IPv6 DNS
dig AAAA google.com
host -t AAAA google.com
```

**Explanation**: DNS AAAA records return IPv6 addresses. Testing DNS resolution over IPv6 verifies that the tunnel provides full IPv6 connectivity.

## Chapter 6: Security Considerations

### Covert Channel

Teredo traffic appears as normal UDP traffic to network monitors. The encapsulated IPv6 packets are hidden within the UDP payload, providing a potential covert channel.

**Explanation**: While Teredo is designed for IPv6 connectivity, its UDP encapsulation can be used to tunnel arbitrary traffic through networks that permit UDP but block IPv6.

### Detection Methods

- Unusual UDP traffic patterns
- Teredo-specific packet signatures
- Non-standard Teredo server usage
- IPv6-in-IPv4 encapsulation

**Explanation**: Security monitoring tools can detect Teredo traffic through packet signatures and traffic analysis. Using standard Teredo servers reduces detection risk.

## Chapter 7: Advanced Configuration

### Custom Teredo Server

```bash
# On your server
sudo miredo -s

# On clients
# /etc/miredo.conf
ServerAddress YOUR_SERVER_IP
```

**Explanation**: Running your own Teredo server provides control over address assignment and reduces reliance on public servers. The server must have public IPv4 and IPv6 connectivity.

### Relay Configuration

```bash
# On relay server
sudo miredo -r

# In miredo.conf
ServerIPv6 2001:0000:4136:9060::
```

**Explanation**: The relay forwards Teredo packets between Teredo clients and the native IPv6 internet. The relay must have both IPv4 and IPv6 connectivity.

### Multiple Interfaces

```bash
# /etc/miredo.conf
InterfaceName teredo1
```

```bash
# /etc/miredo2.conf
InterfaceName teredo2
```

```bash
# Run both
sudo miredo -c /etc/miredo.conf
sudo miredo -c /etc/miredo2.conf
```

**Explanation**: Multiple miredo instances can run simultaneously with different interface names, providing separate tunnels to different Teredo servers.

## Chapter 8: Troubleshooting

### Common Issues

```bash
# Check if teredo interface exists
ip link show teredo

# Check Teredo status
miredo status

# View logs
journalctl -u miredo

# Restart service
sudo systemctl restart miredo
```

**Explanation**: The teredo interface must exist and be in the UP state for the tunnel to work. Logs provide detailed information about connection attempts and errors.

### Firewall Rules

```bash
# Allow Teredo traffic (UDP port 3544)
sudo iptables -A INPUT -p udp --dport 3544 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 3544 -j ACCEPT
```

**Explanation**: Teredo uses UDP port 3544 for tunnel traffic. Ensure the firewall allows this traffic for the tunnel to function.

## Resources

- [miredo Homepage](https://www.remlab.net/miredo/)
- [miredo man page](https://www.remlab.net/miredo/miredo.8.html)
- [RFC 4380 - Teredo](https://tools.ietf.org/html/rfc4380)
- [Kali Linux miredo Page](https://www.kali.org/tools/miredo/)
