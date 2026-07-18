---
title: "sipp Cheatsheet"
description: Quick reference for sipp SIP protocol test tool
tool: sipp
category: voip
difficulty: advanced
---

# sipp Cheatsheet

## Quick Start

```bash
# Basic load test
sipp 192.168.1.100 -r 10 -m 100

# Custom scenario
sipp 192.168.1.100 -sf scenario.xml -r 5 -m 50

# Background mode
sipp 192.168.1.100 -bg -r 20 -m 500 -trace_stat

# UAS mode
sipp -bg -p 5061
```

## SIP Methods

| Method | Purpose |
|--------|---------|
| INVITE | Initiate call |
| REGISTER | Register endpoint |
| OPTIONS | Query capabilities |
| BYE | Terminate call |
| CANCEL | Cancel pending |
| ACK | Acknowledge |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-sf` | Scenario file | `-sf scenario.xml` |
| `-r` | Call rate (cps) | `-r 10` |
| `-m` | Max total calls | `-m 1000` |
| `-l` | Max simultaneous | `-l 100` |
| `-p` | Local port | `-p 5060` |
| `-nr` | No reconnect | `-nr` |
| `-trace_stat` | Statistics file | `-trace_stat` |
| `-trace_screen` | Screen output | `-trace_screen` |
| `-bg` | Background mode | `-bg` |
| `-f` | From URI | `-f user@domain` |
| `-t` | Transport | `-t tls` |
| `-auth` | Authentication | `-auth user:pass` |

## Examples

### Gradual Load Test
```bash
sipp 192.168.1.100 -r 1 -m 1000 -l 10
```

### High-Volume Test
```bash
sipp 192.168.1.100 -r 100 -m 10000 -l 200 -bg
```

### TLS Test
```bash
sipp 192.168.1.100 -t tls -tls_port 5061 -sf scenario.xml
```

### Registration Test
```bash
sipp 192.168.1.100 -sf register.xml -r 1 -m 10
```

## Chaining

```bash
# Start UAS, then UAC
sipp -bg -p 5061 -trace_stat
sipp 127.0.0.1:5061 -r 5 -m 50

# Multiple rate tests
for rate in 1 5 10 20 50; do
    sipp 192.168.1.100 -r $rate -m 1000 -trace_stat -stat_file "stats_${rate}cps.csv"
    sleep 10
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Connection refused | Verify server is running |
| Timeout | Check network connectivity |
| Auth failures | Add `-auth user:pass` |
| High latency | Reduce `-r` rate |
| Memory issues | Limit `-l` concurrent calls |

```bash
# Verify scenario
sipp -check_sc scenario.xml

# Debug mode
sipp 192.168.1.100 -v -v -sf scenario.xml

# Capture SIP traffic
sudo tcpdump -i eth0 "udp port 5060" -w debug.pcap
```
