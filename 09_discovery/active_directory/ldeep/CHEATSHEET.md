# ldeep CHEATSHEET

## Quick Reference

### LDAP Connection
```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1
```

### User Enumeration
```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 users
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 user admin
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 kerberoastable
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 privileged
```

### Group Enumeration
```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 groups
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 group_members "Domain Admins"
```

### Other Enumeration
```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 computers
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 policies
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 trusts
```

### LDAP Search
```bash
ldeep ldap -u user -p 'P@ssw0rd' -d domain.local -s ldap://192.168.1.1 search "(objectClass=user)"
```

---

## Target: 192.168.1.x

```bash
ldeep ldap -u user -p 'Password' -d domain.local -s ldap://192.168.1.1 users
ldeep ldap -u user -p 'Password' -d domain.local -s ldap://192.168.1.1 kerberoastable
```
