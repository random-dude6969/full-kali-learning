# sfuzz (Simple Fuzzer) Cheatsheet

## Basic Usage

```bash
sfuzz -S TARGET -p PORT -T -f /usr/share/sfuzz/sfuzz-sample/basic.http
sfuzz -S 192.168.1.50 -p 8080 -T -f basic.http
sfuzz -S 192.168.1.100 -p 53 -U -f basic.dns
```

## Modes

| Flag | Mode |
|------|------|
| `-T` | TCP (default) |
| `-U` | UDP |
| `-O` | Output to stdout only |

## Flags

| Flag | Description |
|------|-------------|
| `-S TARGET` | Remote host |
| `-p PORT` | Target port |
| `-T` | TCP mode |
| `-U` | UDP mode |
| `-O` | Stdout only |
| `-f FILE` | Config file |
| `-v` | Verbose |
| `-q` | Quiet mode |
| `-X` | Hex output |
| `-t SECS` | Socket timeout |
| `-b NUM` | Start at test # |
| `-e` | End on failure |
| `-R` | Don't close connections |
| `-L FILE` | Log file |
| `-n` | New logfile per fuzz |
| `-r` | Trim trailing newline |
| `-D X=Y` | Define symbol |
| `-l` | Literal fuzzing only |
| `-s` | Sequence fuzzing only |

## Sample Configs

```
/usr/share/sfuzz/sfuzz-sample/basic.http
/usr/share/sfuzz/sfuzz-sample/basic.ftp
/usr/share/sfuzz/sfuzz-sample/basic.dns
```

## Config File Format

```
protocol: TCP
port: 80
req_del: 200
mseq_len: 10024
literal: GET
literal: HTTP/1.1
sequence: 50
literal: Host:
symbol: $target
literal: \r\n
```

## Common Patterns

```bash
# HTTP fuzzing
sfuzz -S TARGET -p 80 -T -f /usr/share/sfuzz/sfuzz-sample/basic.http -v

# Custom protocol
cat > custom.fuzz << 'EOF'
protocol: TCP
port: 5000
req_del: 500
literal: CONNECT
sequence: 256
literal: AUTH
sequence: 1024
literal: \r\n
EOF
sfuzz -S TARGET -p 5000 -T -f custom.fuzz -L results.log

# Parallel fuzzing
sfuzz -S TARGET -p 80 -T -f http.fuzz -L http.log &
sfuzz -S TARGET -p 21 -T -f ftp.fuzz -L ftp.log &
```

## Tips

- Start with sample configs
- Use `-q` for scripting
- Use `-X` for protocol debugging
- Check logs for crash patterns
- Use `-R` for persistent connections
