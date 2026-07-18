---
title: "proxychains-ng - Proxy Chaining"
description: "Route TCP connections through SOCKS4/5 and HTTP proxies using LD_PRELOAD hooking"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "proxychains4"
kali_install: "sudo apt install proxychains4"
version: "4.17"
github: "https://github.com/rofl0r/proxychains-ng"
---

# proxychains-ng

## Overview

proxychains-ng redirects TCP connections through SOCKS4a, SOCKS5, or HTTP CONNECT proxies. It hooks network-related libc functions via LD_PRELOAD, transparently routing all TCP traffic from any application through configured proxy chains. proxychains-ng supports dynamic, strict, and random chain types for multi-hop proxy routing.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install proxychains4
```

### From Source

```bash
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure --prefix=/usr --sysconfdir=/etc
make && sudo make install
```

### Verify Installation

```bash
proxychains4 --version
```

## Chapter 2: Basic Usage

### Run Command Through Proxy

```bash
proxychains4 curl http://ifconfig.me
```

**Explanation**: Runs curl through the configured proxy chain. The curl request is routed through the proxy servers, hiding the true source IP address.

### Interactive Shell

```bash
proxychains4 bash
```

**Explanation**: Starts a bash shell where all TCP connections are routed through the proxy chain. Any command executed in this shell that makes TCP connections will use the proxies.

### Custom Configuration

```bash
proxychains4 -f /path/to/config.conf nmap -sT target_ip
```

**Explanation**: The `-f` flag specifies a custom configuration file. This allows using different proxy configurations for different operations.

## Chapter 3: Configuration

### Configuration File

```
# /etc/proxychains4.conf

# Proxy chain type
# dynamic_chain - Skip dead proxies, use available ones
# strict_chain - All proxies must be available
# random_chain - Random proxy selection

dynamic_chain

# Proxy list
[ProxyList]
socks5 127.0.0.1 1080
http 127.0.0.1 8080
```

**Explanation**: The configuration file defines the chain type and lists available proxies. `dynamic_chain` skips dead proxies, `strict_chain` requires all proxies, and `random_chain` selects randomly.

### Dynamic Chain

```
dynamic_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
```

**Explanation**: Dynamic chain uses as many proxies as possible from the list. If one proxy is unavailable, it is skipped. This provides resilience but not guaranteed anonymity.

### Strict Chain

```
strict_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
http 10.0.0.1:8080
```

**Explanation**: Strict chain requires all proxies in the list to be available. If any proxy fails, the connection fails. This provides predictable routing through specific proxies.

### Random Chain

```
random_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
http 10.0.0.1:8080
```

**Explanation**: Random chain selects proxies randomly from the list for each connection. This provides unpredictable routing but may not always use all available proxies.

## Chapter 4: Proxy Types

### SOCKS5 Proxy

```
[ProxyList]
socks5 127.0.0.1 1080 user password
```

**Explanation**: SOCKS5 proxy with authentication. Supports both TCP and UDP (though proxychains only uses TCP). The most common proxy type for tunneling.

### SOCKS4a Proxy

```
[ProxyList]
socks4a 127.0.0.1 1080
```

**Explanation**: SOCKS4a proxy supports hostname resolution on the proxy side. This prevents DNS leaks by resolving hostnames through the proxy.

### HTTP CONNECT Proxy

```
[ProxyList]
http 127.0.0.1 8080 user password
```

**Explanation**: HTTP CONNECT proxy tunnels TCP connections through the HTTP CONNECT method. Common in corporate environments where HTTP proxies are used for internet access.

## Chapter 5: DNS Resolution

### Remote DNS Resolution

```
# In configuration file
proxy_dns
```

**Explanation**: When `proxy_dns` is enabled, DNS queries are resolved through the proxy chain instead of locally. This prevents DNS leaks that could reveal the true destination.

### Local DNS

```
# Comment out proxy_dns
# proxy_dns
```

**Explanation**: Without `proxy_dns`, DNS is resolved locally. This may leak destination information but can be faster and more compatible.

## Chapter 6: Advanced Features

### Proxy Chain with Authentication

```
[ProxyList]
socks5 10.0.0.1 1080 admin password123
http 10.0.0.2 8080 user pass456
socks4a 10.0.0.3 1080
```

**Explanation**: Different proxy types and authentication can be mixed in the same chain. proxychains-ng handles authentication for each proxy type.

### Custom Configuration Files

```bash
# Create multiple config files
cp /etc/proxychains4.conf ~/.proxychains/work.conf
cp /etc/proxychains4.conf ~/.proxychains/tunnel.conf

# Use specific config
proxychains4 -f ~/.proxychains/work.conf curl http://ifconfig.me
```

**Explanation**: Multiple configuration files allow different proxy setups for different scenarios. Switch between them using the `-f` flag.

### Quiet Mode

```bash
proxychains4 -q curl http://ifconfig.me
```

**Explanation**: The `-q` flag suppresses proxychains output, showing only the command's output. Useful for scripting and automation.

## Chapter 7: Practical Scenarios

### Pivot Through Compromised Host

```bash
# On attacker
proxychains4 ssh user@compromised_host

# Or use SOCKS proxy
ssh -D 1080 user@compromised_host
proxychains4 nmap -sT 10.10.10.0/24
```

**Explanation**: After establishing a SOCKS proxy through SSH, proxychains-ng routes all TCP traffic through it. This allows scanning and accessing internal networks.

### Multi-Hop Anonymization

```
# Configuration
strict_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
http 10.0.0.1:8080
```

**Explanation**: Traffic passes through multiple proxies in sequence, each adding a layer of anonymization. The final destination sees the IP of the last proxy.

### Access Internal Services

```bash
# SSH through proxy
proxychains4 ssh user@internal_server

# Database through proxy
proxychains4 mysql -h internal_db -u user -p
```

**Explanation**: Proxychains-ng enables access to internal services through proxy chains. This is useful for accessing services on restricted networks.

## Chapter 8: Limitations and Evasion

### TCP Only

proxychains-ng only supports TCP connections. UDP, ICMP, and other protocols are not supported.

**Explanation**: This limitation means tools that rely on UDP (like DNS lookups or ping) will not work through proxychains. Use `proxy_dns` to handle DNS through the proxy.

### LD_PRELOAD Detection

Some applications detect and block LD_PRELOAD. Applications with static linking or proprietary network stacks may not work.

**Explanation**: LD_PRELOAD hooking is a well-known technique. Security tools may detect its use and block or alert on connections made through proxychains.

### Application Compatibility

Not all applications work with proxychains-ng. Applications that use their own network implementations (like some browsers) may bypass the proxy.

**Explanation**: proxychains-ng hooks standard libc functions. Applications using custom networking code or direct syscalls may not be affected by the LD_PRELOAD hooks.

## Resources

- [proxychains-ng GitHub](https://github.com/rofl0r/proxychains-ng)
- [proxychains-ng README](https://github.com/rofl0r/proxychains-ng/blob/master/README)
- [LD_PRELOAD Documentation](https://man7.org/linux/man-pages/man3/dlsym.3.html)
