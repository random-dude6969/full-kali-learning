---
tool_name: apktool
display_name: Apktool - APK Reverse Engineering Tool
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: java_android_reversing
install_command: "sudo apt install apktool"
version: "3.0.2"
github_repo: "https://github.com/iBotPeaches/Apktool"
kali_tools_page: "https://www.kali.org/tools/apktool/"
difficulty_level: intermediate
requires_root: false
gui_available: false
tags: [android, apk, reverse-engineering, smali, resources, bytecode]
related_tools: [jadx, dex2jar, jadx-gui, bytecode-viewer]
---

# Apktool - APK Reverse Engineering Tool

## 1. Introduction & Overview

Apktool is the industry-standard tool for reverse engineering Android APK files. It decodes Android resources (layouts, strings, drawables, manifest) back to their nearly original form and allows modification and rebuilding of APKs. Unlike decompilers that focus on Java source code, Apktool specializes in the Android resource layer — XML layouts, `AndroidManifest.xml`, `resources.arsc`, and smali bytecode.

**Created:** 2010 by Connor Tumbleson (iBotPeaches)
**License:** Apache 2.0
**GitHub Stars:** 25,000+

### What It Does

- Decodes APK resources to near-original form (XML, images, manifest)
- Disassembles DEX to smali (readable Dalvik bytecode)
- Rebuilds modified APKs after editing
- Handles complex resource formats: 9-patch images, binary XML, ARSC tables
- Debugging support for smali code

### APK Structure (Reference)

```
app.apk/
├── AndroidManifest.xml        # Binary XML (decoded by apktool)
├── classes.dex                 # Dalvik bytecode (smali when disassembled)
├── classes2.dex                # Secondary DEX (multi-dex apps)
├── res/                        # Decoded resources
│   ├── layout/                 # XML layouts
│   ├── values/                 # Strings, colors, styles
│   ├── drawable/               # Images
│   └── ...
├── resources.arsc              # Compiled resource mapping table
├── lib/                        # Native libraries (.so)
│   ├── armeabi-v7a/
│   └── arm64-v8a/
├── assets/                     # Raw asset files
├── META-INF/                   # Signing information
│   ├── CERT.RSA
│   ├── CERT.SF
│   └── MANIFEST.MF
└── original/                   # Original files preserved by apktool
    ├── AndroidManifest.xml
    └── META-INF/
```

### When to Use

- **Security auditing**: Inspect Android apps for hardcoded secrets, insecure permissions, vulnerable code paths
- **Localization**: Translate app resources to other languages
- **Modifying APKs**: Change permissions, modify layouts, replace resources
- **Malware analysis**: Understand Android malware behavior and communication patterns
- **CTF challenges**: Solve mobile reversing challenges

### When NOT to Use

- You need high-level Java source code (use jadx instead)
- Target app uses heavy obfuscation (ProGuard, R8) — smali output will be obfuscated
- You need to analyze iOS apps — Apktool is Android-only

### Legal & Ethical

Apktool is for authorized security research, malware analysis, and personal app modifications. Reverse engineering commercial apps may violate terms of service. Modifying and redistributing APKs without permission is illegal in most jurisdictions.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Smali** | Human-readable assembly language for Dalvik bytecode |
| **Resource ARSC** | Compiled resource mapping table (`resources.arsc`) |
| **9-patch** | Android's stretchable image format (`.9.png`) |
| **Binary XML** | Android's compiled XML format |
| **Manifest** | `AndroidManifest.xml` — declares components, permissions, API levels |
| **Multi-dex** | Apps with multiple `classes.dex` files for large codebases |

## 2. Installation & Setup

### System Requirements

- Java 8+ (JRE)
- ~50 MB disk space
- Linux, macOS, or Windows

### Installation

```bash
sudo apt install apktool
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  apktool
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 12.8 MB of archives.
After this operation, 12.8 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 apktool all 3.0.2-0kali1 [12.8 MB]
Fetched 12.8 MB in 1s (8.4 MB/s)
Selecting previously unselected package apktool.
Preparing to unpack .../apktool_3.0.2-0kali1_all.deb ...
Unpacking apktool (3.0.2-0kali1) ...
Setting up apktool (3.0.2-0kali1) ...
```

### From Source (Alternative)

```bash
git clone https://github.com/iBotPeaches/Apktool.git
cd Apktool
./gradlew build distZip
```

### Verification

```bash
apktool --version
```

**Expected output:**
```
Apktool v3.0.2 - a tool for reverse engineering Android apk files
```

## 3. Basic Usage

### Decoding an APK

```bash
apktool d target_app.apk -o decoded_app
```

**Explanation:** Decodes `target_app.apk` into the `decoded_app` directory, extracting all resources and smali code.

**Example output:**
```
I: Using Apktool 3.0.2 on target_app.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/user/.local/share/apktool/...
I: Regular manifest package...
I: Decoding file-res/...
I: Decoding values/...
I: Decoding values*/...
I: Copying assets and libs...
I: Copying unknown files/dir...
I: Copying original files/dir...
I: Built apk into: decoded_app
```

