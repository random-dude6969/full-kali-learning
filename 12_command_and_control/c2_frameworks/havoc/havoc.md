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
Havoc is a modern and malleable post-exploitation command and control framework. It features a cross-platform GUI client written in C++/Qt, a Go-based teamserver supporting multiplayer operations, and a flagship agent called "Demon" written in C and ASM. Havoc emphasizes operator flexibility, modular design, and customizable C2 profiles.

### History & Background
- **Created:** 2022 by @C5pider
- **Maintained:** Archived February 2026 (8.5k stars, 1.2k forks)
- **License:** GPL-3.0
- **Purpose:** Post-exploitation C2, red team operations, adversary emulation

Havoc was designed to be as malleable and modular as possible, giving operators the capability to add custom features or modules that evade target detection systems. It is not designed to be inherently evasive but rather to provide the framework for building evasive capabilities.

### When to Use This Tool
- **Red team engagements:** Full-spectrum adversary emulation with custom agents
- **Penetration testing:** Post-exploitation activities requiring stealth and flexibility
- **Custom C2 development:** Building bespoke agents and communication protocols
- **Team operations:** Multi-operator collaboration with session sharing

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- On production systems without a defined scope and rules of engagement
- When the project's archived status is a concern (no new updates)

### Legal and Ethical Considerations
- **Always obtain written permission** before deploying C2 infrastructure
- **Understand the scope** - know exactly what systems you are authorized to access
- **Document all activity** - maintain logs of commands executed and access obtained
- **Clean up thoroughly** - remove all artifacts after engagement completion

### Key Concepts
- **Demon:** Havoc's flagship implant agent (written in C and ASM)
- **Teamserver:** Go-based server supporting multiple operators
- **C2 Profiles:** Customizable communication patterns to blend with legitimate traffic
- **External C2:** Interface for connecting custom agents and tools
- **Module System:** Extensible architecture for post-exploitation capabilities

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 2 GB (4 GB recommended)
- **Disk Space:** 500 MB for full build
- **OS:** Debian 10/11, Ubuntu 20.04/22.04, Kali Linux
- **Build Tools:** Qt, Python 3.10+, Go, C++ compiler
- **Privileges:** Root required for server operations

### Installation from Source
```bash
# Clone the repository
git clone --recursive https://github.com/HavocFramework/Havoc.git
cd Havoc

# Build the teamserver
cd teamserver
go mod download
go build -o havoc-server

# Build the client
cd ../client
mkdir build && cd build
cmake ..
make -j$(nproc)
```

### Quick Start
```bash
# Start the teamserver
./havoc-server --profile ../profiles/havoc.yaotl -v

# Start the client (in another terminal)
./havoc-client
```

### Configuration
Edit the teamserver configuration file:
```yaml
# profiles/havoc.yaotl
Teamserver:
  Host: "0.0.0.0"
  Port: 40056
  User: "admin"
  Password: "password123"

Listeners:
  - Name: "http_listener"
    Type: "HTTP"
    Host: "http://your_ip:8080"
    Port: 8080
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Havoc uses a three-tier architecture:
- **Client:** Cross-platform Qt GUI for operator interaction
- **Teamserver:** Go-based server managing sessions, listeners, and operators
- **Demon Agent:** C/ASM implant running on target hosts

### Demon Agent Features
- Sleep obfuscation via Ekko, Ziliean, or FOLIAGE
- x64 return address spoofing
- Indirect syscalls for Nt* APIs
- SMB support for lateral movement
- Token vault for impersonation
- AMSI/ETW patching via hardware breakpoints
- Proxy library loading
- Stack duplication during sleep

### Communication Flow
```
Operator -> Qt Client -> Go Teamserver -> HTTP/S Listener -> Demon Agent (Target)
```

### C2 Profile System
C2 profiles define how the agent communicates:
- **Request patterns:** URI paths, query parameters, headers
- **Response patterns:** Command encoding, sleep intervals
- **Metadata:** How the agent identifies itself to the server
- **Encryption:** Communication encryption settings

---

## Chapter 4: Basic Usage

### Starting the Framework
```bash
# Terminal 1: Start teamserver
cd Havoc
./teamserver/havoc-server --profile profiles/havoc.yaotl

