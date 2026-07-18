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

## Resources

- [minicom Homepage](https://salsa.debian.org/minicom-team/minicom)
- [minicom man page](https://linux.die.net/man/1/minicom)
- [Kali Linux minicom Page](https://www.kali.org/tools/minicom/)
- [Serial Console Guide](https://www.kernel.org/doc/Documentation/serial-console.txt)
