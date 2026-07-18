# HackRF One — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install hackrf          # Install tools
hackrf_info                      # Verify installation
lsusb | grep "1d50:6089"        # Check USB connection
```

## Device Info
```bash
hackrf_info                      # Device information
hackrf_info -d                   # List connected devices
hackrf_info -v                   # Verbose output
```

## Receiving (RX)
```bash
# Basic receive
hackrf_transfer -r capture.raw -s 20000000 -f 433000000 -n 200000000

# With gain settings
hackrf_transfer -r capture.raw -s 20000000 -f 433000000 -l 32 -g 20 -a 1 -n 100000000

# Until interrupted
hackrf_transfer -r capture.raw -s 20000000 -f 100000000
```

## Transmitting (TX)
```bash
# Basic transmit
hackrf_transfer -t signal.raw -s 20000000 -f 433000000 -x 30

# Max power (USE DUMMY LOAD!)
hackrf_transfer -t signal.raw -s 20000000 -f 433000000 -x 62 -a 1
```

## Spectrum Sweep
```bash
hackrf_sweep > sweep.csv                                    # Full sweep
hackrf_sweep -f 100:6000000000 -w 1000000 -r sweep.csv      # Specific range
hackrf_sweep -f 430:440000000 -w 100000 -l 32 -g 20        # ISM band
```

## Key Parameters
| Param | Description | Range |
|-------|-------------|-------|
| `-s rate` | Sample rate | 2M - 22M samples/s |
| `-f freq` | Center frequency | 1 MHz - 6 GHz |
| `-l gain` | LNA gain | 0-40 dB (8 dB steps) |
| `-g gain` | VGA gain | 0-62 dB (2 dB steps) |
| `-x gain` | TX VGA gain | 0-62 dB (2 dB steps) |
| `-a flag` | RF amplifier | 0=off, 1=on (+14 dB) |
| `-n count` | Sample count | 0 = unlimited |

## Gain Guide
```bash
# RX Total Gain = LNA + VGA + (Amp ? 14 : 0)
# Strong signal: -l 0 -g 0 -a 0
# Weak signal:   -l 32 -g 40 -a 1
# General:       -l 32 -g 20 -a 1

# TX: START LOW! (-x 20), never exceed -x 40 without dummy load
```

## Data Format
- 8-bit signed I/Q samples (2 bytes per sample)
- Range: -128 to +127
- Interleaved: I1 Q1 I2 Q2 I3 Q3 ...
- File size = samples × 2 bytes

## Quick Frequency Reference
| Freq | Use |
|------|-----|
| 315 MHz | Key fobs (Americas) |
| 433 MHz | ISM band, key fobs (Europe) |
| 868 MHz | LoRa/Sigfox (Europe) |
| 915 MHz | LoRa (Americas) |
| 1090 MHz | ADS-B aircraft |
| 2.4 GHz | WiFi, Bluetooth |
| 5 GHz | WiFi 802.11a/n/ac |

## ISM Band Scanner
```bash
#!/bin/bash
for freq in 315000000 433000000 868000000 915000000; do
    echo "Capturing at $((freq/1000000)) MHz..."
    hackrf_transfer -r "capture_${freq}.raw" -s 20000000 -f $freq -l 32 -g 20 -n 10000000
done
```

## Firmware
```bash
hackrf_info | grep Firmware                           # Check version
hackrf_spiflash -w hackrf_one_usb.bin                # Update firmware
# Enter DFU: Unplug, hold DFU button, plug, release after 2s
```

## Clock
```bash
hackrf_clock -o 1    # External 10 MHz reference
hackrf_clock -o 0    # Internal oscillator
hackrf_clock -r      # Check status
```

## Device State
```bash
hackrf_operaclean    # Reset device (if unresponsive)
```

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No device found | Check udev rules, `sudo usermod -aG plugdev $USER` |
| Permission denied | `sudo hackrf_info` or fix udev rules |
| Transfer stalled | Reduce sample rate, try USB 2.0 port |
| Firmware mismatch | `hackrf_spiflash -w firmware.bin` |
| Device not responding | `hackrf_operaclean`, kill competing processes |

## udev Rules
```bash
# /etc/udev/rules.d/53-hackrf.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6089", MODE="0666", GROUP="plugdev"
```

## Related Tools
| Tool | Purpose |
|------|---------|
| `gqrx` | SDR receiver GUI |
| `gnuradio` | Visual signal processing |
| `inspectrum` | Signal visualization |
| `urh` | Protocol reverse engineering |
| `rfcat` | Sub-GHz protocol analysis |

## Legal
- Receiving publicly broadcast signals is generally legal
- Transmitting requires authorization in most jurisdictions
- HackRF is NOT FCC/CE certified for transmission
- Use minimum power necessary for testing
