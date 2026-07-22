# PIPAL - Password Analysis Tool

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

PIPAL (Password Inclusion Analysis) is a password analysis tool designed to analyze password dumps and provide statistical information about passwords. It helps security professionals understand password patterns, identify common passwords, and assess the strength of password policies across organizations.

### 1.1 What is PIPAL?

PIPAL is a Ruby-based tool that:
- Analyzes password dumps for statistical patterns
- Identifies common passwords and base words
- Calculates password length distributions
- Detects character composition patterns
- Provides insights for password policy development

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Author | Robin Wood (Digininja) |
| License | BSD |
| Language | Ruby |
| Version | 3.4.0 |
| Homepage | digininja.org/projects/pipal.php |
| GitHub | digininja/pipal |

### 1.3 Why Use PIPAL?

- **Password Policy Development:** Understand what passwords users actually choose
- **Security Assessment:** Evaluate password strength in organizations
- **Cracking Optimization:** Prioritize password lists based on patterns
- **Compliance:** Verify password policy compliance
- **Research:** Study password usage patterns
- **Education:** Demonstrate password weaknesses to stakeholders

### 1.4 PIPAL Features

| Feature | Description |
|---------|-------------|
| Statistical Analysis | Count, frequency, distribution |
| Base Word Detection | Identify root words |
| Length Analysis | Password length statistics |
| Character Composition | Letters, numbers, symbols |
| Top Lists | Most common passwords |
| Custom Checkers | Extensible analysis |
| Output Formats | Console, file, CSV |

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install pipal

# Verify installation
pipal --version
```

### 2.2 From Source

```bash
# Clone repository
git clone https://github.com/digininja/pipal.git
cd pipal

# Install dependencies
gem install json
gem install levenshtein

# Make executable
chmod +x pipal.rb
```

### 2.3 Ruby Dependencies

```bash
# Install required gems
gem install json
gem install levenshtein
gem install uri
```

---

## 3. Basic Usage

### 3.1 Command Syntax

```bash
pipal [OPTIONS] FILENAME
```

### 3.2 Simple Analysis

```bash
# Analyze a password file
pipal passwords.txt

# Show top 5 results
pipal -t 5 passwords.txt

# Output to file
pipal -o results.txt passwords.txt
```

---

## 4. Command-Line Options

### 4.1 Analysis Options

| Option | Description |
|--------|-------------|
| `FILENAME` | File containing passwords (one per line) |
| `--top, -t N` | Show top N results (default: 10) |
| `--output, -o FILE` | Output results to file |
| `--verbose, -v` | Verbose output |

### 4.2 Advanced Options

| Option | Description |
|--------|-------------|
| `--gkey KEY` | Google Maps API key for zip code lookups |
| `--list-checkers` | Show available checkers |
| `--help, -h` | Show help message |

---

## 5. Analysis Features

### 5.1 Password Count Statistics

```bash
# Basic analysis
pipal passwords.txt

# Output includes:
# Total entries = 5085
# Total unique entries = 5076
```

### 5.2 Top Passwords

```bash
# Show top passwords
pipal -t 10 passwords.txt

# Output:
# Top 10 passwords
# password = 1234 (24.2%)
# 123456 = 892 (17.5%)
# qwerty = 456 (9.0%)
```

### 5.3 Base Word Analysis

PIPAL identifies the root word in passwords:

```bash
# Example output:
# Top 5 base words
# love = 156 (3.1%)
# password = 134 (2.6%)
# angel = 89 (1.7%)
# princess = 78 (1.5%)
# soccer = 67 (1.3%)
```

### 5.4 Length Distribution

```bash
# Password length statistics
# Password length (length ordered)
# 6 = 1234 (24.2%)
# 7 = 892 (17.5%)
# 8 = 567 (11.1%)
# 9 = 234 (4.6%)
# 10 = 123 (2.4%)
```

### 5.5 Character Composition

```bash
# Character type analysis
# Lowercase letters only = 2345 (46.1%)
# Lowercase + numbers = 1567 (30.8%)
# Mixed case = 892 (17.5%)
# With symbols = 234 (4.6%)
```

---

## 6. Practical Use Cases

### 6.1 Analyzing Leaked Password Dumps

```bash
# Download a password dump (for authorized testing only)
wget http://example.com/passwords.txt

# Analyze the dump
pipal passwords.txt -t 20

