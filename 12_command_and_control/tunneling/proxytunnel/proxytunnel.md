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

proxytunnel works by sending an HTTP CONNECT request to the proxy server, requesting a connection to the target host and port. The proxy establishes the connection, and proxytunnel then forwards data between the local port and the remote service through this tunnel.

The tool is particularly useful in corporate environments where HTTP proxies control outbound access. It can tunnel SSH, RDP, VNC, and other protocols through the proxy, providing access to internal services that would otherwise be unreachable.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install proxytunnel
```

### From Source

```bash
git clone https://github.com/ossobv/proxytunnel.git
cd proxytunnel
make
sudo make install
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

### Verify Connection

```bash
# Check if local port is listening
ss -tlnp | grep 2222

# Test connectivity
nc -zv 127.0.0.1 2222
```

**Explanation**: Verify the tunnel is working by checking the local port and testing connectivity.

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

### Digest Authentication

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -D user:password
```

**Explanation**: Digest authentication provides stronger security than basic authentication. The `-D` flag enables digest authentication.

### Proxy Authentication Header

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -H "Proxy-Authorization: Basic dXNlcjpwYXNz"
```

**Explanation**: Custom authentication headers can be provided using the `-H` flag. This allows for custom authentication schemes.

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

### SSH Config Integration

```
# ~/.ssh/config
Host internal-server
    ProxyCommand proxytunnel -p PROXY:8080 -d %h:%p -P user:password
    User admin
    Port 22
```

**Explanation**: Configure SSH to automatically use proxytunnel for specific hosts. This simplifies connection management.

### SSH over Multiple Proxies

```bash
# Chain through multiple proxies
ssh -o ProxyCommand='proxytunnel -p PROXY1:8080 -d PROXY2:8080 -a 127.0.0.1:8888 | proxytunnel -p 127.0.0.1:8888 -d TARGET:22' user@target
```

**Explanation**: Proxytunnel can be chained through multiple proxies for multi-hop tunneling.

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

### Connection Persistence

```bash
# Keep connection alive
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -k
```

**Explanation**: Some implementations support keepalive options to maintain long-lived tunnels.

### Custom User Agent

```bash
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -U "Mozilla/5.0"
```

**Explanation**: Custom user agents can help bypass proxy filters that check user agent strings.

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

### Multi-Service Access

```bash
# SSH
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222

# HTTP
proxytunnel -p PROXY:8080 -d TARGET:80 -a 127.0.0.1:8080

# RDP
proxytunnel -p PROXY:8080 -d TARGET:3389 -a 127.0.0.1:3389
```

**Explanation**: Multiple proxytunnel instances can run simultaneously, each tunneling a different service through the proxy.

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

### Evasion Techniques

```bash
# Use standard ports
proxytunnel -p PROXY:443 -d TARGET:22 -a 127.0.0.1:2222

# Use SSL
proxytunnel -p PROXY:443 -d TARGET:22 -a 127.0.0.1:2222 -S

# Custom user agent
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -U "Mozilla/5.0"
```

**Explanation**: Reduce detection risk by using standard ports, SSL, and legitimate-looking user agents.

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

### Database Access

```bash
# MySQL through proxy
proxytunnel -p PROXY:8080 -d DB_SERVER:3306 -a 127.0.0.1:3306

# Connect to database
mysql -h 127.0.0.1 -P 3306 -u user -p
```

**Explanation**: Access databases through HTTP proxies when direct connections are blocked.

### Web Application Testing

```bash
# Access internal web application
proxytunnel -p PROXY:8080 -d WEB_APP:443 -a 127.0.0.1:8443

# Test with curl
curl -k https://127.0.0.1:8443

# Test with Burp Suite
# Configure Burp to listen on 127.0.0.1:8443
```

**Explanation**: Access internal web applications through HTTP proxies for security testing.

## Chapter 9: Detection and Defense

### Network Signatures

- HTTP CONNECT requests to unusual destinations
- Long-lived CONNECT tunnels
- High data volume on CONNECT connections
- Unusual User-Agent strings

### IDS/IPS Detection

