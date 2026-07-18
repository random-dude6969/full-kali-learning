# Starkiller Cheatsheet

## Quick Reference

### Accessing Starkiller

```bash
# Bundled with Empire 5.0+
# URL: http://localhost:1337

# Manual installation
git clone https://github.com/BC-SECURITY/Starkiller.git
cd Starkiller
yarn install
yarn dev
# URL: http://localhost:8080

# Default credentials:
# Username: empireadmin
# Password: password123
```

### Connection Setup

```bash
# Configure Empire connection
# Settings > Empire Connection
# URL: http://localhost:1337
# Username: empireadmin
# Password: password123
```

### Listener Management

```bash
# Via Web Interface:
# 1. Click "Listeners" in sidebar
# 2. Click "Create Listener"
# 3. Select type (HTTP, HTTPS, etc.)
# 4. Configure options
# 5. Click "Save"

# Common settings:
# Name: http_listener
# Type: HTTP
# Host: http://192.168.1.100:8080
# Port: 8080
```

### Stager Generation

```bash
# Via Web Interface:
# 1. Click "Stagers" in sidebar
# 2. Select stager type
# 3. Configure listener and options
# 4. Click "Generate"
# 5. Copy payload

# Common stagers:
# - multi/launcher (PowerShell)
# - windows/exe
# - windows/dll
# - windows/launcher_bat
```

### Agent Management

```bash
# Via Web Interface:
# 1. Click "Agents" in sidebar
# 2. View list of active agents
# 3. Click agent to interact
# 4. Execute commands or modules

# Agent actions:
# - Interact (open console)
# - Kill (terminate agent)
# - Background (minimize)
# - View properties
```

### Module Execution

```bash
# Via Web Interface:
# 1. Click "Modules" in sidebar
# 2. Search for module
# 3. Select module
# 4. Configure options
# 5. Select target agent
# 6. Click "Execute"

# Common modules:
# - powershell/credentials/mimikatz/logonpasswords
# - powershell/situational_awareness/host/seatbelt
# - powershell/situational_awareness/network/portscan
```

### Dashboard Features

```bash
# Dashboard overview:
# - Active agents count
# - Listener status
# - Event log
# - Statistics charts

# Real-time updates:
# - Live agent connections
# - Module execution results
# - Event feed
```

### Graph View

```bash
# Access graph view:
# Click "Graph" in sidebar

# Features:
# - Agent relationships
# - Pivot paths
# - Network topology
# - Visual connections
```

### Process Browser

```bash
# Access process browser:
# 1. Select agent
# 2. Click "Process Browser"

# Actions:
# - View processes
# - Migrate to process
# - Inject into process
# - Kill process
```

### File Browser

```bash
# Access file browser:
# 1. Select agent
# 2. Click "File Browser"

# Actions:
# - Browse filesystem
# - Upload files
# - Download files
# - Execute files
```

### Bypass Management

```bash
# Access bypass management:
# 1. Select agent
# 2. Click "Bypasses"

# Available bypasses:
# - AMSI bypass
# - ETW bypass
# - AppLocker bypass

# Enable/disable bypasses per agent
```

### Malleable Profile Management

```bash
# Access profile management:
# Settings > Malleable Profiles

# Actions:
# - Import profiles
# - Apply to listeners
# - Customize C2 traffic
```

### API Usage

```bash
# Authenticate
curl -X POST http://localhost:1337/api/auth -d '{"username":"empireadmin","password":"password123"}'

# List agents
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/agents

# Execute module
curl -X POST -H "Authorization: Bearer <token>" http://localhost:1337/api/agents/<agent>/execute -d '{"module":"powershell/credentials/mimikatz/logonpasswords"}'

# List listeners
curl -H "Authorization: Bearer <token>" http://localhost:1337/api/listeners
```

### Multi-operator Support

```bash
# Multiple operators can connect:
# 1. Start Empire server
# 2. Multiple browsers connect to same URL
# 3. Shared agents and listeners
# 4. Real-time updates across all clients
```

### OPSEC Tips

```bash
# Use HTTPS for Starkiller access
# Configure SSL in Empire server

# Implement IP whitelisting
# Settings > Security > IP Whitelist

# Use strong authentication
# Change default password

# Monitor access logs
# Settings > Logs
```

### Troubleshooting

```bash
# Starkiller won't load
node --version  # Check Node.js 16+
yarn install    # Reinstall dependencies
# Clear browser cache

# Can't connect to Empire
# Check: Empire server running
# Verify: API endpoint URL
# Test: credentials

# Modules don't execute
# Check: agent active
# Verify: module options
# Look: error messages

# Real-time updates not working
# Check: WebSocket connection
# Verify: Empire server responding
# Monitor: browser console
```

### Keyboard Shortcuts

```bash
# Common shortcuts:
# Ctrl+K: Quick search
# Ctrl+/: Help
# Esc: Close dialog
# Tab: Switch panels
```

### Settings Configuration

```bash
# Access settings:
# Click gear icon in sidebar

# Options:
# - Empire connection
# - Theme (dark/light)
# - Notifications
# - Auto-refresh interval
# - Default module options
```
