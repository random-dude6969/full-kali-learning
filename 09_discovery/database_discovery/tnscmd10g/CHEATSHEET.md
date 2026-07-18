# tnscmd10g CHEATSHEET

## Quick Reference

### STATUS Command
```bash
tnscmd10g status -h 192.168.1.1
```

### VERSION Command
```bash
tnscmd10g version -h 192.168.1.1
```

### SERVICES Command
```bash
tnscmd10g services -h 192.168.1.1
```

### Reload Listener
```bash
tnscmd10g reload -h 192.168.1.1
```

### Options
```bash
-h host      # Target host
-p port      # Target port (default 1521)
-t timeout   # Timeout
-v           # Verbose
-c command   # Custom command
```

### Network Scan
```bash
for ip in $(nmap -p 1521 192.168.1.0/24 -oG - | grep "1521/open" | awk '{print $2}'); do
  tnscmd10g status -h $ip
done
```

---

## Target: 192.168.1.x

```bash
tnscmd10g status -h 192.168.1.1
tnscmd10g version -h 192.168.1.1
tnscmd10g services -h 192.168.1.1
```
