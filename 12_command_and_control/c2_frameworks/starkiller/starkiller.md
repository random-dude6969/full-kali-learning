---
tool_name: "starkiller"
display_name: "Starkiller"
mitre_tactic: "command_and_control"
mitre_tactic_id: "TA0011"
mitre_subcategory: "c2_frameworks"
install_command: "git clone https://github.com/BC-SECURITY/Starkiller.git"
version: "3.5.0"
github_repo: "https://github.com/BC-SECURITY/Starkiller"
kali_tools_page: "https://www.kali.org/tools/starkiller/"
difficulty_level: "intermediate"
requires_root: false
gui_available: true
tags: ["c2", "gui", "web", "empire", "frontend", "vue"]
related_tools: ["powershell-empire", "cobalt-strike", "havoc"]
---

# Starkiller

## Chapter 1: Introduction & Overview

### What is Starkiller?
Starkiller is a web-based frontend (GUI) for PowerShell Empire. It is a VueJS web application that interfaces remotely with Empire via its API, providing a modern, intuitive interface for managing C2 operations. Starkiller can be used as a replacement for the Empire client or in a mixed environment with both Starkiller and Empire clients.

### History & Background
- **Created:** 2021 by BC-SECURITY
- **Maintained:** Active project (1.7k stars, 245 forks)
- **License:** MIT
- **Purpose:** Web-based GUI for Empire, visual C2 management, team collaboration

Starkiller was designed to provide a modern, web-based alternative to Empire's command-line interface, making C2 operations more accessible and providing features like visual dashboards, graph views, and real-time updates.

### When to Use This Tool
- **Red team operations:** Visual management of Empire C2 infrastructure
- **Penetration testing:** Quick access to Empire modules and agents
- **Purple team exercises:** Visual demonstration of C2 capabilities
- **Training:** Learning Empire through an intuitive interface

### When NOT to Use This Tool
- Without explicit written authorization from the target organization
- When Empire is not available or not the preferred C2 framework
- For command-line automation (use Empire client directly)
- When web-based access is not desired

### Legal and Ethical Considerations
- **Always obtain written permission** before using Starkiller/Empire
- **Understand the scope** - know exactly what systems you are authorized to access
- **Secure the web interface** - use authentication and HTTPS
- **Document all activity** - maintain records of all access obtained

### Key Concepts
- **Frontend:** Web-based user interface (VueJS)
- **API Communication:** Interfaces with Empire server via REST API
- **Real-time Updates:** Live updates of agents, listeners, and modules
- **Visual Dashboard:** Graphical representation of C2 operations
- **Multi-operator:** Supports multiple simultaneous operators

---

## Chapter 2: Installation & Setup

### System Requirements
- **Minimum RAM:** 1 GB
- **Disk Space:** 200 MB
- **Node.js:** 16+
- **Yarn:** 1.22+
- **Empire:** Running Empire server

### Bundled Installation (Empire 5.0+)
As of Empire 5.0, Starkiller is prepackaged and served via Empire's API:
```bash
# Starkiller is automatically available at:
http://localhost:1337
```

### Manual Installation
```bash
# Clone the repository
git clone https://github.com/BC-SECURITY/Starkiller.git
cd Starkiller

# Install dependencies
yarn

# Start development server
yarn dev

# Build for production
yarn build
```

### Configuration
```bash
# Configure Empire connection
# Edit src/config.js or use the web interface

# Default settings:
# Empire URL: http://localhost:1337
# Default credentials: empireadmin / password123
```

### Starting Starkiller
```bash
# If bundled with Empire:
# Access at http://localhost:1337

# If installed separately:
yarn dev
# Access at http://localhost:8080
```

---

## Chapter 3: Core Concepts

### Architecture Overview
Starkiller follows a web application architecture:
- **Frontend:** VueJS single-page application
- **Backend API:** Empire server REST API
- **Database:** Empire's database (SQLite/PostgreSQL)
- **Authentication:** API token-based authentication

### Interface Components
- **Dashboard:** Overview of agents, listeners, and activity
- **Agents:** List and management of active agents
- **Listeners:** Configure and manage C2 listeners
- **Modules:** Browse and execute post-exploitation modules
- **Credentials:** View and manage harvested credentials
- **Events:** Real-time event log

### Communication Flow
```
Browser (Starkiller) -> REST API -> Empire Server -> Agent (Target)
```

### Authentication
- Default credentials: `empireadmin` / `password123`
- API token-based authentication
- Support for HTTPS and custom certificates

---

## Chapter 4: Basic Usage

### Accessing Starkiller
```bash
# Start Empire server
cd Empire && ./ps-empire server

# Access Starkiller in browser
# URL: http://localhost:1337
# Username: empireadmin
# Password: password123
```

