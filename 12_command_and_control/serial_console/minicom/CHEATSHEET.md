---
title: "minicom CHEATSHEET"
description: "Quick reference for minicom serial communication commands"
---

# minicom CHEATSHEET

## Quick Start

```bash
sudo minicom -D /dev/ttyUSB0 -b 115200        # Connect to USB serial
sudo minicom -D /dev/ttyS0 -b 9600             # Connect to COM1
sudo minicom -s                                 # Setup menu
sudo minicom -s -C /tmp/serial.log              # Setup with logging
```

## Command Line Options

| Flag | Description |
|------|-------------|
| `-D DEVICE` | Serial device path |
| `-b BAUD` | Baud rate (9600, 19200, 38400, 57600, 115200) |
| `-s` | Setup/configuration menu |
| `-o` | No modem initialization |
| `-C FILE` | Capture file for logging |
| `-S SCRIPT` | Run script at startup |
| `-p` | Pseudo-terminal |
| `-H` | Hex output mode |
| `-8` | 8-bit clean mode |
| `-c` | Color mode (via xminicom) |

## Common Serial Settings

```
# Cisco (default)
Device: /dev/ttyUSB0
Baud: 9600
Data bits: 8
Parity: None
Stop bits: 1
Flow control: Hardware

# Juniper (default)
Device: /dev/ttyUSB0
Baud: 9600
Data bits: 8
Parity: None
Stop bits: 1
Flow control: Hardware

# Embedded Linux (default)
Device: /dev/ttyUSB0
Baud: 115200
Data bits: 8
Parity: None
Stop bits: 1
Flow control: None

# Packet Soldering (Arduino, ESP32)
Device: /dev/ttyUSB0
Baud: 115200
Data bits: 8
Parity: None
Stop bits: 1
Flow control: None
```

## Minicom Keyboard Shortcuts

| Key | Description |
|-----|-------------|
| `Ctrl+A` | Command prefix |
| `Ctrl+A > Z` | Help menu |
| `Ctrl+A > X` | Exit minicom |
| `Ctrl+A > L` | Capture to file |
| `Ctrl+A > W` | Line wrap toggle |
| `Ctrl+A > E` | Local echo toggle |
| `Ctrl+A > C` | Clear screen |
| `Ctrl+A > R` | Reset terminal |
| `Ctrl+A > S` | Send file (XMODEM/YMODEM) |
| `Ctrl+A > Z > Z` | ZMODEM send/receive |
| `Ctrl+A > P` | Pulse dialing |
| `Ctrl+A > T` | Tone dialing |
| `Ctrl+A > M` | Dial directory |

## File Transfer

```bash
# ZMODEM send (from minicom)
Ctrl+A > Z > Select file

# ZMODEM receive (from minicom)
Ctrl+A > Z > Choose receive

# XMODEM send
Ctrl+A > S > XMODEM > Select file
```

## Device Discovery

```bash
ls /dev/ttyUSB*                                # USB serial adapters
ls /dev/ttyS*                                  # Built-in serial ports
dmesg | grep tty                               # Kernel serial messages
lsusb | grep -i serial                         # USB serial devices
sudo setserial -g /dev/ttyS*                   # Serial port info
```

## Permissions

```bash
sudo usermod -a -G dialout $USER               # Add user to dialout group
sudo chmod 666 /dev/ttyUSB0                    # Temp permission fix
```

## Scripting with runscript

```bash
# Create login script
cat > login.txt << 'EOF'
send "admin\r"
expect "Password:"
send "password\r"
expect "#"
log /tmp/output.txt
send "show version\r"
expect "#"
!sleep 2
send "exit\r"
EOF

# Run with minicom
sudo minicom -S login.txt -D /dev/ttyUSB0 -b 9600
```

## Quick Troubleshooting

```bash
# Device busy
sudo fuser /dev/ttyUSB0                        # Check who's using it
sudo kill -9 PID                               # Kill process

# No permission
sudo chmod 666 /dev/ttyUSB0                    # Quick fix
# Or add to dialout group permanently
sudo usermod -a -G dialout $USER

# Garbled output - check baud rate
# Common baud rates: 9600, 19200, 38400, 57600, 115200
# Most embedded: 115200
# Most routers/switches: 9600
```
