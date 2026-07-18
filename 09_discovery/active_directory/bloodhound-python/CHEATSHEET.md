# bloodhound-python CHEATSHEET

## Quick Reference

### Password Authentication
```bash
bloodhound-python -u user -p 'P@ssw0rd' -d domain.local -ns 192.168.1.1 -c All
```

### Hash Authentication
```bash
bloodhound-python -u user -H aad3b435b51404eeaad3b435b51404ee:da76f... -d domain.local -ns 192.168.1.1 -c All
```

### Kerberos Authentication
```bash
bloodhound-python -u user -k -d domain.local -ns 192.168.1.1 -c All
```

### Collection Types
```bash
-c Users           # Users
-c Groups          # Groups
-c Computers       # Computers
-c ACLs            # ACLs
-c Trusts          # Trusts
-c Sessions        # Sessions
-c All             # Everything
```

### Options
```bash
-u user         # Username
-p pass         # Password
-H hash         # NTLM hash
-d domain       # Domain
-ns nameserver  # DNS server
-dc dc          # Domain controller
--stealth       # Stealth mode
```

### Stealth Collection
```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All --stealth
```

---

## Target: 192.168.1.x

```bash
bloodhound-python -u user@domain.local -p 'Password' -d domain.local -ns 192.168.1.1 -c All
```