# Save results
pipal passwords.txt -o analysis_report.txt
```

### 6.2 Password Policy Assessment

```bash
# Analyze organization passwords
pipal org_passwords.txt -t 100

# Look for patterns:
# - Common base words
# - Length distribution
# - Character composition
# - Keyboard patterns
```

### 6.3 Cracking Optimization

```bash
# Analyze password patterns
pipal leaked_passwords.txt -t 50

# Use results to create targeted wordlists
# Focus on most common base words
```

### 6.4 Compliance Verification

```bash
# Check if passwords meet policy
pipal org_passwords.txt

# Look for:
# - Minimum length compliance
# - Character requirements
# - Common password usage
```

---

## 7. Understanding the Output

### 7.1 Sample Output

```
Total entries = 5085
Total unique entries = 5076

Top 5 passwords
password = 1234 (24.2%)
123456 = 892 (17.5%)
qwerty = 456 (9.0%)
abc123 = 234 (4.6%)
monkey = 123 (2.4%)

Top 5 base words
love = 156 (3.1%)
password = 134 (2.6%)
angel = 89 (1.7%)
princess = 78 (1.5%)
soccer = 67 (1.3%)

Password length (length ordered)
3 = 1 (0.02%)
4 = 11 (0.2%)
5 = 434 (8.5%)
6 = 1863 (36.6%)
7 = 1219 (24.0%)
8 = 865 (17.0%)
9 = 387 (7.6%)
10 = 156 (3.1%)
```

### 7.2 Interpreting Results

- **High percentage common passwords:** Weak password policy
- **Short length distribution:** Insufficient minimum length
- **Low symbol usage:** No complexity requirements
- **Common base words:** Predictable patterns

---

## 8. Custom Checkers

### 8.1 Available Checkers

```bash
# List available checkers
pipal --list-checkers
```

### 8.2 Creating Custom Checkers

```ruby
# Custom checker example
class MyChecker
  def initialize
    @name = "My Custom Checker"
  end
  
  def check(password)
    # Return analysis result
    return "Custom analysis: #{password.length}"
  end
end
```

---

## 9. Integration with Other Tools

### 9.1 With Hashcat

```bash
# Analyze leaked passwords
pipal leaked_passwords.txt -t 100

# Create targeted wordlist from results
# Use with hashcat for efficient cracking
hashcat -m 0 hashes.txt wordlist.txt
```

### 9.2 With John the Ripper

```bash
# Analyze password patterns
pipal org_passwords.txt -o patterns.txt

# Use patterns with John
john --wordlist=wordlist.txt hashes.txt
```

### 9.3 With Crunch

```bash
# Use PIPAL results to inform Crunch patterns
# If PIPAL shows most passwords are 8 chars with numbers
crunch 8 8 1234567890 -o wordlist.txt
```

---

## 10. Reporting

### 10.1 Console Output

```bash
# Standard output
pipal passwords.txt

# Verbose output
pipal -v passwords.txt
```

### 10.2 File Output

```bash
# Save to file
pipal passwords.txt -o report.txt

# View results
cat report.txt
```

### 10.3 Report Interpretation

Key metrics to look for:
- **Total passwords analyzed**
- **Unique password count**
- **Most common passwords**
- **Average password length**
- **Character composition**

---

## 11. Best Practices

### 11.1 Legal Considerations

- Only analyze passwords you have authorization to access
- Don't use real passwords in demonstrations
- Follow data protection regulations
- Document your methodology

### 11.2 Data Preparation

- Clean password files (remove empty lines)
- Normalize line endings
- Remove duplicates if needed
- Handle encoding properly

### 11.3 Analysis Workflow

1. Prepare password file
2. Run PIPAL analysis
3. Review top passwords
4. Analyze patterns
5. Document findings
6. Develop recommendations

---

## 12. Troubleshooting

### Common Issues

**Problem:** Ruby errors

```bash
# Check Ruby version
ruby --version

# Install required gems
gem install json levenshtein
```

**Problem:** Empty output

```bash
# Check file format
head -5 passwords.txt

# Ensure one password per line
cat passwords.txt | head -20
```

**Problem:** Slow performance

```bash
# Use smaller files for testing
head -10000 passwords.txt > sample.txt
pipal sample.txt

