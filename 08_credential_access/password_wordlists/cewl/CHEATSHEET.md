# CeWL Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install cewl
```

### Basic Commands

#### Generate Wordlist
```bash
cewl http://target.com -w wordlist.txt
```

#### With Depth
```bash
cewl -d 3 http://target.com -w wordlist.txt
```

#### Lowercase
```bash
cewl -l http://target.com -w wordlist.txt
```

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-d` | Depth | `cewl -d 3 http://target.com` |
| `-m` | Min word length | `cewl -m 5 http://target.com` |
| `-M` | Max word length | `cewl -M 10 http://target.com` |
| `-w` | Output file | `cewl -w wordlist.txt http://target.com` |
| `-l` | Lowercase | `cewl -l http://target.com` |
| `--email` | Extract emails | `cewl --email http://target.com` |
| `--meta` | Meta data | `cewl --meta http://target.com` |
| `--ua` | User-Agent | `cewl --ua "Mozilla/5.0" http://target.com` |
| `--proxy` | Proxy | `cewl --proxy http://127.0.0.1:8080 http://target.com` |
| `--threads` | Threads | `cewl --threads 10 http://target.com` |

### Usage

#### Basic Crawl
```bash
cewl http://target.com -w wordlist.txt
```

#### Deep Crawl
```bash
cewl -d 5 -m 5 http://target.com -w wordlist.txt
```

#### With Authentication
```bash
cewl --auth_type basic --auth_user admin --auth_pass pass http://target.com -w wordlist.txt
```

### Workflow

1. **Identify** target website
2. **Run** CeWL
3. **Generate** wordlist
4. **Use** with cracking tool

### Integration

#### With Hydra
```bash
# Generate
cewl http://target.com -w wordlist.txt

# Use
hydra -l admin -P wordlist.txt ssh://192.168.1.100
```

#### With Hashcat
```bash
# Generate
cewl http://target.com -w wordlist.txt

# Use
hashcat -m 0 hash.txt wordlist.txt
```

### Email Extraction

```bash
cewl --email http://target.com -w emails.txt
```

### Troubleshooting

#### Connection Refused
```bash
curl -I http://target.com
```

#### No Words Found
```bash
cewl -d 5 http://target.com -w wordlist.txt
```

### Related Tools
- **crunch:** Wordlist generator
- **wordlists:** Common wordlists
- **twofi:** Two-word frequency list
