# RFcat — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install rfcat           # Install
rfcat --version                  # Verify
sudo rfcat -r                    # Launch interactive console
```

## Interactive Console
```bash
sudo rfcat -r                    # Launch console
sudo rfcat -r -i 0               # Specific device
```

## Basic Device Operations
```python
from rflib import *

d = RfCat()                      # Connect to device
d.ping()                         # Test connection
d.getInfo()                      # Device info
d.setModeIDLE()                  # Idle mode
d.setModeRX()                    # Receive mode
d.setModeTX()                    # Transmit mode
```

## Frequency & Modulation
```python
d.setFreq(433920000)             # Set 433.92 MHz
d.setMdmModulation(MOD_ASK_OOK)  # ASK/OOK
d.setMdmModulation(MOD_FSK)      # FSK
d.setMdmModulation(MOD_GFSK)     # GFSK
d.setMdmModulation(MOD_MSK)      # MSK
d.setMdmDRate(4800)              # Data rate: 4800 baud
d.setMdmDeviatn(0)               # Deviation: 0 Hz
d.setPA(0x25)                    # Power level
```

## Receive & Transmit
```python
# Receive packet
pkt = d.RFrecv(timeout=5000)     # 5s timeout
print(pkt.data.hex())            # Packet data as hex
print(d.getRSSI())               # Signal strength

# Transmit packet
d.setModeTX()
d.RFxmit(b'\x01\x02\x03\x04')
```

## Register Access
```python
val = d.peek(0x00)               # Read register
d.poke(0x00, 0x01)              # Write register
data = d.peekX(addr, length)    # Bulk read
d.pokeX(addr, data, length)     # Bulk write
```

## Frequency Bands
| Band | Range | Common Use |
|------|-------|------------|
| Low | 300-348 MHz | 315 MHz key fobs |
| Mid-low | 378-464 MHz | 433 MHz ISM |
| Mid-high | 779-928 MHz | 868/915 MHz IoT |

## Modulation Types
| Type | Constant | Use |
|------|----------|-----|
| ASK/OOK | `MOD_ASK_OOK` | Garage doors, key fobs |
| FSK | `MOD_FSK` | Data modes, sensors |
| GFSK | `MOD_GFSK` | Bluetooth-like |
| MSK | `MOD_MSK` | Minimum shift keying |

## Quick Receive Script
```python
from rflib import *
import time

d = RfCat()
d.setFreq(433920000)
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setMdmDeviatn(0)
d.setModeRX()

print("Listening...")
for i in range(10):
    try:
        pkt = d.RFrecv(timeout=5000)
        print(f"RSSI={d.getRSSI()} Data={pkt.data.hex()}")
    except:
        print("Timeout")
```

## Quick Transmit Script
```python
from rflib import *

d = RfCat()
d.setFreq(433920000)
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setMdmDeviatn(0)
d.setPA(0x25)
d.setModeTX()
d.RFxmit(b'\xAA\x55\x01\x02\x03')
d.setModeIDLE()
```

## Frequency Scanner
```python
from rflib import *
import time

d = RfCat()
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setModeRX()

for freq in range(300000000, 500000000, 1000000):
    d.setFreq(freq)
    time.sleep(0.05)
    rssi = d.getRSSI()
    if rssi > -50:
        print(f"{freq/1e6:.3f} MHz: RSSI={rssi}")
```

## Protocol Analysis Workflow
```python
from rflib import *

d = RfCat()
d.setFreq(433920000)
d.setMdmModulation(MOD_ASK_OOK)
d.setMdmDRate(4800)
d.setModeRX()

packets = []
for i in range(100):
    try:
        pkt = d.RFrecv(timeout=5000)
        packets.append(pkt.data)
    except:
        continue

# Analyze
lengths = [len(p) for p in packets]
print(f"Packets: {len(packets)}")
print(f"Avg length: {sum(lengths)/len(lengths):.1f} bytes")
```

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No device found | Check USB, udev rules |
| Permission denied | `sudo rfcat -r` |
| Device not responding | Re-flash firmware |
| Import error | `pip3 install rflib` |

## udev Rules
```bash
# Yard Stick One
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6048", MODE="0666", GROUP="plugdev"

# CC1111 Dongle
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="16a8", MODE="0666", GROUP="plugdev"
```

## Supported Devices
| Device | Chip | Notes |
|--------|------|-------|
| Yard Stick One | CC1111 | Primary device |
| TI CC1111 Dongle | CC1111 | Legacy |
| TI CC1111EMK | CC1111 | Evaluation kit |

## Related Tools
| Tool | Purpose |
|------|---------|
| `hackrf` | Wideband SDR transceiver |
| `rfcat` (this) | Sub-GHz protocol analysis |
| `yerdos` | Python library for Yard Stick |
| `urh` | Protocol reverse engineering |

## Legal
- Use only with authorization
- Sub-GHz ISM bands: Check local power limits
- Transmitting on licensed frequencies is illegal
