# PIPAL - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Basic analysis
pipal passwords.txt

# Show top 5
pipal -t 5 passwords.txt

# Save to file
pipal -o results.txt passwords.txt
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Version | 3.4.0 |
| Language | Ruby |
| Default Top | 10 |
| Input Format | One password per line |
| Output | Console/File |

---

## Command-Line Options

| Option | Description |
|--------|-------------|
| `FILENAME` | Password file to analyze |
| `--top, -t N` | Show top N results (default: 10) |
| `--output, -o FILE` | Save results to file |
| `--verbose, -v` | Verbose output |
| `--gkey KEY` | Google Maps API for zip codes |
| `--list-checkers` | Show available checkers |
| `--help, -h` | Show help |

---

## Output Sections

### Top Passwords
```
Top 10 passwords
password = 1234 (24.2%)
123456 = 892 (17.5%)
qwerty = 456 (9.0%)
```

### Base Words
```
Top 5 base words
love = 156 (3.1%)
password = 134 (2.6%)
angel = 89 (1.7%)
```

### Length Distribution
```
Password length (length ordered)
6 = 1234 (24.2%)
7 = 892 (17.5%)
8 = 567 (11.1%)
```

### Character Composition
```
Lowercase letters only = 2345 (46.1%)
Lowercase + numbers = 1567 (30.8%)
Mixed case = 892 (17.5%)
```

---

## Common Use Cases

```bash
# Analyze leaked passwords
pipal leaked_passwords.txt -t 20

# Password policy assessment
pipal org_passwords.txt -t 100

# Create report for management
pipal passwords.txt -o password_analysis_report.txt

# Analyze for cracking optimization
pipal wordlist.txt -t 50
```

---

## Analysis Workflow

```bash
# 1. Prepare password file
cat passwords.txt | sort | uniq > cleaned.txt

# 2. Run analysis
pipal cleaned.txt -t 20 -o results.txt

# 3. Review results
cat results.txt

# 4. Identify patterns
# - Most common passwords
# - Average length
# - Character composition
# - Base words
```

---

## Key Metrics to Track

| Metric | Description |
|--------|-------------|
| Total Entries | Number of passwords analyzed |
| Unique Entries | Distinct passwords |
| Top Passwords | Most commonly used |
| Base Words | Root words before modifications |
| Average Length | Mean password length |
| Length Distribution | Spread across lengths |
| Character Types | Letters, numbers, symbols |

---

## Integration Examples

```bash
# With Hashcat
pipal leaked_passwords.txt -t 100
# Use results to create targeted wordlist
hashcat -m 0 hashes.txt wordlist.txt

# With John the Ripper
pipal org_passwords.txt -o patterns.txt
john --wordlist=wordlist.txt hashes.txt

# With Crunch
pipal passwords.txt -t 50
# Use length distribution to inform patterns
crunch 8 8 1234567890 -o wordlist.txt
```

---

## Password Policy Recommendations

Based on PIPAL analysis:

| Finding | Recommendation |
|---------|----------------|
| Short passwords common | Minimum 12 characters |
| No symbols used | Require special characters |
| Common base words | Block top 1000 passwords |
| Predictable patterns | Enforce complexity |
| Keyboard patterns | Block sequential characters |

---

## Troubleshooting

```bash
# Check Ruby version
ruby --version

# Install dependencies
gem install json levenshtein

# Check file format
head -5 passwords.txt

# For large files, use sample
head -10000 passwords.txt > sample.txt
pipal sample.txt
```

---

## Legal Considerations

- Only analyze authorized password data
- Follow data protection regulations
- Document methodology
- Protect sensitive findings
- Don't store real passwords insecurely
