---
tool_name: "dirbuster"
display_name: "DirBuster Cheatsheet"
---

# DirBuster — Quick Reference Cheatsheet

## GUI Launch

```bash
# Launch GUI
dirbuster

# Or with explicit Java
java -jar /usr/share/dirbuster/dirbuster.jar
```

## CLI Usage

```bash
# Basic scan
dirbuster -u https://target.com -l /usr/share/wordlists/dirb/common.txt

# With extensions
dirbuster -u https://target.com -l wordlist.txt -e php,html,txt

# With threads
dirbuster -u https://target.com -l wordlist.txt -t 20

# With report
dirbuster -u https://target.com -l wordlist.txt -r report.txt

# Force extensions
dirbuster -u https://target.com -l wordlist.txt -e php -x

# Lowercase only
dirbuster -u https://target.com -l wordlist.txt -c

# Verbose
dirbuster -u https://target.com -l wordlist.txt -v
```

## CLI Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-u` | Target URL | `dirbuster -u https://target.com` |
| `-l` | Wordlist file | `dirbuster -u URL -l wordlist.txt` |
| `-e` | Extensions | `dirbuster -u URL -e php,html,txt` |
| `-t` | Threads | `dirbuster -u URL -t 20` |
| `-x` | Force extensions | `dirbuster -u URL -x` |
| `-c` | Lowercase only | `dirbuster -u URL -c` |
| `-z` | Wildcard detection | `dirbuster -u URL -z` |
| `-r` | Report file | `dirbuster -u URL -r report.txt` |
| `-f` | Report format | `dirbuster -u URL -f html` |
| `-p` | Proxy | `dirbuster -u URL -p http://127.0.0.1:8080` |
| `-a` | User-Agent | `dirbuster -u URL -a "Mozilla/5.0"` |
| `-H` | Custom header | `dirbuster -u URL -H "Auth: token"` |
| `-v` | Verbose | `dirbuster -u URL -v` |
| `-h` | Help | `dirbuster -h` |

## GUI Configuration

| Setting | Description | Recommended |
|---------|-------------|-------------|
| Target URL | URL to scan | Required |
| Threads | Concurrent requests | 10-50 |
| Word List | Path to wordlist | common.txt |
| Extensions | File extensions | php,html,txt |
| Recursive | Scan found dirs | For thorough scans |
| Wildcard | Detect wildcards | Enable |

## Common Flag Combinations

| Use Case | Command |
|----------|---------|
| Quick scan | `dirbuster -u URL -l common.txt` |
| Thorough scan | `dirbuster -u URL -l big.txt -e php,html,txt -t 30` |
| Authenticated | `dirbuster -u URL -l wordlist.txt -H "Auth: Bearer TOKEN"` |
| Stealth | `dirbuster -u URL -l wordlist.txt -p http://127.0.0.1:8080 -t 5` |
| WordPress | `dirbuster -u URL -l wp_wordlist.txt -e php -r wp_report.txt` |
