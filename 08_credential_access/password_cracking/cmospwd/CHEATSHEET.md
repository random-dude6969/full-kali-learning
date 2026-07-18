# CMOSPwd Cheatsheet

## Quick Reference

### Installation
```bash
sudo apt install cmospwd
```

### Basic Commands

#### Recover User Password
```bash
cmospwd /user
```

#### Recover Admin Password
```bash
cmospwd /admin
```

#### Recover All Passwords
```bash
cmospwd /all
```

### Manufacturer-Specific

#### Award BIOS
```bash
cmospwd /award
```

#### Phoenix BIOS
```bash
cmospwd /phoenix
```

#### AMI BIOS
```bash
cmospwd /ami
```

#### Compaq
```bash
cmospwd /compaq
```

#### Dell
```bash
cmospwd /dell
```

#### HP
```bash
cmospwd /hp
```

### CMOS Dump Recovery

#### From Dump File
```bash
cmospwd /d dump_file.bin
```

#### Verbose Output
```bash
cmospwd /d dump_file.bin /v
```

### Output Options

#### Save to File
```bash
cmospwd /user /o results.txt
```

#### Debug Mode
```bash
cmospwd /all /debug
```

### Workflow

1. **Access** system or extract CMOS
2. **Run** appropriate command
3. **Note** recovered password
4. **Use** password to access BIOS

### Common BIOS Passwords

| Type | Default |
|------|---------|
| Award | admin, password |
| Phoenix | admin, BIOS |
| AMI | AMI, password |

### Troubleshooting

#### No BIOS Detected
```bash
# Try manufacturer-specific
cmospwd /award
cmospwd /phoenix
cmospwd /ami
```

#### Access Denied
```bash
# Run with sudo
sudo cmospwd /user
```

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **ophcrack:** Windows password cracker
