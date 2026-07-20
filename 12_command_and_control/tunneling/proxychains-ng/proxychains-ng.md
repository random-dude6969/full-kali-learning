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

The tool works by injecting a shared library into the target process using LD_PRELOAD. This library intercepts network-related system calls (connect, gethostbyname, etc.) and redirects them through the configured proxy chain. This approach is transparent to the application — no modification or recompilation is required.

proxychains-ng is particularly useful for pivoting through compromised hosts, anonymizing network traffic, and accessing internal networks through proxy servers. It supports authentication, remote DNS resolution, and multiple proxy types.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
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

### Check Library Installation

```bash
# The library should be installed at one of these locations
ls -la /usr/lib/libproxychains4.so
ls -la /usr/lib/x86_64-linux-gnu/libproxychains4.so
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

### Quiet Mode

```bash
proxychains4 -q curl http://ifconfig.me
```

**Explanation**: The `-q` flag suppresses proxychains output, showing only the command's output. Useful for scripting and automation.

### DNS Daemon Mode

```bash
proxychains4-daemon -i 127.0.0.1 -p 1053 -r 224
```

**Explanation**: Runs the DNS resolution daemon. This provides remote DNS resolution through the proxy chain. The `-r` flag specifies the remote subnet (default: 224).

## Chapter 3: Configuration

### Configuration File Location

```bash
# Default configuration file
/etc/proxychains4.conf

# User-specific configuration
~/.proxychains/proxychains.conf

# Custom location
proxychains4 -f /path/to/custom.conf COMMAND
```

### Configuration File Structure

```
# /etc/proxychains4.conf

# Proxy chain type (choose one)
# dynamic_chain - Skip dead proxies, use available ones
# strict_chain - All proxies must be available
# random_chain - Random proxy selection
# round_robin_chain - Round-robin proxy selection

dynamic_chain

# Quiet mode (suppress output)
# quiet_mode

# Proxy DNS through the chain
proxy_dns

# Timeout settings (in seconds)
# tcp_read_time_out 15
# tcp_connect_time_out 8

# Proxy list
[ProxyList]
# type  host  port  [user  pass]
socks5  127.0.0.1  1080
```

### Dynamic Chain

```
dynamic_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
http 10.0.0.1:8080
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

### Round-Robin Chain

```
round_robin_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100:1080
http 10.0.0.1:8080
```

**Explanation**: Round-robin chain distributes connections across proxies in sequence. This provides load balancing across multiple proxy servers.

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

### Mixed Proxy Chain

```
[ProxyList]
socks5 10.0.0.1 1080 admin password123
http 10.0.0.2 8080 user pass456
socks4a 10.0.0.3 1080
```

**Explanation**: Different proxy types can be mixed in the same chain. proxychains-ng handles authentication for each proxy type.

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

### DNS Daemon

```bash
# Start DNS daemon
proxychains4-daemon -i 127.0.0.1 -p 1053

# Configure applications to use the daemon
# In /etc/resolv.conf:
# nameserver 127.0.0.1
```

**Explanation**: The DNS daemon provides a local DNS server that resolves queries through the proxy chain. Configure applications to use this daemon for DNS resolution.

### DNS Leak Prevention

```bash
# Verify DNS is not leaking
proxychains4 dig @8.8.8.8 myip.opendns.com

# Check DNS resolution
proxychains4 nslookup example.com
```

**Explanation**: Verify that DNS queries are going through the proxy chain and not leaking to local DNS servers.

## Chapter 6: Advanced Features

### Multiple Configuration Files

```bash
# Create multiple config files
cp /etc/proxychains4.conf ~/.proxychains/work.conf
cp /etc/proxychains4.conf ~/.proxychains/tunnel.conf

# Use specific config
proxychains4 -f ~/.proxychains/work.conf curl http://ifconfig.me
```

**Explanation**: Multiple configuration files allow different proxy setups for different scenarios. Switch between them using the `-f` flag.

### Proxy Chain with Authentication

```
[ProxyList]
socks5 10.0.0.1 1080 admin password123
http 10.0.0.2 8080 user pass456
socks4a 10.0.0.3 1080
```

**Explanation**: Different proxy types and authentication can be mixed in the same chain. proxychains-ng handles authentication for each proxy type.

### Timeouts

```
# In configuration file
tcp_read_time_out 15
tcp_connect_time_out 8
```

