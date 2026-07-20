---
title: "minicom - Serial Communication Program"
description: "Terminal program for serial console communication with network devices, embedded systems, and hardware"
category: "serial_console"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "minicom"
kali_install: "sudo apt install minicom"
version: "2.11.1"
---

# minicom

## Overview

minicom is a menu-driven serial communication program for Linux. It emulates ANSI and VT102 terminals and provides capabilities for connecting to serial ports, modems, and console devices. minicom is essential for accessing the console interfaces of routers, switches, firewalls, embedded systems, and other hardware that provides serial console access.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install minicom
```

### Verify Installation

```bash
minicom --version
```

## Chapter 2: Basic Usage

### Quick Connection

```bash
sudo minicom -D /dev/ttyUSB0 -b 115200
```

**Explanation**: `-D` specifies the serial device, `-b` sets the baud rate. This connects to a serial device at 115200 bps, which is the most common default for modern devices.

### Connection with Options

```bash
sudo minicom -s
```

**Explanation**: Enters the setup/configuration menu without connecting. This allows configuring serial port parameters, modem options, and file transfer settings before connecting.

### List Available Ports

```bash
ls /dev/ttyUSB* /dev/ttyS*
```

**Explanation**: Shows available serial ports on the system. USB-to-serial adapters typically appear as `/dev/ttyUSB*`, while built-in serial ports appear as `/dev/ttyS*`.

## Chapter 3: Configuration

### Setup Menu

```bash
sudo minicom -s
```

**Explanation**: Opens the configuration menu with options for:
- **File transfer protocols**: Configure XMODEM, ZMODEM, YMODEM
- **Serial port setup**: Set device, baud rate, data bits, stop bits, parity
- **Modem and dialing**: Configure modem initialization strings
- **Screen and keyboard**: Set terminal emulation and key mappings

### Configure Serial Port

```
File transfer protocols > Serial port setup
```

**Explanation**: Configure the serial connection parameters:
- **Serial Device**: `/dev/ttyUSB0` or `/dev/ttyS0`
- **Baud Rate**: 9600, 19200, 38400, 57600, 115200
- **Data Bits**: 7 or 8
- **Parity**: None, Even, Odd
- **Stop Bits**: 1 or 2
- **Hardware Flow Control**: Yes or No

### Save Configuration

```
Save setup as dfl
```

**Explanation**: Saves the current configuration as the default for future minicom sessions. The configuration is stored in `/etc/minicom/minirc.dfl`.

## Chapter 4: Connection Profiles

### Cisco Router

```bash
sudo minicom -D /dev/ttyUSB0 -b 9600
```

**Explanation**: Cisco devices typically use 9600 baud, 8 data bits, no parity, 1 stop bit (9600 8N1). The serial connection provides console access for configuration and management.

### Juniper Device

```bash
sudo minicom -D /dev/ttyUSB0 -b 9600
```

**Explanation**: Juniper devices also use 9600 8N1 by default. Console access provides device configuration, monitoring, and recovery capabilities.

### Embedded Linux

```bash
sudo minicom -D /dev/ttyUSB0 -b 115200
```

**Explanation**: Most embedded Linux devices use 115200 baud for serial console. This provides access to the boot loader (U-Boot) and Linux console.

## Chapter 5: File Transfer

### ZMODEM Transfer

```
Ctrl+A > Z > Select file
```

**Explanation**: ZMODEM is the preferred file transfer protocol for serial connections. It provides error checking, resume capability, and automatic transfer initiation. Send files from the host to the device or vice versa.

### XMODEM Transfer

```
Ctrl+A > S > Select XMODEM > Choose file
```

**Explanation**: XMODEM is an older protocol with basic error checking. Use when ZMODEM is not supported by the target device. XMODEM has lower throughput than ZMODEM.

### YMODEM Transfer

```
Ctrl+A > S > Select YMODEM > Choose file
```

**Explanation**: YMODEM improves on XMODEM with 1KB block sizes and batch file transfer capability. It is supported by many embedded systems and boot loaders.

## Chapter 6: Terminal Functions

### Capture Session

```
Ctrl+A > L > Enter filename
```

**Explanation**: Captures all terminal output to a file. This creates a log of the serial session for documentation, analysis, or debugging purposes.

### Reset Terminal

```
Ctrl+A > X > Yes
```

**Explanation**: Resets the terminal emulation. Useful when the display becomes corrupted or characters are not displaying correctly.

### Toggle Local Echo

```
Ctrl+A > E
```

**Explanation**: Toggles local echo on/off. When enabled, typed characters are displayed locally. Disable when the remote device echoes characters back to prevent double display.

## Chapter 7: Command Line Options

### Baud Rate Override

```bash
sudo minicom -D /dev/ttyUSB0 -b 115200 -o
```

**Explanation**: `-b` sets the baud rate, `-o` prevents modem initialization at startup. This is useful for connecting to devices that do not use standard modem initialization.

### Custom Configuration

```bash
sudo minicom -C /path/to/logfile -D /dev/ttyUSB0
```

**Explanation**: `-C` specifies a capture file for logging the session. All serial communication is logged to the specified file.

### Non-Interactive Mode

```bash
sudo minicom -D /dev/ttyUSB0 -b 115200 -H
```

**Explanation**: `-H` enables hex output mode, displaying all received data in hexadecimal format. Useful for debugging binary protocols or identifying non-printable characters.

## Chapter 8: Advanced Techniques

### Scripted Login

```bash
sudo minicom -S /path/to/script -D /dev/ttyUSB0
```

**Explanation**: Runs a script file at startup using `runscript`. Scripts automate login sequences, device configuration, and data collection tasks.

### Pseudo-Terminal

```bash
sudo minicom -p /dev/pts/0 -D /dev/ttyUSB0
```

**Explanation**: Connects to a pseudo-terminal instead of a real serial port. This is useful for testing or when the serial device is accessed through other software.

### Script Automation

```
expect "login:"
send "admin\r"
expect "Password:"
send "password123\r"
expect "#"
send "show version\r"
log /tmp/device_output.txt
```

**Explanation**: minicom's runscript language automates device interaction. Scripts use `expect` to wait for specific strings and `send` to transmit commands.

## Chapter 9: Command Reference

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-D, --device` | Serial device | /dev/modem |
| `-b, --baudrate` | Baud rate | 9600 |
| `-o, --noinit` | Skip modem initialization | false |
| `-p, --ptty` | Pseudo terminal device | none |
| `-C, --capturefile` | Capture output to file | none |
| `-S, --script` | Run script at startup | none |
| `-H, --hexdump` | Hex output mode | disabled |
| `-R, --restrict` | Restrict scrollback | false |
| `-Z, --statline` | Status line mode | enabled |
| `-8, --8bit` | 8-bit clean mode | disabled |
| `-v, --verbose` | Verbose output | quiet |

