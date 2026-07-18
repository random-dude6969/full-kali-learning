# cisco-ocs CHEATSHEET

## Quick Reference

### Basic Scan
```bash
cisco-ocs 192.168.1.1
```

### Options
```bash
-t host      # Target host
-p port      # Target port
-a           # All scans
-v           # Verbose
-o file      # Output file
```

### Network Scan
```bash
for ip in 192.168.1.{1..254}; do
  cisco-ocs -t $ip -o "${ip}_results.txt"
done
```

---

## Target: 192.168.1.x

```bash
cisco-ocs 192.168.1.1 -v
```
