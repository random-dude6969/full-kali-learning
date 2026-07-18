---
tool_name: "freerdp"
display_name: "FreeRDP"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "pass_the_hash"
install_command: "sudo apt install freerdp2-x11"
version: "3.30.0"
github_repo: "https://github.com/FreeRDP/FreeRDP"
kali_tools_page: "https://www.kali.org/tools/freerdp2-x11/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: true
tags: ["rdp", "remote-desktop", "pass-the-hash", "lateral-movement", "windows"]
related_tools: ["rdesktop", "xfreerdp", "remmina"]
---

# FreeRDP

## Chapter 1: Introduction & Overview

### What is FreeRDP?

FreeRDP is a free, open-source implementation of the Remote Desktop Protocol (RDP). It provides a full-featured RDP client for connecting to Windows machines graphically. In penetration testing, FreeRDP is the primary tool for pass-the-hash RDP connections, allowing authentication using NTLM hashes without knowing the cleartext password.

### History & Background

- **Created:** 2009 by Marc-André Moreau
- **Maintained:** FreeRDP community (13.5k GitHub stars, 15.4k forks)
- **License:** Apache-2.0
- **Purpose:** Open-source RDP client with full protocol support

FreeRDP replaced `rdesktop` as the preferred RDP client in Kali Linux. It supports RDP 8.0+, NLA (Network Level Authentication), CredSSP, and pass-the-hash authentication.

### When to Use This Tool

- **Pass-the-hash RDP:** Authenticate to Windows using NTLM hashes
- **Remote access:** Interactive GUI access to Windows machines
- **Lateral movement:** Move through networks using RDP
- **Credential testing:** Test RDP access with compromised credentials

### Key Concepts