```
# Suricata rule for proxytunnel
alert http any any -> any any (msg:"HTTP CONNECT Tunnel"; \
  method:CONNECT; \
  threshold:type both,track by_src, count 10, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor proxy logs
tail -f /var/log/squid/access.log | grep CONNECT

# Check for unusual CONNECT destinations
grep CONNECT /var/log/squid/access.log | awk '{print $7}' | sort | uniq -c

# Review proxytunnel connections
ss -tnp | grep -E "ESTAB.*:8080"
```

### Defense Recommendations

1. Monitor for HTTP CONNECT requests to unusual destinations
2. Implement CONNECT method restrictions
3. Log and audit CONNECT destinations
4. Use application-layer firewalls
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check proxy is accessible
curl -x http://PROXY:8080 http://example.com

# Check firewall rules
iptables -L -n | grep 8080

# Test proxy authentication
curl -x http://user:pass@PROXY:8080 http://example.com
```

### Proxy Authentication Issues

```bash
# Test basic auth
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -P user:pass -v

# Test NTLM auth
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -N -u USER -s PASS -d DOMAIN -v

# Check proxy logs
tail -f /var/log/squid/access.log
```

### Tunnel Not Working

```bash
# Test with verbose mode
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -vvv

# Check if CONNECT is allowed
curl -x http://PROXY:8080 -p TARGET:22

# Verify target is reachable
ssh TARGET
```

### Performance Issues

```bash
# Test throughput
dd if=/dev/zero bs=1M count=10 | ssh -p 2222 user@127.0.0.1 "cat > /dev/null"

# Check latency
time ssh -p 2222 user@127.0.0.1 "echo test"

# Monitor connection
iftop -i eth0

# Optimize TCP settings
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
```

**Explanation**: Adjust system TCP buffer sizes for better throughput. Monitor connection health using network tools.

## Chapter 11: Detection and Defense

### Network Signatures

- HTTP CONNECT requests to unusual destinations
- Long-lived CONNECT tunnels
- High data volume on CONNECT connections
- Unusual User-Agent strings

### IDS/IPS Detection

```
# Suricata rule for proxytunnel
alert http any any -> any any (msg:"HTTP CONNECT Tunnel"; \
  method:CONNECT; \
  threshold:type both,track by_src, count 10, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor proxy logs
tail -f /var/log/squid/access.log | grep CONNECT

# Check for unusual CONNECT destinations
grep CONNECT /var/log/squid/access.log | awk '{print $7}' | sort | uniq -c

# Review proxytunnel connections
ss -tnp | grep -E "ESTAB.*:8080"
```

### Defense Recommendations

1. Monitor for HTTP CONNECT requests to unusual destinations
2. Implement CONNECT method restrictions
3. Log and audit CONNECT destinations
4. Use application-layer firewalls
5. Implement network segmentation

## Chapter 12: Command Reference

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-p` | Proxy address | required |
| `-d` | Destination address | required |
| `-a` | Local listen address | 127.0.0.1:0 |
| `-P` | Proxy auth (user:pass) | none |
| `-N` | Enable NTLM auth | false |
| `-u` | NTLM username | none |
| `-s` | NTLM password | none |
| `-D` | NTLM domain | none |
| `-S` | SSL to proxy | false |
| `-H` | Custom HTTP header | none |
| `-U` | Custom User-Agent | none |
| `-v` | Verbose output | quiet |
| `-k` | Keep connection alive | false |

### HTTP Methods Used

| Method | Description |
|--------|-------------|
| CONNECT | Establish tunnel |
| GET | Proxy request |
| POST | Proxy request |

### Proxy Authentication Headers

| Header | Description |
|--------|-------------|
| Proxy-Authorization | Basic/Digest auth |
| NTLM-Auth | NTLM authentication |
| Custom headers | User-defined |

## Resources

- [proxytunnel Homepage](https://proxytunnel.sourceforge.net/)
- [proxytunnel man page](https://proxytunnel.sourceforge.net/man/proxytunnel.1.html)
- [Kali Linux proxytunnel Page](https://www.kali.org/tools/proxytunnel/)
- [HTTP CONNECT Method RFC 7231](https://tools.ietf.org/html/rfc7231#section-4.3.6)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [HTTP Proxy RFC 7230](https://tools.ietf.org/html/rfc7230)