**Result structure:**
```
decoded_app/
├── AndroidManifest.xml        # Readable XML
├── apktool.yml               # Apktool metadata
├── original/                  # Original files
├── res/                       # Decoded resources
├── smali/                     # Disassembled Dalvik bytecode
│   └── com/
│       └── example/
│           └── app/
│               └── MainActivity.smali
└── smali_classes2/            # Secondary DEX (if present)
```

### Rebuilding an APK

```bash
apktool b decoded_app -o modified_app.apk
```

**Explanation:** Rebuilds the modified `decoded_app/` directory into a new APK.

**Example output:**
```
I: Using Apktool 3.0.2
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether sources has changed...
I: Smaling smali_classes2 folder into classes2.dex...
I: Building resources...
I: Building apk file...
I: Copying raw files...
I: Built apk into: modified_app.apk
```

### Signing the Modified APK

```bash
keytool -genkey -v -keystore release.keystore -alias mykey -keyalg RSA -keysize 2048 -validity 10000
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore release.keystore modified_app.apk mykey
```

**Explanation:** Android requires APKs to be signed. Generate a keystore and sign the modified APK.

### Flags Reference

| Flag | Description |
|------|-------------|
| `d`, `decode` | Decode an APK to directory |
| `b`, `build` | Build an APK from directory |
| `-o FILE` | Output file or directory |
| `-f`, `force` | Force overwrite existing output |
| `-s`, `no-src` | Don't disassemble DEX (resources only) |
| `-r`, `no-res` | Don't decode resources |
| `-d`, `debug` | Build with debug mode |
| `-b`, `debuggable` | Mark as debuggable |
| `-a`, `aapt` | Path to aapt binary |
| `-t`, `tag` | Add a tag to the debug manifest |
| `--keep-fresn` | Keep file timestamps |
| `-p DIR` | Directory for framework files |

## 4. Intermediate Usage

### Decoding Resources Only (No Smali)

```bash
apktool d -r target_app.apk -o decoded_resources
```

**Explanation:** Decodes only resources, skipping DEX disassembly. Useful for resource-focused analysis.

### Disassembling DEX Only (No Resources)

```bash
apktool d -s target_app.apk -o decoded_smali
```

**Explanation:** Extracts DEX files without decoding resources. Faster when you only need bytecode.

### Using Framework Files

```bash
# Install framework
apktool if framework-res.apk

# Decode app using framework
apktool d target_app.apk
```

**Explanation:** Some apps reference Android framework resources. Install the framework first to decode them correctly.

### Batch Processing

```bash
#!/bin/bash
mkdir -p decoded_apps/
for apk in *.apk; do
    echo "Decoding $apk..."
    apktool d "$apk" -o "decoded_apps/$(basename "$apk" .apk)"
done
```

**Explanation:** Automates decoding of multiple APK files.

### Extracting the Manifest

```bash
apktool d -r -s target_app.apk -o temp/
cat temp/AndroidManifest.xml
grep -i "permission\|intent-filter\|service\|receiver" temp/AndroidManifest.xml
```

**Explanation:** Quick manifest analysis for permissions, services, and broadcast receivers.

## 5. Advanced Usage

### Smali Code Analysis

```bash
# Search for sensitive API calls
grep -r "getDeviceId\|getSubscriberId\|getLine1Number" decoded_app/smali/
grep -r "http://\|https://" decoded_app/smali/
grep -r "Runtime.getRuntime" decoded_app/smali/
```

**Explanation:** Find hardcoded URLs, dangerous API calls, and dynamic code execution patterns in smali.

### Permission Analysis

```bash
grep -i "uses-permission" decoded_app/AndroidManifest.xml | sort -u
```

**Explanation:** Extract all declared permissions to assess app's access level.

### Modifying Permissions

```xml
<!-- In decoded_app/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET"/>
<!-- Remove dangerous permissions by deleting the line -->
```

**Explanation:** Edit the manifest to change app permissions before rebuilding.

### Understanding Smali Basics

```smali
# Smali syntax reference
.class public Lcom/example/app/MainActivity;
.super Landroid/app/Activity;

.method public onCreate(Landroid/os/Bundle;)V
    .registers 4
    invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V
    const-string v0, "MainActivity"
    invoke-static {v0}, Landroid/util/Log;->d(Ljava/lang/String;)I
    return-void
.end method
```

**Explanation:** Smali is the assembly language for Dalvik VM. Understanding basic smali syntax enables reading and modifying APK behavior.

### Automated Security Audit

