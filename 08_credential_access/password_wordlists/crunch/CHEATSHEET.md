# Crunch Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install crunch
```

### Basic Commands

#### Generate Wordlist
```bash
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```

#### Numbers Only
```bash
crunch 4 4 0123456789 -o wordlist.txt
```

### Placeholders

| Placeholder | Description |
|-------------|-------------|
| `@` | Lowercase letters |
| `,` | Uppercase letters |
| `%` | Numbers |
| `^` | Symbols |

### Flags Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Pattern | `crunch 8 8 -t @@@@%%%%` |
| `-c` | Numbers per line | `crunch 8 8 -c 1000` |
| `-d` | Duplicate limit | `crunch 8 8 -d 2` |
| `-o` | Output file | `crunch 8 8 -o wordlist.txt` |
| `-b` | Start byte | `crunch 8 8 -b 1GB` |
| `-s` | Start string | `crunch 8 8 -s aaaa0000` |
| `-p` | Permutation | `crunch 4 4 -p abc def` |
| `-u` | Unique words | `crunch 8 8 -u` |
| `-z` | Gzip | `crunch 8 8 -z output.gz` |

### Common Patterns

```bash
# 4 letters + 4 numbers
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# 2 uppercase + 4 numbers
crunch 6 6 -t %%%%,, -o wordlist.txt

# word + 2 numbers
crunch 6 6 -t @@@%%% -o wordlist.txt
```

### Usage

#### Basic Generation
```bash
crunch 8 8 -t @@@@%%%% -o wordlist.txt
```

#### With Charset File
```bash
crunch 8 8 -f charset.lst mixalpha-numeric -o wordlist.txt
```

#### Resume Generation
```bash
crunch 8 8 -t @@@@%%%% -s aaaa0000 -o wordlist.txt
```

### Workflow

1. **Determine** password pattern
2. **Generate** wordlist
3. **Use** with cracking tool

### Integration

#### With Hydra
```bash
# Generate
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Use
hydra -l admin -P wordlist.txt ssh://192.168.1.100
```

#### With Hashcat
```bash
# Generate
crunch 8 8 -t @@@@%%%% -o wordlist.txt

# Use
hashcat -m 0 hash.txt wordlist.txt
```

### Troubleshooting

#### Invalid Pattern
```bash
# Use correct placeholders
# @ = lowercase
# , = uppercase
# % = numbers
# ^ = symbols
```

### Related Tools
- **cewl:** Custom wordlist from websites
- **wordlists:** Common wordlists
- **maskprocessor:** Mask-based generation