### Creating a Listener
1. Click "Listeners" in the sidebar
2. Click "Create Listener"
3. Select listener type (HTTP, HTTPS, etc.)
4. Configure options (host, port, etc.)
5. Click "Save"

### Generating a Stager
1. Click "Stagers" in the sidebar
2. Select a stager type
3. Configure the listener and options
4. Click "Generate"
5. Copy the payload and execute on target

### Managing Agents
1. Click "Agents" in the sidebar
2. View list of active agents
3. Click an agent to interact
4. Execute commands or modules

### Executing Modules
1. Click "Modules" in the sidebar
2. Search for a module
3. Select the module
4. Configure options
5. Select target agent
6. Click "Execute"

---

## Chapter 5: Advanced Usage

### Visual Dashboard
The dashboard provides:
- **Agent overview:** Active agents, their status, and metadata
- **Listener status:** Active listeners and their configurations
- **Event log:** Real-time feed of all actions
- **Statistics:** Charts and graphs of activity

### Graph View
Starkiller includes a graph view for visualizing:
- **Agent relationships:** Connections between agents
- **Pivot paths:** Lateral movement routes
- **Network topology:** Discovered network structure

### Process Browser
- View processes on target systems
- Migrate to different processes
- Inject into legitimate processes

### Bypass Management
- View and manage AMSI/ETW bypasses
- Enable/disable bypasses per agent
- Custom bypass scripts

### Malleable Profile Management
- Import and manage Cobalt Strike malleable profiles
- Apply profiles to listeners
- Customize C2 traffic patterns

### File Browser
- Browse filesystem on target systems
- Upload and download files
- Execute files remotely

---

## Chapter 6: Integration & Automation

### Empire API Integration
Starkiller communicates with Empire via REST API:
```bash
# Example API calls (for automation)

# Authenticate
curl -X POST http://localhost:1337/api/auth -d '{"username":"empireadmin","password":"password123"}'

# List agents
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/agents

# Execute module
curl -X POST -H "Authorization: Bearer <token>" http://localhost:1337/api/agents/<agent>/execute -d '{"module":"powershell/credentials/mimikatz/logonpasswords"}'
```

### Multi-operator Support
- Multiple operators can connect to the same Empire server
- Shared agents and listeners
- Real-time updates across all connected clients

### Integration with Other Tools
- **Empire:** Primary backend for all C2 operations
- **Metasploit:** Use for initial access, Empire for post-exploitation
- **Custom modules:** Extend via Empire's module system

### Automation Workflows
```bash
# Automate common tasks via API
# 1. Create listener
# 2. Generate stager
# 3. Wait for agent connection
# 4. Execute reconnaissance modules
# 5. Collect results
```

---

## Chapter 7: OPSEC & Defense Evasion

### Web Interface Security
- Use HTTPS for Starkiller access
- Implement IP whitelisting
- Use strong authentication
- Monitor access logs

### Empire Integration Evasion
- Use encrypted listeners (HTTPS)
- Implement malleable C2 profiles
- Use AMSI/ETW bypasses
- Apply obfuscation to payloads

### Best Practices
1. Always use HTTPS for Starkiller access
2. Implement strong authentication
3. Use malleable profiles to mimic legitimate traffic
4. Monitor for detection and adapt
5. Rotate infrastructure regularly

### Common Detection Methods
- Web server logs showing API access
- Unusual HTTP traffic patterns
- Authentication failures
- Module execution signatures

---

## Chapter 8: Troubleshooting & FAQ

### Common Issues

**Problem: Starkiller won't load**
```bash
# Check if Empire server is running
curl http://localhost:1337/api/auth

# Check Node.js version
node --version

# Reinstall dependencies
yarn install

# Clear browser cache
```

**Problem: Can't connect to Empire**
- Verify Empire server is running
- Check API endpoint URL
- Verify credentials
- Check firewall rules

**Problem: Modules don't execute**
- Verify agent is active
- Check module options
- Look for error messages in console
- Test with simpler modules

**Problem: Real-time updates not working**
- Check WebSocket connection
- Verify Empire server is responding
- Check browser console for errors

### Debug Mode
```bash
# Start Starkiller with debug output
yarn dev --debug

# Check Empire API logs
tail -f empire/logs/empire.log

# Monitor network requests in browser
# Open browser developer tools > Network tab
```

---

## Resources

- **GitHub Repository:** https://github.com/BC-SECURITY/Starkiller
- **Kali Tools Page:** https://www.kali.org/tools/starkiller/
- **Empire Wiki:** https://bc-security.gitbook.io/empire-wiki/
- **Starkiller Blog Post:** https://www.bc-security.org/post/an-introduction-to-starkiller
- **MITRE ATT&CK - Command and Control:** https://attack.mitre.org/tactics/TA0011/
- **Related Tools:** PowerShell Empire, Metasploit Framework, Havoc
