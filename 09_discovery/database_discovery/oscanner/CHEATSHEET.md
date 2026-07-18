# oscanner CHEATSHEET

## Quick Reference

### Basic Scan
```bash
oscanner -h 192.168.1.1
```

### Specify Port
```bash
oscanner -h 192.168.1.1 -P 1521
```

### Guess SID
```bash
oscanner -h 192.168.1.1 -S
```

### Test Default Credentials
```bash
oscanner -h 192.168.1.1 -D
```

### Options
```bash
-h host      # Target host
-P port      # Target port (default 1521)
-S           # SID guessing
-D           # Default credential test
-s service   # Service name
-F file      # Custom SID list
-U file      # User wordlist
-P file      # Password wordlist
-t timeout   # Timeout
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
```

### Default Credentials
```
sys: change_on_install
system: manager
scott: tiger
hr: hr
```

---

## Target: 192.168.1.x

```bash
oscanner -h 192.168.1.1 -S -D -v
```