### Keyboard Shortcuts

| Shortcut | Function |
|----------|----------|
| `Ctrl+A` then `X` | Exit minicom |
| `Ctrl+A` then `L` | Toggle capture to file |
| `Ctrl+A` then `Z` | Help menu |
| `Ctrl+A` then `E` | Toggle local echo |
| `Ctrl+A` then `R` | Upload file (ZMODEM) |
| `Ctrl+A` then `S` | Send file |
| `Ctrl+A` then `J` | Scroll down |
| `Ctrl+A` then `K` | Scroll up |
| `Ctrl+A` then `G` | Run script |
| `Ctrl+A` then `H` | Toggle halt output |
| `Ctrl+A` then `T` | Set terminal type |
| `Ctrl+A` then `W` | Toggle line wrap |
| `Ctrl+A` then `B` | Scroll backward |
| `Ctrl+A` then `F` | Toggle flow control |

### Configuration File Locations

| Path | Description |
|------|-------------|
| `/etc/minicom/minirc.dfl` | Default configuration |
| `~/.minirc.dfl` | User default configuration |
| `/etc/minicom/minirc.sys` | System configuration |
| `~/.minirc.*` | Named profile |
| `/var/lib/minicom/` | Lock files |

## Chapter 10: Real-World Scenarios

### Scenario 1: Router Password Recovery

