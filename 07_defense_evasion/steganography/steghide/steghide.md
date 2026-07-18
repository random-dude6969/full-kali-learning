---
tool_name: "steghide"
display_name: "Steghide"
mitre_tactic: "defense_evasion"
mitre_tactic_id: "TA0005"
mitre_subcategory: "steganography"
install_command: "sudo apt install steghide"
version: "0.5.1"
github_repo: "https://github.com/StefanoDeVuono/steghide"
kali_tools_page: "https://www.kali.org/tools/steghide/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["steganography", "jpeg", "bmp", "wav", "au", "encryption", "forensics"]
related_tools: ["outguess", "stegsnow", "stegosuite", "zsteg"]
---

# Steghide

## Chapter 1: Introduction & Overview

### What is Steghide?

Steghide is a steganography program that hides data in various kinds of image and audio files. It modifies the least-significant bits of pixel values and audio samples using a pseudo-random distribution algorithm, making the embedding resistant to first-order statistical tests. Steghide supports encryption of embedded data using Blowfish (via Mcrypt) and compression via zlib.

### History & Background

- **Created:** Stefan Hetzl
- **Maintained:** Steghigh organization on GitHub
- **License:** GPL-2.0
- **Language:** C++
- **Current version:** 0.5.1
- **Dependencies:** libmhash, libmcrypt, libjpeg, zlib

### When to Use This Tool

- **CTF challenges:** Hide/extract data from images and audio files
- **Covert communication:** Embed encrypted messages in innocuous files
- **Digital forensics:** Detect and extract hidden data from suspect files
- **Security testing:** Test steganography detection capabilities

### Key Concepts

- **Cover File:** Original file used to hide data (JPEG, BMP, WAV, AU)
- **Stego File:** Modified file containing hidden data
- **Passphrase:** Used for encryption key derivation and embedding location
- **LSB (Least Significant Bit):** Modified to encode hidden data
- **Pseudo-Random Distribution:** Randomly distributes hidden bits across the file

### Supported Formats

| Type | Format | Extension |
|------|--------|-----------|
| Image | JPEG | .jpg, .jpeg |
| Image | BMP | .bmp |
| Audio | WAV | .wav |
| Audio | AU | .au |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Kali Linux / Unix / Windows (Cygwin)
- **Disk Space:** ~477 KB
- **Privileges:** User-level

### Installation

```bash
sudo apt update
sudo apt install steghide
```

### Verification

```bash
steghide --version
```

### Dependencies

| Library | Purpose |
|---------|---------|
| libmhash | Hash algorithms for passphrase key derivation |
| libmcrypt | Symmetric encryption (Blowfish) |
| libjpeg | JPEG file handling |
| zlib | Data compression |

---

## Chapter 3: Basic Usage

### Embed Data

```bash
steghide embed -cf cover_image.jpg -ef secret.txt
```
**Explanation:** Embeds `secret.txt` into `cover_image.jpg`. Prompts for passphrase.

```bash
steghide embed -cf cover_image.jpg -ef secret.txt -p "MySecretPass"
```
**Explanation:** Embeds with specified passphrase (no prompt).

### Extract Data

```bash
steghide extract -sf stego_image.jpg
```
**Explanation:** Extracts hidden data from `stego_image.jpg`. Prompts for passphrase.

```bash
steghide extract -sf stego_image.jpg -p "MySecretPass"
```
**Explanation:** Extracts with specified passphrase.

### Get File Info

```bash
steghide info stego_image.jpg
```
**Explanation:** Displays information about the file (format, capacity, embedded data details).

---

## Chapter 4: Intermediate Usage

### Embedding Options

#### Specify Output File

```bash
steghide embed -cf cover.jpg -ef secret.txt -sf output.jpg -p "Pass123"
```
**Explanation:** Embeds and writes to a new file instead of overwriting the cover.

#### Force Overwrite

```bash
steghide embed -cf cover.jpg -ef secret.txt -f -p "Pass123"
```
**Explanation:** Overwrites existing output file without prompting.

#### Encryption Selection

```bash
steghide embed -cf cover.jpg -ef secret.txt -e blowfish -p "Pass123"
```
**Explanation:** Uses Blowfish encryption for the embedded data.

```bash
steghide embed -cf cover.jpg -ef secret.txt -e rijndael-128 -p "Pass123"
```
**Explanation:** Uses AES-128 encryption.

```bash
steghide encinfo
```
**Explanation:** Lists all supported encryption algorithms.

#### Compression Control

```bash
steghide embed -cf cover.jpg -ef secret.txt -z 9 -p "Pass123"
```
**Explanation:** Maximum compression (level 9) before embedding.

```bash
steghide embed -cf cover.jpg -ef secret.txt -Z -p "Pass123"
```
**Explanation:** No compression before embedding.

