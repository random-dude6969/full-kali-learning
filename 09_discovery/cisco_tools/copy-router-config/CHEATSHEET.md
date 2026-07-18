# copy-router-config CHEATSHEET

## Quick Reference

### TFTP Copy
```bash
copy-router-config -t 192.168.1.1 -m tftp -s 192.168.1.100
```

### SCP Copy
```bash
copy-router-config -t 192.168.1.1 -m scp -u admin -p password
```

### SNMP Copy
```bash
copy-router-config -t 192.168.1.1 -m snmp -c public
```

### Options
```bash
-t host      # Target host
-m method    # tftp, scp, snmp
-s server    # TFTP server
-u user      # Username
-p pass      # Password
-c config    # running or startup
-o dir       # Output directory
-v           # Verbose
```

### Batch Backup
```bash
while read ip; do
  copy-router-config -t $ip -m tftp -s 192.168.1.100
done < routers.txt
```

---

## Target: 192.168.1.x

```bash
copy-router-config -t 192.168.1.1 -m tftp -s 192.168.1.100
```