```bash
# Connect to router console
sudo minicom -D /dev/ttyUSB0 -b 9600

# At boot, interrupt to enter ROM monitor
# Cisco: Send BREAK signal (Ctrl+Break in minicom)
# Juniper: Hold space during boot
# Aruba: Press Ctrl+B during boot

# Reset password (Cisco example)
rommon 1> confreg 0x2142
rommon 2> reset

# After boot, recover config
Router# copy startup-config running-config
Router# enable
Router# configure terminal
Router(config)# enable password newpassword
Router(config)# config-register 0x2102
Router(config)# exit
Router# copy running-config startup-config
```

**Explanation**: Serial console access provides hardware-level access for password recovery and device recovery when other access methods fail.

### Scenario 2: Embedded Linux Debugging

```bash
# Connect to embedded device
sudo minicom -D /dev/ttyUSB0 -b 115200

# During boot, interrupt U-Boot
Hit any key to stop autoboot: 3
2

# View boot parameters
U-Boot> printenv
U-Boot> setenv bootargs "console=ttyS0,115200 root=/dev/mtdblock2"

# Modify boot to enter single-user mode
U-Boot> setenv bootargs "console=ttyS0,115200 root=/dev/mtdblock2 single"
U-Boot> boot

# After boot, check system
# / # mount -o remount,rw /
# / # passwd root
# / # reboot
```

**Explanation**: Embedded Linux devices often use U-Boot bootloader with serial console access for debugging, password recovery, and firmware updates.

### Scenario 3: Network Device Configuration Backup

```bash
# Connect to device
sudo minicom -D /dev/ttyUSB0 -b 9600

# Capture configuration
Ctrl+A > L > /tmp/cisco-config.txt

# Cisco backup commands
Router# show running-config
Router# show startup-config
Router# show interfaces
Router# show ip route

# Disable capture
Ctrl+A > L
```

**Explanation**: Serial console capture provides a complete log of device configurations for backup and documentation purposes.

### Scenario 4: Modem AT Commands

```bash
# Connect to modem
sudo minicom -D /dev/ttyUSB0 -b 115200

# Basic AT commands
AT          # Test modem
ATI         # Modem information
AT+GMI      # Manufacturer
AT+GMR      # Firmware version
AT+GSN      # IMEI number
AT+CSQ      # Signal quality
AT+CREG?    # Network registration
ATD*99***1# # Dial connection
ATH         # Hang up
AT+CMEE=1   # Enable error messages
```

**Explanation**: AT commands control modems and cellular modules. minicom provides direct access for modem configuration, troubleshooting, and data connection setup.

### Scenario 5: Industrial Control Systems

```bash
# Connect to PLC/SCADA device
sudo minicom -D /dev/ttyUSB0 -b 9600

# Common industrial protocols over serial
# Modbus RTU
# Profibus
# DeviceNet
# CC-Link

# Monitor traffic
Ctrl+A > L > /tmp/industrial-capture.txt

# Device-specific commands vary by manufacturer
# Allen-Bradley, Siemens, Schneider, etc.
```

**Explanation**: Industrial control systems often use serial connections for configuration and monitoring. Serial console access provides direct device communication.

## Chapter 11: Detection and Defense

### Network Monitoring

```bash
# Monitor serial port usage
lsof /dev/ttyUSB*
lsof /dev/ttyS*

# Check for active serial connections
fuser /dev/ttyUSB0
fuser /dev/ttyS0

# Monitor minicom processes
ps aux | grep minicom
```

### Physical Security

- Restrict physical access to serial ports
- Disable unused serial ports in BIOS
- Use USB serial adapters with authorization
- Implement serial console access controls
- Monitor for unauthorized USB serial adapters

