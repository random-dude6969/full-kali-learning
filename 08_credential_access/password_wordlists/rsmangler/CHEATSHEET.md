# Rsmangler Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install rsmangler
```

### Basic Commands

#### Basic Mangling
```bash
rsmangler -f wordlist.txt -o mangled.txt
```

#### With Transformations
```bash
rsmangler -f wordlist.txt --first-upper --capitalize -o mangled.txt
```

### Flags Reference

| Flag | Description |
|------|-------------|
| `-f` | Input file |
| `-o` | Output file |
| `--first-upper` | First letter uppercase |
| `--first-lower` | First letter lowercase |
| `--all-upper` | All uppercase |
| `--all-lower` | All lowercase |
| `--capitalize` | Capitalize first letter |
| `--duplicate` | Duplicate words |
| `--toggle-case` | Toggle case |
| `--add-suffix` | Add suffix |
| `--add-prefix` | Add prefix |
| `--replace` | Replace characters |

### Usage

#### Basic
```bash
rsmangler -f wordlist.txt -o mangled.txt
```

#### With Suffix
```bash
rsmangler -f wordlist.txt --add-suffix "123" -o mangled.txt
```

#### With Replacements
```bash
rsmangler -f wordlist.txt --replace "a:4,e:3" -o mangled.txt
```

### Workflow

1. **Create** base wordlist
2. **Run** rsmangler
3. **Apply** transformations
4. **Use** with cracking tool

### Integration

#### With Crunch
```bash
# Generate base
crunch 6 6 -t @@@%%% -o base.txt

# Mangle
rsmangler -f base.txt --first-upper -o mangled.txt
```

#### With Hydra
```bash
# Generate
rsmangler -f wordlist.txt --first-upper -o mangled.txt

# Use
hydra -l admin -P mangled.txt ssh://192.168.1.100
```

### Troubleshooting

#### No Input File
```bash
ls -la wordlist.txt
rsmangler -f wordlist.txt -o mangled.txt
```

### Related Tools
- **crunch:** Wordlist generator
- **cewl:** Custom wordlist from websites
- **wordlists:** Common wordlists