- **RDP:** Remote Desktop Protocol for graphical remote access
- **NLA:** Network Level Authentication (pre-authentication)
- **CredSSP:** Credential Security Support Provider
- **Pass-the-Hash:** Authenticate using NTLM hash instead of password
- **xfreerdp:** Command-line RDP client (FreeRDP's main binary)

### FreeRDP vs Other RDP Clients

| Feature | FreeRDP | rdesktop | Remmina |
|---------|---------|----------|---------|
| Pass-the-Hash | Yes | No | Limited |
| RDP 8.0+ | Yes | No | Yes |
| NLA Support | Full | Limited | Full |
| CredSSP | Yes | No | Yes |
| Channel Support | Full | Basic | Full |
| Clipboard | Yes | Basic | Yes |
| Drive Redirection | Yes | Yes | Yes |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux, Ubuntu, Debian, or any Linux with X11
- **RAM:** 256 MB minimum
- **Disk Space:** ~50 MB
- **Display:** X11 server (for GUI mode)

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update && sudo apt install freerdp2-x11
```

**Explanation:** Installs FreeRDP with X11 support. The `freerdp2-x11` package includes `xfreerdp`.

#### Method 2: From Source

```bash
# Install dependencies
sudo apt install cmake gcc libssl-dev libx11-dev libxext-dev \
  libxinerama-dev libxcursor-dev libxi-dev libxdamage-dev \
  libxv-dev libxxf86vm-dev libxtst-dev

# Clone repository
git clone https://github.com/FreeRDP/FreeRDP.git
cd FreeRDP

# Build
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make -j$(nproc)

# Install
sudo make install
```

**Explanation:** Building from source provides the latest features and patches.

### Verify Installation

```bash
xfreerdp --version
```

**Explanation:** Confirms FreeRDP is installed and shows the version.

---

## Chapter 3: Basic Usage

### Basic Connection

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123
```

**Explanation:** Connects to a Windows machine using username and password. Opens a graphical RDP session.

### Pass-the-Hash Connection

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /pth:'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'
```

**Explanation:** The `/pth` flag enables pass-the-hash authentication using the LM:NT hash format. This is the key pentesting feature.

### Connection Options

```bash
# Full options example
xfreerdp /v:192.168.1.100 \
  /u:Administrator \
  /pth:'LM:NT' \
  /w:1920 /h:1080 \
  /f \
  /cert-ignore \
  /auto-reconnect \
  /dynamic-resolution
```

**Explanation:** Common options include resolution settings, fullscreen mode, certificate handling, and auto-reconnect.

### Essential Flags

| Flag | Description | Example |
|------|-------------|---------|
| `/v:host` | Target host | `/v:192.168.1.100` |
| `/u:user` | Username | `/u:Administrator` |
| `/p:pass` | Password | `/p:Password123` |
| `/pth:hash` | Pass-the-hash | `/pth:'LM:NT'` |
| `/d:domain` | Domain name | `/d:CONTOSO` |
| `/w:width` | Screen width | `/w:1920` |
| `/h:height` | Screen height | `/h:1080` |
| `/f` | Fullscreen | `/f` |
| `/cert-ignore` | Ignore certificate | `/cert-ignore` |
| `/sec:tls` | Security protocol | `/sec:tls` |
| `/sec:nla` | NLA authentication | `/sec:nla` |
| `/sec:rdp` | RDP security | `/sec:rdp` |
| `/sound` | Enable sound | `/sound` |
| `/mic` | Enable microphone | `/mic` |
| `/drive:name,path` | Drive redirection | `/drive:share,/tmp` |
| `/ clipboard` | Clipboard sharing | `/clipboard` |
| `/ printers` | Printer redirection | `/printers` |
| `/usb` | USB redirection | `/usb` |
| `/app:program` | RemoteApp | `/app:notepad.exe` |
| `/bpp:color` | Color depth | `/bpp:32` |
| `/fonts` | Smooth fonts | `/fonts` |
| `/smartcard` | Smartcard support | `/smartcard` |
| `/log-level:level` | Logging verbosity | `/log-level:INFO` |

---

## Chapter 4: Intermediate Usage

### Domain Authentication

```bash
# With domain
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 /d:CONTOSO

# With domain pass-the-hash
xfreerdp /v:192.168.1.100 /u:Administrator /d:CONTOSO \
  /pth:'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0'
```

**Explanation:** Domain authentication requires the `/d` flag. Pass-the-hash works with domain accounts.

### File Transfer via Drive Redirection

```bash
# Mount local directory as drive
xfreerdp /v:192.168.1.100 /u:Administrator /pth:'LM:NT' \
  /drive:tools,/path/to/local/tools
```

**Explanation:** Drive redirection makes a local directory accessible as a network drive on the remote machine.

### RemoteApp Execution

```bash
# Run specific application
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /app:"C:\Windows\System32\cmd.exe"
```

**Explanation:** RemoteApp launches a specific application instead of a full desktop session.

### Gateway Configuration

```bash
# Through RD Gateway
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /gateway:gateway.example.com \
  /gateway-username:admin \
  /gateway-password:Password123 \
  /gateway-domain:CONTOSO
```

**Explanation:** RD Gateway provides RDP access through a proxy, useful for NAT traversal and security.

### Load Balance and High Availability

```bash
# Connect to specific session
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /admin /disp:1920x1080
```

**Explanation:** The `/admin` flag connects to the console session for administrative tasks.

### Performance Tuning

```bash
# Optimize for bandwidth
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /bpp:16 \
  /gfx:avc444 \
  /rfx \
  /nsc \
  /compression
```

**Explanation:** Reducing color depth and enabling compression improves performance on slow connections.

### Audio and Multimedia

```bash
# Enable audio
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /sound:sys:pulse \
  /mic:sys:pulse
```

**Explanation:** Audio and microphone redirection enables multimedia remote access.

---

## Chapter 5: Advanced Usage

### Advanced Authentication

#### CredSSP Authentication

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /sec:credssp
```

**Explanation:** CredSSP provides enhanced credential protection during RDP authentication.

#### NLA Authentication

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /sec:nla
```

**Explanation:** NLA requires authentication before establishing a session, reducing attack surface.

#### RDP Security (Legacy)

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /sec:rdp
```

**Explanation:** RDP security is legacy and less secure. Use only when NLA/CredSSP are unavailable.

### SSL/TLS Configuration

```bash
# Custom certificate
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /cert:fb660dc3ac0de4648079bb25214434c8d7e28f37 \
  /cert-name:"CN=Target"

# Ignore certificate validation
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /cert-ignore
```

**Explanation:** Certificate management options for SSL/TLS connections.

### Channel Configuration

```bash
# Enable specific channels
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /channels:rdpsnd,rdpdr,cliprdr

# Disable specific channels
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /no-sound \
  /no-clipboard
```

**Explanation:** RDP channels provide additional functionality. Controlling them improves security and performance.

### Keyboard and Input

```bash
# Map keyboard layout
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /keyboard:0x0409

# Grab keyboard
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /grab-keyboard
```

**Explanation:** Keyboard configuration ensures correct input mapping for different locales.

### Display Configuration

```bash
# Multi-monitor
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /multimon /f

# Dynamic resolution
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /dynamic-resolution

# Specific monitor
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /monitor:1
```

**Explanation:** Display options for multi-monitor setups and dynamic resizing.

### Clipboard and File Operations

```bash
# Clipboard with format
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /clipboard \
  /clipboard-clear-delay:0

# Multiple drive redirections
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /drive:share1,/tmp/share1 \
  /drive:share2,/tmp/share2
```

**Explanation:** Advanced clipboard and file transfer options for productivity.

### Proxy Configuration

```bash
# SOCKS proxy
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /proxy:type:socks,hostname:127.0.0.1,port:1080

# HTTP proxy
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /proxy:type:http,hostname:proxy.example.com,port:8080
```

**Explanation:** Proxy support enables RDP connections through intermediary servers.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Get graphical access to a Windows target

**Steps:**

```bash
# 1. Find RDP enabled
nmap -p 3389 192.168.1.0/24

# 2. Test credentials
xfreerdp /v:192.168.1.100 /u:admin /p:password /cert-ignore

# 3. If you have hash only
xfreerdp /v:192.168.1.100 /u:admin /d:WORKGROUP \
  /pth:'aad3b435b51404ee...' /cert-ignore

# 4. Transfer tools via drive redirection
xfreerdp /v:192.168.1.100 /u:admin /pth:'LM:NT' \
  /drive:tools,/tmp/tools /cert-ignore
```

**Explanation:** Standard CTF workflow from discovery to exploitation using pass-the-hash.

### Scenario 2: Penetration Test

**Objective:** Lateral movement via RDP

**Steps:**

```bash
# 1. Initial access with pass-the-hash
xfreerdp /v:192.168.1.100 /u:Administrator /d:CONTOSO \
  /pth:'LM:NT' /cert-ignore

# 2. Enumerate from Windows desktop
# 3. Extract credentials from target
# 4. Move to next host
xfreerdp /v:192.168.1.200 /u:Administrator /d:CONTOSO \
  /pth:'new_LM:new_NT' /cert-ignore
```

**Explanation:** Pass-the-hash RDP for lateral movement through Windows networks.

### Scenario 3: Red Team Operation

**Objective:** Covert RDP access

**Steps:**

```bash
# 1. Connect through proxy
xfreerdp /v:192.168.1.100 /u:Administrator /d:CONTOSO \
  /pth:'LM:NT' \
  /proxy:type:socks,hostname:127.0.0.1,port:1080 \
  /cert-ignore \
  /log-level:OFF

# 2. Use RemoteApp for stealth
xfreerdp /v:192.168.1.100 /u:Administrator /d:CONTOSO \
  /pth:'LM:NT' \
  /app:"C:\Windows\System32\cmd.exe" \
  /cert-ignore
```

**Explanation:** Covert RDP access using proxy and RemoteApp to minimize detection.

### Scenario 4: Credential Testing

**Objective:** Test RDP access across network

**Steps:**

```bash
# 1. Single host test
xfreerdp /v:192.168.1.100 /u:admin /p:password /cert-ignore

# 2. Pass-the-hash test
xfreerdp /v:192.168.1.100 /u:admin /pth:'LM:NT' /cert-ignore

# 3. Automated testing with CrackMapExec
crackmapexec rdp 192.168.1.0/24 -u admin -p password
crackmapexec rdp 192.168.1.0/24 -u admin -H 'LM:NT'
```

**Explanation:** Combine FreeRDP with CrackMapExec for comprehensive RDP testing.

---

## Chapter 7: Detection & Defense

### How Defenders Detect FreeRDP

#### Network Indicators

- **RDP connections:** Unusual sources connecting to port 3389
- **Protocol anomalies:** Non-standard RDP negotiation patterns
- **Traffic volume:** Large data transfers over RDP

#### Host-Based Indicators

- **RDP session logs:** Event IDs 4624 (Type 10), 4625
- **Security events:** RDP authentication attempts
- **Process creation:** Unusual processes spawned by RDP sessions

#### Log Analysis

```bash
# Key Windows Event IDs
# 4624 - Logon (Type 10 = RDP)
# 4625 - Failed logon
# 4778 - Session reconnected
# 4779 - Session disconnected
```

**Explanation:** Multiple event IDs track RDP activity for detection.

### Mitigation Strategies

#### Preventive Controls

1. **Network Level Authentication:** Require NLA for all RDP connections
2. **Firewall rules:** Restrict RDP to authorized management networks
3. **RDP gateway:** Centralize RDP access through a gateway
4. **Jump servers:** Use dedicated jump servers for RDP access

#### Detective Controls

1. **Monitor RDP logons:** Alert on unusual RDP sources
2. **Track session activity:** Monitor what users do during RDP sessions
3. **Network monitoring:** Detect RDP traffic patterns
4. **Behavioral analytics:** Identify anomalous RDP usage

### Blue Team Response

1. **Enable NLA** to require pre-authentication
2. **Restrict RDP access** via GPO and firewall rules
3. **Deploy RDP gateway** for centralized access control
4. **Implement privileged access workstations** (PAWs)
5. **Monitor for pass-the-hash** using Windows Event IDs

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "TLS handshake failed"

**Cause:** Certificate issues or TLS version mismatch

**Solution:**

```bash
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 \
  /cert-ignore /sec:tls
```

**Explanation:** Ignoring certificate validation resolves most TLS handshake issues.

#### Error: "Authentication failure"

**Cause:** Incorrect credentials or NLA required

**Solution:**

```bash
# Try with NLA
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 /sec:nla

# Or without NLA
xfreerdp /v:192.168.1.100 /u:Administrator /p:Password123 /sec:rdp
```

**Explanation:** Different security protocols may be required depending on server configuration.

#### Error: "Pass-the-hash not working"

**Cause:** Hash format incorrect or target doesn't support PtH

**Solution:**

```bash
# Ensure correct hash format (LM:NT)
xfreerdp /v:192.168.1.100 /u:Administrator \
  /pth:'aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0' \
  /cert-ignore

# Some systems require empty LM hash
xfreerdp /v:192.168.1.100 /u:Administrator \
  /pth:'00000000000000000000000000000000:31d6cfe0d16ae931b73c59d7e0c089c0' \
  /cert-ignore
```

**Explanation:** Pass-the-hash requires the correct LM:NT format. Some systems reject non-zero LM hashes.

#### Error: "Connection refused"

**Cause:** RDP not enabled or firewall blocking

**Solution:**

```bash
# Enable RDP on target
# System Properties > Remote > Allow remote connections

# Check if port is open
nmap -p 3389 192.168.1.100
```

**Explanation:** RDP must be enabled and the port must be accessible.

### FAQ

**Q: Does FreeRDP support pass-the-hash?**
A: Yes. Use the `/pth` flag with the LM:NT hash format.

**Q: Can FreeRDP connect through a SOCKS proxy?**
A: Yes. Use `/proxy:type:socks,hostname:127.0.0.1,port:1080`.

**Q: What's the difference between xfreerdp and rdesktop?**
A: FreeRDP (xfreerdp) is actively maintained, supports modern RDP features, and includes pass-the-hash. rdesktop is deprecated.

**Q: Can FreeRDP do RemoteApp?**
A: Yes. Use `/app:"path\to\app.exe"` to launch specific applications.

**Q: How do I transfer files via RDP?**
A: Use drive redirection: `/drive:name,/local/path`.

---

## Resources

### Official Documentation

- [FreeRDP GitHub](https://github.com/FreeRDP/FreeRDP)
- [FreeRDP Wiki](https://github.com/FreeRDP/FreeRDP/wiki)
- [FreeRDP Man Pages](https://github.com/FreeRDP/FreeRDP/wiki/FAQ)
- [FreeRDP API Documentation](https://pub.freerdp.com/api/)

### Tutorials

- [FreeRDP Usage Guide - HackTricks](https://book.hacktricks.xyz/)
- [RDP Security - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/protocols/rdp)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)

### Related Tools

- **rdesktop:** Legacy RDP client (deprecated)
- **Remmina:** GUI RDP client
- **xfreerdp:** FreeRDP's command-line client
- **CrackMapExec:** Network pentesting with RDP support

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [VulnHub](https://www.vulnhub.com/)
