---
title: "sshuttle - Transparent VPN over SSH"
description: "Transparent proxy/VPN-like tool that tunnels traffic through SSH without requiring a VPN client on the remote server"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "sshuttle"
kali_install: "sudo apt install sshuttle"
version: "1.1.2"
github: "https://github.com/sshuttle/sshuttle"
---

# sshuttle

## Overview

sshuttle creates a transparent proxy server that routes traffic through an SSH tunnel. Unlike traditional VPNs, sshuttle does not require installation on the remote server — only Python is needed. It uses iptables to capture and forward traffic through the SSH connection, providing a simple but effective VPN-like solution for pivoting through compromised hosts.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install sshuttle
```

### From pip

```bash
pip install sshuttle
```

### Verify Installation

```bash
sshuttle --version
```

## Chapter 2: Basic Usage

### Route Subnet Through SSH

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: `-r` specifies the remote SSH server, and the subnet defines which traffic to route through the tunnel. All traffic to 10.0.0.0/8 is routed through the SSH connection.

### Route All Traffic

```bash
sshuttle -r user@attacker_ip 0/0
```

**Explanation**: Routes all traffic (0/0) through the SSH tunnel. This creates a full VPN, redirecting all network traffic through the remote server.

### Auto-Detect Subnets

```bash
sshuttle -r user@attacker_ip --auto-nets
```

**Explanation**: The `--auto-nets` option automatically detects which subnets are accessible through the remote server and routes traffic accordingly.

## Chapter 3: Subnet Routing

### Multiple Subnets

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 192.168.0.0/16 172.16.0.0/12
```

**Explanation**: Multiple subnets can be specified, each separated by spaces. All traffic to these subnets is routed through the SSH tunnel.

### Exclude Subnets

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -x 10.0.1.0/24
```

**Explanation**: The `-x` flag excludes specific subnets from the tunnel. Traffic to 10.0.1.0/24 remains on the local network while other 10.x traffic goes through the tunnel.

### Subnet File

```bash
sshuttle -r user@attacker_ip -s /path/to/subnets.txt
```

**Explanation**: The `-s` flag specifies a file containing subnets to route. The file should list one subnet per line, useful for managing large numbers of subnets.

## Chapter 4: DNS Tunneling

### Forward DNS Through Tunnel

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns
```

**Explanation**: The `--dns` option captures local DNS queries and forwards them to the remote DNS server. This prevents DNS leaks that could reveal target hostnames.

### Custom DNS Server

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns --to-ns 8.8.8.8
```

**Explanation**: The `--to-ns` option specifies which DNS server to use for forwarded queries. This allows using a specific DNS server on the remote network.

## Chapter 5: Authentication

### SSH Key Authentication

```bash
sshuttle -r user@attacker_ip -e "ssh -i /path/to/key" 10.0.0.0/8
```

**Explanation**: The `-e` flag specifies a custom SSH command. This allows using SSH key authentication, custom ports, or other SSH options.

### SSH Config

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: sshuttle uses the standard SSH client, which reads `~/.ssh/config`. Configure SSH options there for key files, proxy commands, and other settings.

## Chapter 6: Connection Methods

### NAT Method

```bash
sshuttle --method nat -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The NAT method uses iptables NAT rules to redirect traffic. This is the default method on Linux and provides transparent routing.

### TProxy Method

```bash
sshuttle --method tproxy -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The TProxy method uses iptables TPROXY for transparent proxying. This method supports UDP traffic and may be more compatible with certain network configurations.

### PF Method (BSD)

```bash
sshuttle --method pf -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The PF method uses BSD's packet filter for transparent routing. This is used on macOS and BSD systems.

## Chapter 7: Advanced Features

### Daemon Mode

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -D
```

**Explanation**: The `-D` flag runs sshuttle as a daemon in the background. Useful for persistent tunnels that should survive terminal disconnection.

### Hostname Scanning

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --auto-hosts
```

**Explanation**: The `--auto-hosts` option continuously scans for remote hostnames and updates the local `/etc/hosts` file. This provides hostname resolution for internal services.

### Latency Control

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --no-latency-control
```

**Explanation**: The `--no-latency-control` option sacrifices latency for improved bandwidth. Use this for bulk data transfers where latency is not critical.

## Chapter 8: Practical Scenarios

### Internal Network Access

```bash
# SSH to pivot host
ssh -D 1080 user@pivot_host

# Or use sshuttle directly
sshuttle -r user@pivot_host 10.10.10.0/24

# Access internal services
ssh user@10.10.10.5
curl http://10.10.10.10
nmap -sT 10.10.10.0/24
```

**Explanation**: sshuttle provides transparent access to internal networks through a compromised host. Tools work normally without proxychains configuration.

### Multiple Pivots

```bash
# First pivot
sshuttle -r user@external 10.10.10.0/24

# Second pivot through first
sshuttle -r user@10.10.10.5 172.16.0.0/16
```

**Explanation**: sshuttle can chain through multiple pivot points. Each sshuttle instance adds a route to the next network segment.

### Daemon with Auto-Connect

```bash
# Start daemon
sshuttle -r user@attacker_ip 10.0.0.0/8 -D --pidfile /tmp/sshuttle.pid

# Check if running
cat /tmp/sshuttle.pid
ps aux | grep sshuttle
```

**Explanation**: Running sshuttle as a daemon with a PID file allows monitoring and management of the tunnel process.

## Resources

- [sshuttle GitHub](https://github.com/sshuttle/sshuttle)
- [sshuttle Documentation](https://sshuttle.readthedocs.io/)
- [Kali Linux sshuttle Page](https://www.kali.org/tools/sshuttle/)
