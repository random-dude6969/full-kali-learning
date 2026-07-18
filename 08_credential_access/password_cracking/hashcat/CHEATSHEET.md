# Hashcat Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install hashcat
```

### Basic Commands

#### MD5 Cracking
```bash
hashcat -m 0 hashes.txt rockyou.txt
```

#### Show Results
```bash
hashcat -m 0 hashes.txt rockyou.txt --show
```

### Hash Modes

| Mode | Hash Type | Example |
|------|-----------|---------|
| 0 | MD5 | `hashcat -m 0 hashes.txt rockyou.txt` |
| 100 | SHA1 | `hashcat -m 100 hashes.txt rockyou.txt` |
| 1400 | SHA2-256 | `hashcat -m 1400 hashes.txt rockyou.txt` |
| 1700 | SHA2-512 | `hashcat -m 1700 hashes.txt rockyou.txt` |
| 1000 | NTLM | `hashcat -m 1000 hashes.txt rockyou.txt` |
| 1800 | sha512crypt | `hashcat -m 1800 hashes.txt rockyou.txt` |
| 3200 | bcrypt | `hashcat -m 3200 hashes.txt rockyou.txt` |
| 5600 | NetNTLMv2 | `hashcat -m 5600 hashes.txt rockyou.txt` |

### Attack Modes

| Mode | Name | Example |
|------|------|---------|
| 0 | Straight | `hashcat -a 0 hashes.txt rockyou.txt` |
| 1 | Combination | `hashcat -a 1 hashes.txt word1.txt word2.txt` |
| 3 | Brute-force | `hashcat -a 3 hashes.txt ?a?a?a?a?a?a` |
| 6 | Hybrid Word+Mask | `hashcat -a 6 hashes.txt rockyou.txt ?d?d?d` |
| 7 | Hybrid Mask+Word | `hashcat -a 7 hashes.txt ?d?d?d rockyou.txt` |

### Mask Placeholders

| Placeholder | Description |
|-------------|-------------|
| `?l` | Lowercase (a-z) |
| `?u` | Uppercase (A-Z) |
| `?d` | Digits (0-9) |
| `?a` | All printable |
| `?b` | Binary |

### Common Masks

```bash
# 8 lowercase
hashcat -a 3 hashes.txt ?l?l?l?l?l?l?l?l

# 6 digits
hashcat -a 3 hashes.txt ?d?d?d?d?d?d

# 8 alphanumeric
hashcat -a 3 hashes.txt ?a?a?a?a?a?a?a?a

# Custom charset
hashcat --custom-charset=1=abc123 -a 3 hashes.txt ?1?1?1?1
```

### Rules

#### Built-in Rules
```bash
hashcat -a 0 -r rules/best64.rule hashes.txt rockyou.txt
hashcat -a 0 -r rules/d3ad0ne.rule hashes.txt rockyou.txt
```

#### Generate Rules
```bash
hashcat -a 0 -g 4 hashes.txt rockyou.txt
```

### Session Management

#### Start Session
```bash
hashcat --session=mycrack -m 1000 hashes.txt rockyou.txt
```

#### Check Status
```bash
hashcat --status=mycrack
```

#### Restore Session
```bash
hashcat --restore=mycrack
```

### Performance

#### Check GPU
```bash
hashcat -I
```

#### Optimize
```bash
hashcat -O hashes.txt rockyou.txt
```

#### Workload Profile
```bash
hashcat --workload-profile=3 hashes.txt rockyou.txt
```

### Input/Output

#### Output File
```bash
hashcat -o cracked.txt hashes.txt rockyou.txt
```

#### Show Uncracked
```bash
hashcat -m 0 hashes.txt --left
```

#### Remove Cracked
```bash
hashcat --remove hashes.txt rockyou.txt
```

### WiFi Cracking

#### WPA/WPA2
```bash
# Capture
hcxdumptool -i wlan0mon --pmkid -o capture.pcapng

# Convert
hcxpcapngtool -o hash.hc22000 capture.pcapng

# Crack
hashcat -m 22000 hash.hc22000 rockyou.txt
```

### Troubleshooting

#### No Hashes Loaded
```bash
hashcat hashes.txt --example-hashes | head -50
```

#### GPU Not Found
```bash
hashcat -I
sudo apt install nvidia-driver
```

### Related Tools
- **john:** John the Ripper
- **hcxdumptool:** WiFi handshake capture
- **hashid:** Hash identifier
