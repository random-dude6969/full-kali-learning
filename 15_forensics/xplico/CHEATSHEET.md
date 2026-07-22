# xplico Cheatsheet — Network Forensics Analysis Tool

## Quick Command

```bash
xplico [options] -m <module>
```

## Installation

```bash
sudo apt install xplico
```

## Most Common Commands

```bash
# Start web interface
sudo xplico-webui-start

# Stop web interface
sudo xplico-webui-stop

# Access web UI
# URL: http://127.0.0.1:9876
# User: xplico
# Pass: xplico
```

## Web Interface Workflow

```bash
# 1. Start service
sudo xplico-webui-start

# 2. Open browser to http://127.0.0.1:9876

# 3. Create new case

# 4. Import pcap file

# 5. Browse extracted content:
#    - Sessions
#    - HTTP (web pages, images)
#    - Email (messages, attachments)
#    - FTP (transferred files)
#    - VoIP (call recordings)
#    - DNS (queries)
#    - Files (all extracted)
```

## Command-Line Options

| Option | Description |
|--------|-------------|
| `-v` | Version |
| `-c file` | Configuration file |
| `-h` | Help |
| `-s` | Silent mode |
| `-g` | Protocol graph tree |
| `-l` | Log to screen |
| `-i prot` | Protocol info |
| `-m module` | Capture module |

## Capture Modules

| Module | Description |
|--------|-------------|
| `rltm` | Real-time capture |
| `pcapf` | Pcap file |
| `pcapd` | Pcap daemon |

## Supported Protocols

| Protocol | Content Extracted |
|----------|-------------------|
| HTTP | Web pages, images, files |
| POP3/IMAP/SMTP | Emails, attachments |
| FTP | Transferred files |
| SIP/VoIP | Voice calls |
| DNS | Domain queries |
| IRC/MSN | Chat messages |

## Command-Line Processing

```bash
# Process pcap file
xplico -m pcapf -i capture.pcap -c config.cfg

# Real-time capture
xplico -m rltm -i eth0 -c config.cfg
```

## Integration

```bash
# Capture + Analysis workflow
sudo tcpdump -i eth0 -w capture.pcap
sudo xplico-webui-start
# Import capture.pcap via web UI

# Xplico + tcpflow
xplico for extraction, tcpflow for stream reassembly

# Xplico + Wireshark
xplico for content, Wireshark for packets
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Service won't start | `sudo systemctl status xplico` |
| Can't access web UI | Check port 9876, restart service |
| No sessions decoded | Verify pcap has traffic, try different decoder |
| Database errors | Remove /tmp/xplico/db/*, restart |