# Terminal 2: Start client
cd Havoc/client
./build/havoc-client
```

### Connecting to Teamserver
1. Launch the Havoc client
2. Enter connection details:
   - Host: `127.0.0.1` (or teamserver IP)
   - Port: `40056`
   - User: `admin`
   - Password: `password123`
3. Click "Connect"

### Generating a Demon Payload
```bash
# Via GUI: Listeners > Generate Payload
# Select listener, architecture, and output format
```

### Setting Up a Listener
```bash
# Via GUI: Listeners > Add Listener
# Configure:
#   - Name: http_listener
#   - Type: HTTP
#   - Host: http://your_ip:8080
#   - Port: 8080
```

### Managing Sessions
```bash
# View sessions in the "Sessions" tab
# Right-click a session for options:
#   - Interact (open console)
#   - Kill (terminate agent)
#   - Background (minimize session)
```

---

## Chapter 5: Advanced Usage

### Post-Exploitation with Demon
```bash
# Once you have a Demon session, use built-in commands:
sysinfo                    # System information
getuid                     # Current user
getprivs                   # Enabled privileges
hashdump                   # Dump password hashes
ps                         # List processes
migrate <pid>              # Migrate to another process
download <file>            # Download file from target
upload <file>              # Upload file to target
```

### Lateral Movement
```bash
# Use token impersonation
token::impersonate

# Use SMB lateral movement
shell > Invoke-TokenManipulation -Enumerate
```

### Pivoting
```bash
# Set up SOCKS proxy through session
# Right-click session > Pivoting > SOCKS Proxy
# Configure local port and proxy settings
```

### External C2
Havoc supports connecting external agents:
```python
# Example External C2 connection
from havoc_external_c2 import ExternalC2

c2 = ExternalC2("127.0.0.1", 40056)
c2.connect()
# Implement your custom agent communication
```

### Custom Modules
```bash
# Load custom modules via the GUI
# Modules > Load Module > Select .so/.dll file
```

---

## Chapter 6: Integration & Automation

### Python API
Havoc provides a Python API for automation:
```python
from havoc import Havoc

# Connect to teamserver
h = Havoc("127.0.0.1", 40056, "admin", "password123")
h.connect()

# List sessions
sessions = h.sessions()

# Execute command on session
h.execute(session_id, "whoami")
```

### Integration with Other Tools
- **Custom Agents:** Build and connect your own agents via External C2
- **Talon:** Official alternative agent (https://github.com/HavocFramework/Talon)
- **Modules:** Community modules for additional functionality

### Automation Workflows
```bash
# Automate common tasks with the Python API
# Create scripts for:
#   - Automated reconnaissance
#   - Session management
#   - Reporting
```

---

## Chapter 7: OPSEC & Defense Evasion

### Demon Agent Evasion
- **Sleep obfuscation:** Ekko/Ziliean/FOLIAGE to hide memory
- **Indirect syscalls:** Bypass userland hooks
- **Return address spoofing:** Hide call stack
- **AMSI/ETW patching:** Disable detection at runtime
- **Stack duplication:** Mimic legitimate sleep behavior

### C2 Profile Customization
```yaml
# Custom C2 profile example
HTTP:
  Method: "GET"
  URI: "/api/v1/data"
  Headers:
    - "User-Agent: Mozilla/5.0"
    - "Authorization: Bearer <token>"
  Response:
    Type: "JSON"
    Sleep: 60
```

### Best Practices
1. Always customize C2 profiles to match the target environment
2. Use legitimate-looking domains and certificates
3. Implement sleep jitter to avoid pattern detection
4. Monitor for EDR/AV detection and adapt
5. Test payloads against target defenses before deployment

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Build fails with Qt errors**
```bash
# Install required Qt packages
sudo apt install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools

# Ensure Python 3.10+
python3 --version
```

**Problem: Teamserver won't start**
```bash
# Check Go installation
go version

# Verify dependencies
cd teamserver && go mod tidy

# Check port availability
netstat -tlnp | grep 40056
```

**Problem: Client can't connect**
- Verify teamserver is running
- Check firewall rules on port 40056
- Ensure credentials match configuration
- Check for SSL/TLS issues

**Problem: Demon agent doesn't check in**
- Verify the payload was generated correctly
- Check network connectivity from target
- Ensure the listener is running and accessible
- Look for AV/EDR blocking the agent

### Debug Mode
```bash
# Start teamserver with verbose output
./havoc-server --profile profiles/havoc.yaotl -v

# Check agent logs
# The Demon agent outputs debug information to stderr
```

---

## Resources

- **GitHub Repository:** https://github.com/HavocFramework/Havoc
- **Kali Tools Page:** https://www.kali.org/tools/havoc/
- **Havoc Website:** https://havocframework.com
- **Talon Agent:** https://github.com/HavocFramework/Talon
- **Python API:** https://github.com/HavocFramework/havoc-py
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **Related Tools:** Adaptix C2, PowerShell Empire, Metasploit Framework
