# smtp-user-enum CHEATSHEET

## Quick Reference

### VRFY Enumeration
```bash
smtp-user-enum -V VRFY -u admin -t 192.168.1.1
```

### EXPN Enumeration
```bash
smtp-user-enum -V EXPN -u admin -t 192.168.1.1
```

### RCPT TO Enumeration
```bash
smtp-user-enum -V RCPT -u admin -t 192.168.1.1
```

### Wordlist Attack
```bash
smtp-user-enum -V RCPT -U users.txt -t 192.168.1.1
smtp-user-enum -V RCPT -U /usr/share/wordlists/unix_users.txt -t 192.168.1.1
```

### Options
```bash
-V method     # VRFY, EXPN, or RCPT
-u user       # Single user
-U file       # User wordlist
-t host       # Target host
-p port       # Target port (default 25)
-S            # Use SSL/TLS
-M AUTH       # Authentication method
-f email      # From address
-o file       # Output file
-v            # Verbose
```

---

## Target: 192.168.1.x

```bash
smtp-user-enum -V RCPT -U /usr/share/wordlists/unix_users.txt -t 192.168.1.1
```
