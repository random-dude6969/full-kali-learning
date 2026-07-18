---
title: "proxytunnel - HTTP CONNECT Tunneling"
description: "Create TCP tunnels through HTTP CONNECT proxies for accessing internal services"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "proxytunnel"
kali_install: "sudo apt install proxytunnel"
version: "1.1.2"
---

# proxytunnel

## Overview

proxytunnel creates TCP tunnels through HTTP CONNECT proxies. It encapsulates arbitrary TCP traffic within HTTP CONNECT requests, allowing access to services behind corporate proxies. proxytunnel is useful for reaching SSH, VNC, and other TCP services when direct connections are blocked by proxy infrastructure.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install proxytunnel
```

### Verify Installation

```bash
proxytunnel -h
```

## Chapter 2: Basic Usage

### Create Tunnel

```bash
proxytunnel -p PROXY_HOST:8080 -d TARGET_HOST:22 -a 127.0.0.1:2222
```

**Explanation**: `-p` specifies the proxy address, `-d` specifies the destination, `-a` specifies the local address to listen on. This creates a local port 2222 that tunnels through the proxy to the target's SSH port.

### Connect via Tunnel

```bash
ssh -p 2222 user@127.0.0.1
```

**Explanation**: Connects to the local port 2222, which is tunneled through the HTTP proxy to the target's SSH service. The SSH connection is transparent to the user.

## Chapter 3: Proxy Authentication

### Basic Authentication

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -P user:password
```

**Explanation**: `-P` provides proxy authentication credentials. The credentials are base64-encoded and sent in the Proxy-Authorization header.

### NTLM Authentication

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -N -u USER -s PASSWORD -d DOMAIN
```

**Explanation**: NTLM authentication is supported for Microsoft proxy servers. The `-N` flag enables NTLM, `-u` specifies the username, `-s` the password, and `-d` the domain.

## Chapter 4: SSH Tunneling

### SSH Through Proxy

```bash
# Create tunnel
proxytunnel -p PROXY:8080 -d INTERNAL_SSH:22 -a 127.0.0.1:2222

# Connect
ssh -p 2222 user@127.0.0.1
```

**Explanation**: Creates an SSH tunnel through the HTTP proxy. The attacker connects to localhost:2222, and the traffic is tunneled through the proxy to the internal SSH server.

### SSH ProxyCommand

```bash
ssh -o ProxyCommand='proxytunnel -p PROXY:8080 -d %h:%p' user@target
```

**Explanation**: The `-o ProxyCommand` option uses proxytunnel as the transport for SSH. This integrates directly with SSH without requiring a separate tunnel process.

## Chapter 5: Advanced Features

### SSL to Proxy

```bash
proxytunnel -p PROXY:443 -d TARGET:22 -a 127.0.0.1:2222 -S
```

**Explanation**: The `-S` flag enables SSL connection to the proxy. This is useful when the proxy requires HTTPS connections.

### Custom Headers

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -H "X-Custom: value"
```

**Explanation**: Custom HTTP headers are added to the CONNECT request. This can help with proxy authentication or bypassing content filters.

### Verbose Mode

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -v
```

**Explanation**: The `-v` flag enables verbose output, showing the HTTP CONNECT handshake and tunnel establishment process.

## Chapter 6: VNC and RDP

### VNC Through Proxy

```bash
proxytunnel -p PROXY:8080 -d VNC_HOST:5900 -a 127.0.0.1:5900
```

**Explanation**: Tunnels VNC traffic through the HTTP proxy. Connect to localhost:5900 with a VNC client to access the remote desktop.

### RDP Through Proxy

```bash
proxytunnel -p PROXY:8080 -d RDP_HOST:3389 -a 127.0.0.1:3389
```

**Explanation**: Tunnels RDP traffic through the HTTP proxy. Connect to localhost:3389 with an RDP client to access the remote Windows desktop.

## Chapter 7: Security Considerations

### Traffic Analysis

HTTP CONNECT tunneling is detectable by:
- CONNECT method usage
- Unusual connection patterns
- Long-lived CONNECT tunnels
- Data volume anomalies

**Explanation**: While HTTP CONNECT tunneling bypasses simple firewall rules, sophisticated monitoring can detect the tunnel through traffic analysis.

### Proxy Detection

Some proxies:
- Log CONNECT destinations
- Limit CONNECT tunnel duration
- Block non-standard ports
- Require specific user agents

**Explanation**: Proxy servers may have policies that restrict or monitor CONNECT tunnels. Understanding the proxy's policies is essential for successful tunneling.

## Chapter 8: Practical Scenarios

### Corporate Environment

```bash
# SSH to internal server
proxytunnel -p corp-proxy:8080 -d internal-server:22 -a 127.0.0.1:2222 -P user:pass
ssh -p 2222 user@127.0.0.1

# Web server access
proxytunnel -p corp-proxy:8080 -d webserver:80 -a 127.0.0.1:8080 -P user:pass
curl http://127.0.0.1:8080
```

**Explanation**: In corporate environments with HTTP proxies, proxytunnel provides access to internal services. The HTTP CONNECT method is typically allowed through corporate proxies.

### Pivoting

```bash
# Tunnel to internal network
proxytunnel -p PROXY:8080 -d PIVOT_HOST:22 -a 127.0.0.1:2222
ssh -D 1080 -p 2222 user@127.0.0.1

# Use SOCKS proxy
proxychains4 nmap -sT 10.10.10.0/24
```

**Explanation**: proxytunnel can be combined with SSH SOCKS proxy for multi-hop pivoting. Traffic is tunneled through the HTTP proxy, then through SSH to the pivot host, and finally through the SOCKS proxy.

## Resources

- [proxytunnel Homepage](https://proxytunnel.sourceforge.net/)
- [proxytunnel man page](https://proxytunnel.sourceforge.net/man/proxytunnel.1.html)
- [Kali Linux proxytunnel Page](https://www.kali.org/tools/proxytunnel/)
