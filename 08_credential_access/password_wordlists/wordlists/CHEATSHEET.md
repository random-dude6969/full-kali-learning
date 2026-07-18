# Wordlists Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install wordlists
```

### Basic Commands

#### List Wordlists
```bash
ls /usr/share/wordlists/
ls /usr/share/seclists/
```

#### Extract Rockyou
```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Common Wordlists

#### Password Lists
| Wordlist | Location | Description |
|----------|----------|-------------|
| rockyou.txt | /usr/share/wordlists/ | 14M passwords |
| common.txt | /usr/share/wordlists/dirb/ | 4K words |

#### Directory Lists
| Wordlist | Location | Description |
|----------|----------|-------------|
| common.txt | /usr/share/wordlists/dirb/ | 4K dirs |
| big.txt | /usr/share/wordlists/dirb/ | 20K dirs |

#### Subdomain Lists
| Wordlist | Location | Description |
|----------|----------|-------------|
| subdomains-top1million-5000.txt | /usr/share/seclists/ | 5K subdomains |

### Usage

#### With Hydra
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
```

#### With Gobuster
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
```

### Combining Wordlists

```bash
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt
```

### Filtering

#### By Length
```bash
awk 'length >= 8 && length <= 12' wordlist.txt > filtered.txt
```

#### By Pattern
```bash
grep -E '^[a-zA-Z0-9]{8}$' wordlist.txt > pattern.txt
```

### Workflow

1. **Select** appropriate wordlist
2. **Use** with cracking tool
3. **Filter** if needed

### Troubleshooting

#### No Such File
```bash
ls /usr/share/wordlists/
gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **seclists:** Comprehensive collection