### System Configuration

```bash
# Disable unused serial ports in /etc/inittab
# S0:23:respawn:/sbin/getty -L ttyS0 9600 vt100
# Comment out unused getty lines

# Disable serial console in GRUB
# Remove: console=ttyS0,115200 from kernel parameters

# Restrict serial port permissions
chmod 600 /dev/ttyUSB*
chgrp dialout /dev/ttyUSB*
```

### Audit Logging

```bash
# Log serial port access
auditctl -w /dev/ttyUSB0 -p rwxa -k serial_access
auditctl -w /dev/ttyS0 -p rwxa -k serial_access

# Review audit logs
ausearch -k serial_access

# Monitor minicom execution
auditctl -w /usr/bin/minicom -p x -k minicom_exec
```

## Chapter 12: Troubleshooting

### Permission Denied

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Check device permissions
ls -la /dev/ttyUSB*
ls -la /dev/ttyS*

# Temporarily change permissions
sudo chmod 666 /dev/ttyUSB0

# Or run as root
sudo minicom -D /dev/ttyUSB0
```

### Device Not Found

```bash
# Check USB devices
lsusb
dmesg | grep tty

# Check available ports
ls /dev/tty*
ls /dev/serial/by-id/

# Install USB serial driver
sudo apt install brltty
# Or specific driver for your adapter
```

### Garbled Output

```bash
# Check baud rate
# Common: 9600, 19200, 38400, 57600, 115200
sudo minicom -D /dev/ttyUSB0 -b 115200

# Check data bits, parity, stop bits
# Most common: 8N1 (8 data bits, No parity, 1 stop bit)

# Toggle local echo
Ctrl+A > E

# Reset terminal
Ctrl+A > X > Yes
```

### Connection Drops

```bash
# Check cable connections
# Check USB power management
echo 'on' | sudo tee /sys/bus/usb/devices/*/power/control

# Disable USB autosuspend
echo -1 | sudo tee /sys/module/usbcore/parameters/autosuspend

# Use powered USB hub for unstable connections
```

### No Output from Device

```bash
# Verify correct port
dmesg | grep -i "tty\|serial\|usb"

# Check cable type
# RS-232: DB9/DB25 connectors
# USB: USB-to-serial adapter
# RJ-45: Console cable (check pinout)

# Test with loopback
# Connect TX to RX on connector
# Type characters, should echo back
sudo minicom -D /dev/ttyUSB0 -b 9600
```

## Chapter 13: Comparison with Alternatives

### minicom vs screen

| Feature | minicom | screen |
|---------|---------|--------|
| Serial support | Full | Basic |
| File transfer | ZMODEM/XMODEM/YMODEM | No |
| Configuration | Menu-based | Command line |
| Session capture | Built-in | Script redirection |
| Scripting | runscript | No |
| Terminal emulation | ANSI/VT102 | Limited |

### minicom vs picocom

| Feature | minicom | picocom |
|---------|---------|---------|
| Size | Large | Small |
| Configuration | Complex | Simple |
| File transfer | Yes | No |
| Speed | Slower startup | Fast startup |
| Dependencies | More | Fewer |

### minicom vs cu

| Feature | minicom | cu |
|---------|---------|-----|
| File transfer | ZMODEM | No |
| Configuration | Interactive | Command line |
| Platform | Linux only | Unix |
| Scripting | Yes | Limited |
| Terminal emulation | Full | Basic |

## Resources

- [minicom Homepage](https://salsa.debian.org/minicom-team/minicom)
- [minicom man page](https://linux.die.net/man/1/minicom)
- [Kali Linux minicom Page](https://www.kali.org/tools/minicom/)
- [Serial Console Guide](https://www.kernel.org/doc/Documentation/serial-console.txt)
- [MITRE ATT&CK Physical Access](https://attack.mitre.org/tactics/TA0009/)
- [RS-232 Standard](https://en.wikipedia.org/wiki/RS-232)