```bash
#!/bin/bash
APK=$1
decoded=$(mktemp -d)

apktool d "$APK" -o "$decoded"

echo "=== Package: $(grep 'package=' "$decoded/AndroidManifest.xml") ==="
echo ""
echo "=== Permissions ==="
grep "uses-permission" "$decoded/AndroidManifest.xml" | sed 's/.*name="//' | sed 's/".*//'
echo ""
echo "=== Hardcoded URLs ==="
grep -roh "http[s]*://[a-zA-Z0-9./?=_-]*" "$decoded/smali/" 2>/dev/null | sort -u | head -20
echo ""
echo "=== Dangerous APIs ==="
grep -rl "getDeviceId\|getSubscriberId\|exec\|Runtime" "$decoded/smali/" 2>/dev/null
echo ""
echo "=== Debuggable ==="
grep "debuggable" "$decoded/AndroidManifest.xml"
echo ""
echo "=== Backup Allowed ==="
grep "allowBackup" "$decoded/AndroidManifest.xml"

rm -rf "$decoded"
```

**Explanation:** Automated security assessment script that checks for common Android vulnerabilities.

## 6. Real-World Scenarios

### Scenario 1: Malware Analysis

**Target:** Suspicious APK file

```bash
# Decode
apktool d suspicious.apk -o malware_analysis/

# Check manifest for permissions and components
cat malware_analysis/AndroidManifest.xml

# Search for C2 URLs
grep -roh "http[s]*://[^\"']*" malware_analysis/smali/ | sort -u

# Look for data exfiltration
grep -r "SmsManager\|sendTextMessage\|getDeviceId" malware_analysis/smali/

# Check for dynamic loading
grep -r "DexClassLoader\|PathClassLoader\|loadClass" malware_analysis/smali/
```

### Scenario 2: CTF Mobile Challenge

**Context:** Reverse engineer a mobile app to find hidden flag

```bash
apktool d challenge.apk -o challenge_decoded/
# Look for hardcoded strings
grep -r "flag\|FLAG\|secret\|KEY" challenge_decoded/smali/
# Check assets for hidden files
ls challenge_decoded/original/
cat challenge_decoded/assets/config.json
```

### Scenario 3: Security Audit — Hardcoded Credentials

```bash
apktool d banking_app.apk -o banking_audit/
# Search for hardcoded secrets
grep -rn "password\|api_key\|secret\|token\|auth" banking_audit/smali/
grep -rn "password\|api_key\|secret" banking_audit/res/values/strings.xml
```

### Scenario 4: Permission Audit

```bash
apktool d target.apk -o target_decoded/
# Extract all permissions
grep "uses-permission" target_decoded/AndroidManifest.xml | \
    sed 's/.*name="//' | sed 's/".*//' | sort -u > permissions.txt
# Compare against expected permissions
diff expected_permissions.txt permissions.txt
```

## 7. Detection & Defense

### How Defenders Detect

- **APK modification detection**: Signature verification fails on modified APKs
- **Integrity checks**: Apps can verify their own signature at runtime
- **Anti-tampering**: Some apps use Google Play Integrity API
- **Obfuscation detection**: ProGuard/R8 obfuscation makes smali harder to read

### Mitigation (For App Developers)

- Enable code obfuscation (ProGuard/R8)
- Implement runtime integrity checks
- Use Android App Bundle for dynamic delivery
- Obfuscate resource names
- Implement certificate pinning

### Blue Team Response

```bash
# Verify APK signature
apksigner verify --verbose target_app.apk
# Check for known malware indicators
yara -r malware_rules/ suspicious.apk
```

## 8. Troubleshooting

### Common Errors

**"Can't find framework resources"**
```
apktool d target.apk
I: Framework built-in resources not found
```
**Fix:** Install the framework: `apktool if framework-res.apk`

**"Invalid apk file"**
**Fix:** Verify the file: `file target.apk` (should say "Zip archive data"). Try re-downloading.

**Build fails with resource errors**
```
I: Building resources...
aapt: error: resource android:attr/lane not found
```
**Fix:** Ensure the framework is installed and the target Android SDK version matches.

### Performance Optimization

- Use `-r` (no-res) when you only need smali code
- Use `-s` (no-src) when you only need resources
- For large APKs, increase Java heap: `JAVA_OPTS="-Xmx4g" apktool d large.apk`

### FAQ

**Q: Apktool vs jadx?**
A: Apktool focuses on resource decoding and APK rebuilding. JADX focuses on Java source code decompilation. Use both: Apktool for resources, JADX for code analysis.

**Q: Can I decompile obfuscated APKs?**
A: Yes, but the smali code will be obfuscated (e.g., `a.b.c()` instead of `MainActivity`). Use JADX's deobfuscator or manual analysis.

**Q: How do I handle multi-DEX APKs?**
A: Apktool handles them automatically. Each DEX file becomes a `smali_classesN/` directory.

---

## Resources

- **GitHub**: https://github.com/iBotPeaches/Apktool
- **Documentation**: https://apktool.org/docs
- **XDA Thread**: https://forum.xda-developers.com/t/util-dec-2-2020-apktool-tool-for-reverse-engineering-apk-files.1755243/
- **Smali Syntax**: https://smalilib.blogspot.com/
- **Android Security**: https://developer.android.com/topic/security/best-practices
