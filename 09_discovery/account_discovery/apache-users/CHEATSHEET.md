# apache-users CHEATSHEET

## Quick Reference

### Basic Scan
```bash
apache-users -h 192.168.1.1
```

### Options
```bash
-h host          # Target host
-p port          # Target port (default 80)
-w wordlist      # Custom wordlist
-t threads       # Thread count
-v               # Verbose
-s               # HTTPS
-d userdir       # Custom UserDir
-o output        # Output file
```

### Custom Wordlist
```bash
apache-users -h 192.168.1.1 -w /usr/share/wordlists/common_users.txt
```

### Multiple Targets
```bash
for ip in 192.168.1.{1..10}; do
  apache-users -h $ip -o "${ip}_users.txt"
done
```

---

## Target: 192.168.1.x

```bash
apache-users -h 192.168.1.1 -v
```
