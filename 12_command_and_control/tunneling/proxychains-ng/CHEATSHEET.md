---
title: "proxychains-ng CHEATSHEET"
description: "Quick reference for proxychains-ng proxy chaining commands"
---

# proxychains-ng CHEATSHEET

## Basic Usage

```bash
proxychains4 curl http://ifconfig.me            # Route through proxies
proxychains4 nmap -sT 10.10.10.0/24             # Scan through proxies
proxychains4 ssh user@target                     # SSH through proxies
proxychains4 bash                                # Interactive shell
proxychains4 -q curl http://ifconfig.me          # Quiet mode
```

## Configuration

```
# /etc/proxychains4.conf

# Chain types
dynamic_chain                                    # Skip dead proxies
# strict_chain                                   # All proxies required
# random_chain                                   # Random selection

# DNS through proxy
proxy_dns

# Proxy list
[ProxyList]
socks5 127.0.0.1 1080
socks4a 192.168.1.100 1080
http 10.0.0.1 8080 user password
```

## Chain Types

```
dynamic_chain                                    # Resilient, skip dead
strict_chain                                     # All must be alive
random_chain                                     # Random selection
```

## Proxy Types

```
socks4 HOST PORT                                 # SOCKS4
socks4a HOST PORT                                # SOCKS4a (remote DNS)
socks5 HOST PORT [USER PASS]                     # SOCKS5
http HOST PORT [USER PASS]                       # HTTP CONNECT
```

## Quick Setup

```bash
# Create SSH SOCKS proxy
ssh -D 1080 user@remote_host

# Configure proxychains
echo "dynamic_chain
proxy_dns
[ProxyList]
socks5 127.0.0.1 1080" > ~/.proxychains.conf

# Use
proxychains4 -f ~/.proxychains.conf curl http://ifconfig.me
```

## Multiple Proxies

```
strict_chain
[ProxyList]
socks5 127.0.0.1 1080
socks5 192.168.1.100 1080
http 10.0.0.1 8080 user pass
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-f FILE` | Custom config file |
| `-q` | Quiet mode |
| `-v` | Verbose mode |

## Practical Examples

```bash
# Web browsing through proxy
proxychains4 firefox

# Nmap through proxy (TCP connect scan only)
proxychains4 nmap -sT -p 80,443 target

# SSH through proxy
proxychains4 ssh user@target

# Wget through proxy
proxychains4 wget http://target/file.txt

# Metasploit through proxy
proxychains4 msfconsole
```

## Troubleshooting

```bash
# Test proxy connection
proxychains4 telnet proxy_host proxy_port

# Check config syntax
proxychains4 -f config.conf curl http://ifconfig.me

# Enable debug
proxychains4 -v curl http://ifconfig.me
```
