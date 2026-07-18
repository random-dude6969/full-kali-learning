# CHIRP — Quick Reference Cheatsheet

## Installation
```bash
sudo apt install chirp           # Install
chirp --version                  # Verify
chirp                            # Launch GUI
```

## CLI Commands
```bash
# Read radio to image
chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r -o backup.img

# Write image to radio
chirp --clone-write -p /dev/ttyUSB0 -r baofeng_uv5r backup.img

# Export to CSV
chirp --export -i backup.img -o channels.csv

# Import from CSV
chirp --import -i channels.csv -o updated.img

# List supported radios
chirp --list

# View image info
chirp --info -i radio.img
```

## GUI Workflow
1. Connect programming cable
2. Launch CHIRP
3. Radio → Download From Radio
4. Select port, vendor, model
5. Edit channels in tabbed interface
6. Radio → Upload To Radio

## Serial Port Setup
```bash
ls /dev/ttyUSB*                  # Find port
sudo usermod -a -G dialout $USER # Add to dialout group
ls -la /dev/ttyUSB0              # Check permissions
```

## Channel Programming
| Field | Description | Example |
|-------|-------------|---------|
| Name | Channel name (max 16 chars) | "LOCAL RPT" |
| Frequency | Frequency in MHz | 446.000000 |
| Duplex | +, -, split, or none | + |
| Offset | Offset in MHz | 0.600 |
| Tone | TSQL, DTCS, or None | TSQL |
| Tone Freq | CTCSS tone | 67.0 |
| Power | Low, Medium, High | Medium |
| Skip | S for skip, blank for scan | |

## Common Frequencies

### 2m Band
| Use | Freq (MHz) | Tone |
|-----|------------|------|
| Calling | 146.520 | None |
| Repeater | 146.940 | 100.0 |
| ARES | 145.490 | 100.0 |

### 70cm Band
| Use | Freq (MHz) | Tone |
|-----|------------|------|
| Calling | 446.000 | None |
| Repeater | 444.550 | 100.0 |
| FRS | 462.5625 | None |

### NOAA Weather
| Station | Freq (MHz) |
|---------|------------|
| NOAA 1 | 162.400 |
| NOAA 2 | 162.425 |
| NOAA 3 | 162.450 |
| NOAA 4 | 162.475 |
| NOAA 5 | 162.500 |
| NOAA 6 | 162.525 |
| NOAA 7 | 162.550 |

## CSV Format
```csv
Name,Frequency,Duplex,Offset,Tone,Tone Frequency,DCS Code,DCS Polarity,DTCS Code,DTCS Polarity,Power,Comment,Cross-Tone,Cross-DCS
"Repeater",446.000000,+,0.600,TSQL,67.0,023,N,023,N,Low,,,
```

## Supported Manufacturers
| Manufacturer | Popular Models |
|-------------|----------------|
| Baofeng | UV-5R, UV-82, BF-F8HP |
| Kenwood | TH-D74A, TM-D700 |
| Yaesu | FT-60, FT-817, FT-891 |
| Icom | IC-2100, IC-7000, IC-7100 |
| WouXun | KG-UV6P, KG-UV8D |
| TYT | TH-9000D, TH-UV3R |

## Programming Cable Types
| Cable | Radio Brands |
|-------|-------------|
| K-type (2-pin) | Baofeng, WouXun, TYT, Kenwood |
| Yaesu SCU-35 | Newer Yaesu models |
| Icom OPC-478 | Icom CI-V interface |

## Backup Script
```bash
#!/bin/bash
BACKUP_DIR="/backup/radios/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

chirp --clone-read -p /dev/ttyUSB0 -r baofeng_uv5r \
    "$BACKUP_DIR/uv5r_$(date +%H%M%S).img"
echo "Backup complete: $BACKUP_DIR"
```

## Batch Programming
```bash
#!/bin/bash
IMAGE="master_config.img"
for i in 1 2 3; do
    echo "Programming radio $i..."
    chirp --clone-write -p /dev/ttyUSB0 -r baofeng_uv5r "$IMAGE"
    read -p "Swap radio and press Enter..."
done
```

## Troubleshooting
| Problem | Solution |
|---------|----------|
| No port found | Check USB, `ls /dev/ttyUSB*` |
| Permission denied | `sudo usermod -a -G dialout $USER` |
| Radio not responding | Check cable, power, correct model |
| Image corrupted | Re-read from radio |
| Import errors | Check CSV format, encoding |

## FAQ
**Q: Is CHIRP legal?**
A: Yes, for programming radios you own. Programming to unauthorized frequencies is illegal.

**Q: Can I damage my radio?**
A: Unlikely. CHIRP writes memory data, not firmware.

**Q: Which cable do I need?**
A: Most Baofeng/WouXun use K-type (2-pin). Check your radio docs.

## Related Tools
| Tool | Purpose |
|------|---------|
| `gqrx` | SDR receiver GUI |
| `hackrf` | SDR transceiver |
| `uv3r-programmer` | UV-3R specific tool |

## Resources
- **Website**: https://chirpmyradio.com
- **GitHub**: https://github.com/kk7ds/chirp
- **Supported Radios**: https://chirpmyradio.com/wiki/SupportedRadios