**Explanation**: Adjust timeout values for slow proxy servers. The `tcp_read_time_out` controls how long to wait for data, and `tcp_connect_time_out` controls connection timeout.

### Quiet Mode

```
# In configuration file
quiet_mode
```

**Explanation**: Quiet mode suppresses proxychains output, showing only the command's output. Useful for scripting and automation.

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

# Web application through proxy
proxychains4 curl http://internal-web/
```

**Explanation**: Proxychains-ng enables access to internal services through proxy chains. This is useful for accessing services on restricted networks.

### Nmap Through Proxy

```bash
# TCP connect scan through proxy
proxychains4 nmap -sT -Pn 10.10.10.0/24

# Specific port scan
proxychains4 nmap -sT -Pn -p 22,80,443 10.10.10.0/24
```

**Explanation**: Nmap scanning through proxychains requires `-sT` (TCP connect scan) because SYN scanning requires raw sockets which don't work through proxies. The `-Pn` flag skips host discovery.

### Metasploit Through Proxy

```bash
# Configure Metasploit to use proxy
proxychains4 msfconsole

# Or configure in Metasploit
msfconsole
msf6> setg Proxies socks5:127.0.0.1:1080
```

**Explanation**: Metasploit can be run through proxychains or configured to use proxies directly. The `setg` command sets global options.

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

### DNS Resolution Issues

Some applications may not work correctly with `proxy_dns` enabled. This can cause timeouts or connection failures.

**Explanation**: Remote DNS resolution can be slower than local resolution. Some applications may have timeout issues.

### Workarounds

```bash
# Use with specific applications that support SOCKS
proxychains4 curl --socks5 127.0.0.1:1080 http://ifconfig.me

# Use with applications that support HTTP proxy
proxychains4 wget -e "http_proxy=http://127.0.0.1:8080" http://ifconfig.me
```

**Explanation**: Some applications have native proxy support. Using the application's built-in proxy settings may be more reliable than LD_PRELOAD hooking.

## Chapter 9: Detection and Defense

### Network Signatures

- Multiple proxy connections in sequence
- Unusual DNS resolution patterns
- Connections to known proxy servers

### IDS/IPS Detection

```
# Suricata rule for proxychains
alert tcp any any -> any any (msg:"Proxy Chain Detected"; \
  flow:to_server,established; \
  content:"SOCKS"; \
  threshold:type both,track by_src, count 10, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for proxy connections
ss -tnp | grep -E "ESTAB.*1080|8080"

# Check for LD_PRELOAD
cat /proc/*/environ | grep LD_PRELOAD

# Review proxy server logs
tail -f /var/log/squid/access.log
```

### Defense Recommendations

1. Monitor for connections to known proxy servers
2. Implement egress filtering
3. Use application-layer firewalls
4. Monitor for LD_PRELOAD usage
5. Implement network segmentation

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check proxy server is running
ss -tlnp | grep 1080

# Test proxy connection
curl --socks5 127.0.0.1:1080 http://ifconfig.me

# Check proxychains configuration
cat /etc/proxychains4.conf
```

### DNS Resolution Issues

```bash
# Disable proxy_dns for testing
# proxy_dns

# Use local DNS
proxychains4 nslookup example.com 8.8.8.8
```

### Application Compatibility

```bash
# Check if application uses libc
ldd /usr/bin/application | grep libc

# Try with different chain type
# Change dynamic_chain to strict_chain or random_chain
```

### Performance Issues

```bash
# Reduce timeout values
tcp_read_time_out 10
tcp_connect_time_out 5

# Use fewer proxies
[ProxyList]
socks5 127.0.0.1 1080
```

### LD_PRELOAD Errors

```bash
# Check if library is installed
ls -la /usr/lib/libproxychains4.so

# Set LD_PRELOAD manually
LD_PRELOAD=/usr/lib/libproxychains4.so curl http://ifconfig.me
```

## Resources

- [proxychains-ng GitHub](https://github.com/rofl0r/proxychains-ng)
- [proxychains-ng README](https://github.com/rofl0r/proxychains-ng/blob/master/README)
- [LD_PRELOAD Documentation](https://man7.org/linux/man-pages/man3/dlsym.3.html)
- [Kali Linux proxychains-ng Page](https://www.kali.org/tools/proxychains-ng/)
- [SOCKS Protocol RFC 1928](https://tools.ietf.org/html/rfc1928)
