# BED Cheatsheet

## Basic Usage

```bash
bed -s FTP -t 192.168.1.50 -p 21       # Fuzz FTP
bed -s HTTP -t 192.168.1.50 -p 80      # Fuzz HTTP
bed -s SSH -t 192.168.1.50 -p 22       # Fuzz SSH
bed -s SMTP -t 192.168.1.50 -p 25      # Fuzz SMTP
bed -s POP3 -t 192.168.1.50 -p 110     # Fuzz POP3
bed -s IMAP -t 192.168.1.50 -p 143     # Fuzz IMAP
```

## Verbosity

```bash
bed -s HTTP -t TARGET -p 80 -v 0   # Minimal
bed -s HTTP -t TARGET -p 80 -v 2   # Standard
bed -s HTTP -t TARGET -p 80 -v 3   # Verbose
bed -s HTTP -t TARGET -p 80 -v 4   # Maximum (all data)
```

## Protocols

| Protocol | Default Port | Description |
|----------|-------------|-------------|
| FTP | 21 | File Transfer |
| HTTP | 80 | Web |
| SMTP | 25 | Email send |
| IMAP | 143 | Email read |
| POP3 | 110 | Email retrieve |
| SSH | 22 | Secure Shell |
| TELNET | 23 | Telnet |
| IRC | 6667 | Chat |
| NNTP | 119 | News |

## Flags

| Flag | Description |
|------|-------------|
| `-s SERVICE` | Protocol to test |
| `-t TARGET` | Target host |
| `-p PORT` | Target port |
| `-v LEVEL` | Verbosity (0-4) |
| `-d` | Debug mode |
| `-o FILE` | Output file |
| `-m NUM` | Tries per mutation (default: 3) |

## Common Workflows

```bash
# Full service scan
for svc in FTP HTTP SMTP; do
    bed -s $svc -t 192.168.1.50 -p $port -o "bed_${svc}.txt"
done

# With Nmap
nmap -sV -p 21,25,80,110,143,993 TARGET
bed -s FTP -t TARGET -p 21 -v 3
bed -s HTTP -t TARGET -p 80 -v 3

# Reliable results
bed -s HTTP -t TARGET -p 8080 -m 5
```

## What BED Tests

- Buffer overflows (oversized fields)
- Format string vulnerabilities (%s, %n)
- Integer overflows (boundary values)
- Input validation failures
- Protocol parsing errors

## Detection Indicators

- Service crashes during testing
- Unexpected error messages
- Connection drops
- Core dumps generated

## Tips

- Test one service at a time
- Use `-v 3` for actionable output
- Increase `-m` for reliable crash detection
- Check service health after testing
- Use for quick screening before AFL