#### Skip Checksum

```bash
steghide embed -cf cover.jpg -ef secret.txt -K -p "Pass123"
```
**Explanation:** Skips CRC32 checksum of embedded data (slightly less robust).

#### Skip Filename Embedding

```bash
steghide embed -cf cover.jpg -ef secret.txt -N -p "Pass123"
```
**Explanation:** Does not embed the original filename.

### Extraction Options

#### Specify Output Filename

```bash
steghide extract -sf stego.jpg -xf recovered.txt -p "Pass123"
```
**Explanation:** Extracts data to a specific output filename.

#### Verbose Output

```bash
steghide extract -sf stego.jpg -v -p "Pass123"
```
**Explanation:** Displays detailed extraction information.

#### Quiet Mode

```bash
steghide extract -sf stego.jpg -q -p "Pass123"
```
**Explanation:** Suppresses information messages during extraction.

### Info Command

```bash
steghide info stego.jpg
```
**Explanation:** Shows file format, capacity, and embedded data info (without extracting).

```bash
steghide info stego.jpg -p "Pass123"
```
**Explanation:** Shows embedded data details including encryption, compression, and original filename.

---

## Chapter 5: Advanced Usage

### Batch Processing

```bash
#!/bin/bash
KEY="SharedSecret"
for img in /path/to/cover_images/*.jpg; do
    steghide embed -cf "$img" -ef secret.txt -sf "/tmp/stego_$(basename $img)" -p "$KEY" -f -q
done
```
**Explanation:** Embeds the same secret into multiple cover images.

### Automated Extraction Scan

```bash
#!/bin/bash
KEY="password"
for img in /tmp/suspect_images/*; do
    steghide extract -sf "$img" -xf "/tmp/extracted_$(basename $img)" -p "$KEY" -f -q 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Data found in: $img"
    fi
done
```
**Explanation:** Attempts extraction from multiple files, reporting successes.

### Dictionary Attack on Steghide

```bash
#!/bin/bash
for pass in $(cat wordlist.txt); do
    steghide extract -sf stego.jpg -xf /dev/null -p "$pass" -f -q 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Password found: $pass"
        break
    fi
done
```
**Explanation:** Tries passwords from a wordlist to crack steghide encryption.

### Combining with GPG

```bash
# Encrypt before embedding
gpg -c secret.txt
steghide embed -cf cover.jpg -ef secret.txt.gpg -sf stego.jpg -p "StegPass" -f

# Extract and decrypt
steghide extract -sf stego.jpg -xf recovered.gpg -p "StegPass" -f
gpg -d recovered.gpg > secret.txt
```
**Explanation:** Double encryption layer — GPG before steganographic embedding.

### WAV File Steganography

```bash
# Embed in audio
steghide embed -cf cover_song.wav -ef secret.txt -p "AudioKey" -f

# Extract from audio
steghide extract -sf stego_song.wav -p "AudioKey"
```
**Explanation:** Hides data in WAV audio files using sample modification.

### BMP File Steganography

```bash
# Embed in BMP image
steghide embed -cf cover.bmp -ef secret.txt -p "BmpKey" -f

# Extract from BMP
steghide extract -sf stego.bmp -p "BmpKey"
```
**Explanation:** Hides data in BMP images using pixel bit modification.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge

**Objective:** Extract hidden flag from image

**Steps:**
```bash
# 1. Check file info
steghide info challenge.jpg

# 2. Try empty passphrase
steghide extract -sf challenge.jpg -xf output.txt -p "" -f

# 3. Try common passwords
steghide extract -sf challenge.jpg -xf output.txt -p "password" -f
steghide extract -sf challenge.jpg -xf output.txt -p "flag" -f
steghide extract -sf challenge.jpg -xf output.txt -p "stego" -f

# 4. Dictionary attack
for pass in $(cat /usr/share/wordlists/rockyou.txt | head -1000); do
    steghide extract -sf challenge.jpg -xf /dev/null -p "$pass" -f -q 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[+] Found: $pass"
        steghide extract -sf challenge.jpg -xf flag.txt -p "$pass" -f
        break
    fi
done

# 5. Check output
cat flag.txt
```

### Scenario 2: Covert Communication

**Objective:** Send encrypted message hidden in image

**Steps:**
```bash
# 1. Prepare message
echo "Exfil data: database credentials dbadmin:P@ssw0rd123" > intel.txt

# 2. Embed with strong passphrase
steghide embed -cf vacation_photo.jpg -ef intel.txt -sf shared_photo.jpg -p "OnlyWeKnowThis2024" -f

# 3. Send shared_photo.jpg via normal channels

# 4. Recipient extracts
steghide extract -sf shared_photo.jpg -p "OnlyWeKnowThis2024" -xf intel.txt
```

