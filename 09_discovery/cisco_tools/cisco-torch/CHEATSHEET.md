# cisco-torch CHEATSHEET

## Quick Reference

### Fingerprint
```bash
cisco-torch -F 192.168.1.1
```

### Vulnerability Scan
```bash
cisco-torch -V 192.168.1.1
```

### Download Config
```bash
cisco-torch -C 192.168.1.1
```

### Test Credentials
```bash
cisco-torch -T 192.168.1.1
```

### Options
```bash
-F           # Fingerprint
-V           # Vulnerability scan
-C           # Download config
-T           # Test credentials
-p port      # Target port
-v           # Verbose
-o file      # Output file
```

### Batch Scan
```bash
cisco-torch -F -i targets.txt
cisco-torch -V 192.168.1.0/24 -o vuln_report.txt
```

---

## Target: 192.168.1.x

```bash
cisco-torch -F 192.168.1.1
cisco-torch -V 192.168.1.1
```
