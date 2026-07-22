# Maltego - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Kali Linux
sudo apt install maltego
maltego

# Access Transform Hub
# Transform Hub > Browse Transforms
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Version | 4.11.3 |
| Java | OpenJDK 11 |
| Port | 8443 (API) |
| License | CE (Free) / Classic (Paid) |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Graph Analysis | Visual relationship mapping |
| Transform System | Extensible data gathering |
| OSINT Integration | Multiple data sources |
| Custom Entities | Create new entity types |
| Report Generation | PDF, HTML, PNG export |
| API Access | Programmatic control |

---

## Entity Types

| Entity | Description |
|--------|-------------|
| Person | Individual people |
| Email Address | Email addresses |
| Domain | Internet domains |
| IP Address | Network addresses |
| Phone Number | Phone numbers |
| Company | Organizations |
| Website | Web pages |
| Alias | Online usernames |
| Social Profile | Social media accounts |
| Document | Files and documents |

---

## Common Transforms

| Transform | Input | Output |
|-----------|-------|--------|
| To DNS Names | Domain | Subdomains |
| To IP Address | Domain | IP addresses |
| To MX Record | Domain | Mail servers |
| To Whois Info | Domain | Registrant info |
| To Person | Domain | Associated people |
| To Email | Domain | Email addresses |
| To Social Profile | Person | Social media |
| To Open Ports | IP | Services |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + N | New graph |
| Ctrl + O | Open graph |
| Ctrl + S | Save graph |
| Ctrl + A | Select all entities |
| Ctrl + F | Find entity |
| Delete | Remove selected |
| Ctrl + + | Zoom in |
| Ctrl + - | Zoom out |
| Ctrl + 0 | Reset zoom |
| F5 | Refresh |

---

## Graph Operations

| Action | Description |
|--------|-------------|
| Add Entity | Drag from palette |
| Connect | Right-click > Link to |
| Run Transform | Right-click > Run Transform |
| Color Code | Right-click > Set Color |
| Add Note | Right-click > Add Note |
| Group | Select multiple > Ctrl+G |

---

## OSINT Investigation Templates

### Domain Investigation
```
Start: example.com
├── To DNS Names
├── To IP Address
├── To MX Record
├── To Whois Info
├── To Person
└── To Email Address
```

### Email Investigation
```
Start: user@example.com
├── To Domain
├── To Person
├── To Social Profiles
├── To Phone Number
└── To Related Emails
```

### Person Investigation
```
Start: John Doe
├── To Email Addresses
├── To Social Profiles
├── To Aliases
├── To Phone Numbers
└── To Related People
```

---

## Import/Export

```bash
# Import Nmap XML
File > Import Graph > Nmap XML

# Import Nessus XML
File > Import Graph > Nessus XML

# Export graph as image
File > Export Graph > PNG

# Export report
Report > Generate Report > PDF
```

---

## Custom Transforms

```python
# Python transform example
from maltego_trx.maltego import MaltegoTransform
from maltego_trx.transform import DiscoverableTransform

class MyTransform(DiscoverableTransform):
    @classmethod
    def create_entities(cls, request, response):
        input_val = request.Value
        # Your logic here
        response.addEntity("maltego.Person", "Result")
```

---

## Troubleshooting

```bash
# Check Java version
java -version

# Set Java path
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

# Clear Maltego cache
rm -rf ~/.local/share/Maltego/

# Increase memory
# Edit maltego.ini
# Add: -Xmx4g
```

---

## Transform Hub

```
Transform Hub > Browse Transforms
├── Built-in transforms
├── Community transforms
├── Commercial transforms
└── Local transforms
```

---

## Report Types

| Type | Description |
|------|-------------|
| PDF | Formal report |
| HTML | Web-ready |
| PNG | Graph image |
| CSV | Data export |
| GraphML | Graph data |

---

## API Access

```bash
# Get API info
curl http://localhost:8443/api/v1/info

# List entities
curl -H "Authorization: Bearer TOKEN" \
    http://localhost:8443/api/v1/entities

# Run transform
curl -X POST http://localhost:8443/api/v1/transforms/run \
    -H "Content-Type: application/json" \
    -d '{"transform":"toIP","entity":{"type":"Domain","value":"example.com"}}'
```