### Scenario 3: Digital Forensics Investigation

**Objective:** Detect and extract hidden data from suspect image

**Steps:**
```bash
# 1. Basic file analysis
file suspect.jpg
exiftool suspect.jpg
binwalk suspect.jpg

# 2. Check file size anomaly
ls -la suspect.jpg
# Compare with expected size for resolution

# 3. Try steghide info
steghide info suspect.jpg

# 4. Try extraction with empty passphrase
steghide extract -sf suspect.jpg -xf extracted.txt -p "" -f

# 5. Try extraction with common passwords
steghide extract -sf suspect.jpg -xf extracted.txt -p "password" -f
steghide extract -sf suspect.jpg -xf extracted.txt -p "123456" -f

# 6. Dictionary attack
stegcracker suspect.jpg /usr/share/wordlists/rockyou.txt

# 7. Check extracted data
cat extracted.txt
```

---

## Chapter 7: Detection & Defense

### How Steghide is Detected

#### File Analysis

```bash
# File size comparison
ls -la original.jpg stego.jpg
# Stego file is usually slightly larger

# Entropy analysis
binwalk -E suspect.jpg
# Higher entropy in stego files

# Magic byte verification
hexdump -C suspect.jpg | head -5
```

#### Statistical Analysis

```bash
# Chi-square test
stegdetect suspect.jpg

# Visual analysis
# Zoom into image, look for unusual patterns
# Check for color banding in smooth gradients
```

#### Steghide-Specific Detection

```bash
# Steghide has a known signature in JPEG files
# Check for steghide markers
strings suspect.jpg | grep -i "steghide"

# Check file structure
binwalk suspect.jpg
# Look for embedded data markers
```

### Mitigation Strategies

```bash
# For organizations:
# 1. Recompress all outbound images (destroys steganography)
# 2. Convert images to different format
# 3. Strip metadata from images
# 4. Monitor bulk image uploads
# 5. Deploy steganalysis tools
```

### Blue Team Response

1. **Image recompression** — Recompress all outbound images at lower quality
2. **Format conversion** — Convert JPEG to PNG and back (destroys LSB data)
3. **Metadata stripping** — Remove EXIF data from all outbound images
4. **Upload monitoring** — Alert on bulk image uploads
5. **Steganalysis tools** — Deploy automated detection tools
6. **Network DLP** — Detect sensitive data in image uploads

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "steghide: could not embed data"

**Cause:** Cover file too small or already contains hidden data

**Solution:**
```bash
# Use a larger cover file
# Check if file already has data
steghide info suspect.jpg
```

#### Error: "steghide: passphrase wrong"

**Cause:** Incorrect passphrase during extraction

**Solution:**
```bash
# Ensure exact passphrase match (case-sensitive)
# Try empty passphrase
steghide extract -sf stego.jpg -p "" -f
```

#### Error: "steghide: file format not supported"

**Cause:** File is not JPEG, BMP, WAV, or AU

**Solution:**
```bash
# Check file type
file image.jpg
# Convert to supported format
convert image.png image.jpg
```

### FAQ

**Q: Is steghide legal to use?**
A: Yes, for authorized security testing, CTF challenges, and educational purposes.

**Q: Can steghide encrypt data?**
A: Yes, steghide encrypts embedded data using Blowfish or other algorithms via libmcrypt.

**Q: How much data can steghide hide?**
A: Depends on cover file size and quality. A typical JPEG can hold ~5-15% of its size in hidden data.

**Q: What's the difference between steghide and outguess?**
A: Steghide uses pseudo-random LSB distribution with encryption. OutGuess modifies Huffman code redundancy in JPEG, preserving statistical properties better.

**Q: Can steghide data survive JPEG recompression?**
A: No, JPEG recompression destroys steghide data. This is both a weakness and a defense mechanism.

---

## Resources

### Official Documentation

- [Steghide GitHub](https://github.com/StefanoDeVuono/steghide)
- [Steghide SourceForge](http://steghide.sourceforge.net/)
- [Kali Linux Steghide](https://www.kali.org/tools/steghide/)
- [Steghide Man Page](https://steghide.sourceforge.net/documentation/man_page.php)

### Tutorials

- [Steghide Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/steganography)
- [CTF Steganography Guide](https://github.com/apsdehal/ctf-wiki/tree/master/crypto/stego)

### Related Tools

- **outguess:** JPEG steganography using Huffman code redundancy
- **stegsnow:** ASCII text steganography
- **stegosuite:** GUI steganography tool
- **zsteg:** PNG/BMP steganography
- **stegcracker:** Steghide password cracker

### Practice Resources

- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [CTFtime](https://ctftime.org/)
