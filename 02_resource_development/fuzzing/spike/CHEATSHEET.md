# SPIKE Fuzzer Cheatsheet

## Quick Start

```bash
generic_send_tcp TARGET PORT /usr/share/spike/http.spk 0 0
generic_send_udp TARGET 53 /usr/share/spike/dns.spk 0 0
```

## Commands

| Command | Description |
|---------|-------------|
| `generic_send_tcp` | Send SPIKE over TCP |
| `generic_send_udp` | Send SPIKE over UDP |
| `generic_send_telnet` | Telnet-aware send |
| `generic_chunked_send_tcp` | Chunked TCP send |
| `spikeL compile` | Compile SPIKE script |

## Arguments

```bash
generic_send_tcp HOST PORT SPIKE_FILE VARIABLE LIMIT
```

| Arg | Description |
|-----|-------------|
| HOST | Target IP |
| PORT | Target port |
| SPIKE_FILE | .spk script path |
| VARIABLE | Which variable to fuzz (0=first) |
| LIMIT | Max fuzz level (0=unlimited) |

## Sample SPIKE Script

```c
// my_fuzz.spike
s_init_fuzzing();
s_read(100);

s_string("GET ");
s_string_variable("/");
s_string(" HTTP/1.1\r\n");
s_string("Host: ");
s_string_variable("target");
s_string("\r\n\r\n");

s_xmit();
s_read(100);
```

## Compile and Run

```bash
spikeL compile my_fuzz.spike
generic_send_tcp 192.168.1.50 8080 my_fuzz 0 0
```

## Key Functions

| Function | Description |
|----------|-------------|
| `s_string("...")` | Regular string (not fuzzed) |
| `s_string_variable("...")` | Fuzzed string |
| `s_binary("\x00")` | Binary data |
| `s_binary_variable(N)` | Fuzzed binary (N bytes) |
| `s_xmit()` | Send buffer |
| `s_read(N)` | Read N bytes |
| `s_init_fuzzing()` | Initialize |

## Common SPIKE Files

```
/usr/share/spike/webserver.spk    # HTTP
/usr/share/spike/http.spk         # HTTP
/usr/share/spike/smb.spk          # SMB
/usr/share/spike/dns.spk          # DNS
/usr/share/spike/ftp.spk          # FTP
```

## Tips

- Start with pre-built SPIKE files
- Add `s_read()` between commands
- Use block size limits for stability
- Check for crashes with `nc -zv TARGET PORT`
- Consider boofuzz for new projects
