# rdesktop Cheatsheet

## Basic Syntax

```bash
rdesktop [options] server[:port]
```

---

## Authentication

### Username/Password

```bash
rdesktop -u administrator -p 'P@ssw0rd' 192.168.1.10
```

### Prompt for Password

```bash
rdesktop -u administrator -p - 192.168.1.10
```

### Domain Authentication

```bash
rdesktop -u administrator -p 'P@ssw0rd' -d CORP 192.168.1.10
```

### Workgroup (default)

```bash
rdesktop -u administrator -p 'P@ssw0rd' -d . 192.168.1.10
```

---

## Display Options

| Flag | Description |
|------|-------------|
| `-f` | Full-screen mode |
| `-g WxH` | Desktop geometry (e.g., `1920x1080`) |
| `-g WxH@DPI` | Geometry with DPI (e.g., `1920x1080@120`) |
| `-g workarea` | 75% of local work area |
| `-a DEPTH` | Colour depth (8, 15, 16, 24, 32) |
| `-D` | Disable window manager decorations |
| `-K` | Keep window manager key bindings |
| `-T TITLE` | Window title |
| `-B` | Use X-server BackingStore |

---

## Session Options

| Flag | Description |
|------|-------------|
| `-0` | Attach to console session (session 0) |
| `-4` | Force RDP version 4 |
| `-5` | Force RDP version 5 (default) |
| `-N` | Enable numlock synchronization |
| `-t` | Disable use of remote Ctrl |
| `-b` | Force bitmap updates |
| `-P` | Persistent bitmap caching |
| `-z` | Enable RDP compression |
| `-x EXPERIENCE` | RDP5 experience (m=broadband, b=broadband, l=LAN) |

---

## Device Redirection

### Drive Redirection

```bash
# Mount local /mnt/data as share "data"
rdesktop -r disk:data=/mnt/data 192.168.1.10

# Multiple drives
rdesktop -r disk:data=/mnt/data,tools=/tmp/tools 192.168.1.10

# Access on remote: \\tsclient\data
```

### Clipboard

```bash
# Standard clipboard (default)
rdesktop -r clipboard:CLIPBOARD 192.168.1.10

# Both primary and clipboard
rdesktop -r clipboard:PRIMARYCLIPBOARD 192.168.1.10

# Disable clipboard
rdesktop -r clipboard:off 192.168.1.10
```

### Sound

```bash
# Redirect to local (ALSA)
rdesktop -r sound:local 192.168.1.10

# Redirect to local (PulseAudio)
rdesktop -r sound:local:pulse 192.168.1.10

# Leave sound on remote
rdesktop -r sound:remote 192.168.1.10

# Disable sound
rdesktop -r sound:off 192.168.1.10
```

### Printer

```bash
rdesktop -r printer:myprinter 192.168.1.10
```

### Serial Port

```bash
rdesktop -r comport:COM1=/dev/ttyS0 192.168.1.10
rdesktop -r comport:COM1=/dev/ttyS0,COM2=/dev/ttyS1 192.168.1.10
```

### Parallel Port

```bash
rdesktop -r lptport:LPT1=/dev/lp0 192.168.1.10
```

### Smartcard

```bash
rdesktop -i -o sc-card-name="eToken" -p - 192.168.1.10
```

---

## Quick Reference

```bash
# Basic connection
rdesktop 192.168.1.10

# With credentials
rdesktop -u admin -p 'password' 192.168.1.10

# Full screen with drive sharing
rdesktop -f -r disk:data=/mnt/data -u admin -p 'pass' 192.168.1.10

# Specific resolution
rdesktop -g 1920x1080 -u admin -p 'pass' 192.168.1.10

# Low bandwidth
rdesktop -x m -z -a 8 -u admin -p 'pass' 192.168.1.10

# LAN performance
rdesktop -x l -z -P -a 32 -u admin -p 'pass' 192.168.1.10

# Keyboard layout
rdesktop -k de -u admin -p 'pass' 192.168.1.10

# Console session
rdesktop -0 -u admin -p 'pass' 192.168.1.10

# TLS 1.2
rdesktop -V 1.2 -u admin -p 'pass' 192.168.1.10

# German keyboard, custom resolution
rdesktop -k de -g 1280x720 -u admin -p 'pass' 192.168.1.10
```

---

## RDP Discovery

```bash
# Check if RDP is open
nmap -p 3389 192.168.1.10

# RDP version detection
nmap -sV -p 3389 192.168.1.10

# Scan for RDP across subnet
nmap -p 3389 192.168.1.0/24 --open

# Test connection (will show login screen)
rdesktop 192.168.1.10
```

---

## Common Patterns

### Quick File Transfer

```bash
# Mount local /tmp
rdesktop -r disk:share=/tmp -u admin -p 'pass' 192.168.1.10

# In remote session: \\tsclient\share
copy C:\file.txt \\tsclient\share\
```

### SSH Tunnel for RDP

```bash
# Create tunnel
ssh -L 3389:192.168.1.10:3389 user@jumpbox

# Connect through tunnel
rdesktop 127.0.0.1
```

### Multiple Sessions

```bash
rdesktop -u admin1 -p 'pass1' -T "Session1" 192.168.1.10 &
rdesktop -u admin2 -p 'pass2' -T "Session2" 192.168.1.10 &
```

---

## Error Handling

| Error | Fix |
|-------|-----|
| `Connection refused` | RDP disabled; check port 3389 |
| `Connection timed out` | Firewall blocking; check routing |
| `Protocol error` | NLA required; use xfreerdp or disable NLA |
| Black screen | Resolution mismatch; try `-g 1024x768` |
| Keyboard issues | Wrong layout; use `-k en-us` |
| No sound | ALSA not configured; check `alsamixer` |
| Clipboard not working | Ensure `-r clipboard:CLIPBOARD` |
| Drive not accessible | Access via `\\tsclient\<sharename>` |
| TLS error | Try `-V 1.2` or `-V 1.0` |

---

## Detection Indicators

- **Event ID 4624:** Logon Type 10 (RemoteInteractive) on target
- **Port 3389:** RDP traffic between hosts
- **Clipboard activity:** Unusual clipboard patterns
- **Drive redirection:** File access via `\\tsclient\`
- **Bitmap cache:** Local cache file creation
- **Process:** `rdesktop` process on Linux attacker