# For large files, be patient
```

---

## 13. Advanced Analysis Techniques

### 13.1 Pattern Detection

PIPAL can help identify common password patterns:

- **Keyboard Patterns:** qwerty, asdfgh, zxcvbn
- **Date Patterns:** Birthdays, years (1990, 2000)
- **Name Patterns:** First names, last names
- **Word Patterns:** Common words with modifications

### 13.2 Entropy Analysis

```bash
# Analyze password entropy
pipal passwords.txt -v

# Look for:
# - Low entropy passwords (easy to crack)
# - High entropy passwords (strong)
# - Entropy distribution
```

### 13.3 Character Set Analysis

```bash
# Analyze character composition
pipal passwords.txt

# Key metrics:
# - Uppercase/lowercase ratio
# - Number usage percentage
# - Symbol usage percentage
# - Character diversity
```

### 13.4 Length Analysis

```bash
# Analyze length distribution
pipal passwords.txt

# Key insights:
# - Most common lengths
# - Minimum length compliance
# - Length vs strength correlation
```

---

## 14. Creating Targeted Wordlists

### 14.1 Using PIPAL Results

```bash
# 1. Analyze leaked passwords
pipal leaked_passwords.txt -t 100

# 2. Identify most common base words
# Example output:
# love = 156 (3.1%)
# password = 134 (2.6%)
# angel = 89 (1.7%)

# 3. Create variations
# Add numbers, symbols, years
```

### 14.2 Wordlist Generation

```bash
# Create wordlist from base words
echo -e "love\npassword\nangel\nprincess\nsoccer" > base_words.txt

# Add variations with crunch
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Or use mentalist
mentalist -g base_words.txt -o wordlist.txt
```

### 14.3 Hybrid Attack Wordlists

```bash
# Combine base words with patterns
# Common patterns:
# - Word + 123
# - Word + year (1990-2024)
# - Word + !@#
# - Word + uppercase first letter

# Generate variations
for word in $(cat base_words.txt); do
    echo "$word" >> wordlist.txt
    echo "$word" + "123" >> wordlist.txt
    echo "$word" + "!" >> wordlist.txt
    echo "${word^}" >> wordlist.txt
done
```

---

## 15. Password Policy Recommendations

### 15.1 Based on PIPAL Analysis

| Finding | Recommendation |
|---------|----------------|
| Short passwords common | Minimum 12 characters |
| No symbols used | Require special characters |
| Common base words | Block top 1000 passwords |
| Predictable patterns | Enforce complexity |
| Keyboard patterns | Block sequential characters |

### 15.2 Implementation Guidelines

```bash
# Minimum requirements
# - 12+ characters
# - Uppercase + lowercase
# - Numbers + symbols
# - No dictionary words
# - No personal information
# - No keyboard patterns
```

### 15.3 Compliance Standards

| Standard | Requirements |
|----------|--------------|
| NIST SP 800-63B | 8+ chars, no composition rules |
| PCI DSS | 7+ chars, complexity required |
| HIPAA | Strong authentication |
| SOX | Access control requirements |

---

## 16. Integration with Password Crackers

### 16.1 Hashcat Integration

```bash
# 1. Analyze leaked passwords
pipal leaked_passwords.txt -t 50

# 2. Create targeted wordlist
# Focus on most common patterns

# 3. Use with Hashcat
hashcat -m 0 hashes.txt wordlist.txt

# 4. Apply rules
hashcat -m 0 hashes.txt wordlist.txt -r rules/best64.rule
```

### 16.2 John the Ripper Integration

```bash
# 1. Analyze password patterns
pipal passwords.txt -o patterns.txt

# 2. Use with John
john --wordlist=wordlist.txt hashes.txt

# 3. Apply mask attacks
john --mask=?l?l?l?l?d?d?d?d hashes.txt
```

### 16.3 Rule Generation

```bash
# Based on PIPAL results, create custom rules
# Example: If most passwords end with numbers
# Create rule to append numbers

# Hashcat rule format
:   # Append
l   # Lowercase
u   # Uppercase
c   # Capitalize
```

---

## References

- [PIPAL GitHub](https://github.com/digininja/pipal)
- [PIPAL Documentation](https://digininja.org/projects/pipal.php)
- [Kali Linux PIPAL Package](https://www.kali.org/tools/pipal/)
- [Digininja Blog](https://digininja.org/blog/)
- [Password Security Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
