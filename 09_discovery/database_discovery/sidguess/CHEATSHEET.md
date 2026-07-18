# sidguess CHEATSHEET

## Quick Reference

### Basic Guess
```bash
sidguess 192.168.1.1
```

### Specify Port
```bash
sidguess 192.168.1.1 -p 1521
```

### Custom Wordlist
```bash
sidguess 192.168.1.1 -w custom_sids.txt
```

### Options
```bash
-p port      # Target port (default 1521)
-w file      # Wordlist file
-t threads   # Thread count
-T timeout   # Timeout
-d delay     # Delay between attempts
-v           # Verbose
-o file      # Output file
```

### Common SIDs
```
ORCL
XE
PROD
TEST
DEV
DB01
ORACLE
```

### Network Scan
```bash
for ip in $(nmap -p 1521 192.168.1.0/24 -oG - | grep "1521/open" | awk '{print $2}'); do
  sidguess $ip -o "${ip}_sids.txt"
done
```

---

## Target: 192.168.1.x

```bash
sidguess 192.168.1.1 -v
```
