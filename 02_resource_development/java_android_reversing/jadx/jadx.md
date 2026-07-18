---
tool_name: jadx
display_name: JADX - DEX/APK to Java Source Decompiler
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: java_android_reversing
install_command: "sudo apt install jadx"
version: "1.5.6"
github_repo: "https://github.com/skylot/jadx"
kali_tools_page: "https://www.kali.org/tools/jadx/"
difficulty_level: beginner
requires_root: false
gui_available: true
tags: [android, decompiler, java, dex, apk, source-code, deobfuscation]
related_tools: [apktool, dex2jar, jadx-gui, bytecode-viewer, jd-gui]
---

# JADX - DEX/APK to Java Source Decompiler

## 1. Introduction & Overview

JADX is the most popular open-source Android decompiler, converting DEX (Dalvik Executable) files back to readable Java source code. With nearly 50,000 GitHub stars, it's the go-to tool for Android security researchers, malware analysts, and CTF players. JADX provides both a command-line interface and a feature-rich GUI.

**Created:** 2013 by skylot
**License:** Apache 2.0
**GitHub Stars:** 49,700+

### What It Does

- Decompiles DEX, APK, AAR, AAB, and ZIP files to Java source code
- Decodes `AndroidManifest.xml` and resources from `resources.arsc`
- Built-in deobfuscator that renames obfuscated classes/methods
- GUI with syntax highlighting, navigation, and search
- Smali debugger support in GUI mode
- Plugin system for extensibility

### DEX Format Reference

| DEX Component | Description |
|---------------|-------------|
| **header** | Magic number, checksum, SHA-1 signature |
| **string_ids** | Table of string literals |
| **type_ids** | Table of type references |
| **proto_ids** | Method prototypes (return type + parameters) |
| **field_ids** | Field references |
| **method_ids** | Method references |
| **class_defs** | Class definitions with bytecode |
| **data** | Actual bytecode instructions, strings, annotations |

### When to Use

- **Android app security audit**: Read decompiled Java source to find vulnerabilities
- **Malware analysis**: Understand malware behavior through readable code
- **API analysis**: Study how apps interact with backend services
- **CTF challenges**: Reverse engineer Android apps to find flags
- **Learning**: Study Android app architecture and patterns

### When NOT to Use

- Target app has extremely heavy obfuscation — JADX can decompile but output may be unreadable
- You need to modify and rebuild APKs (use Apktool instead)
- iOS or desktop Java applications (use JD-GUI for JAR files)

### Legal & Ethical

JADX is for authorized security research and education. Decompiling commercial apps may violate terms of service. Always obtain permission before analyzing third-party applications.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Decompilation** | Converting bytecode back to source code (lossy process) |
| **Deobfuscation** | Renaming obfuscated identifiers to human-readable names |
| **ProGuard/R8** | Android's default code obfuscators |
| **Smali** | Assembly-level Dalvik bytecode (JADX goes higher — to Java) |
| **AAB** | Android App Bundle — Google Play's modern delivery format |

## 2. Installation & Setup

### System Requirements

- Java 11+ (64-bit)
- ~200 MB disk space
- 4+ GB RAM recommended for large APKs

### Installation

```bash
sudo apt install jadx
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  jadx
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.4 MB of archives.
After this operation, 42.6 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 jadx all 1.5.6-0kali1 [18.4 MB]
Fetched 18.4 MB in 1s (14.2 MB/s)
Selecting previously unselected package jadx.
Preparing to unpack .../jadx_1.5.6-0kali1_all.deb ...
Unpacking jadx (1.5.6-0kali1) ...
Setting up jadx (1.5.6-0kali1) ...
```

### From GitHub Releases (Alternative)

```bash
# Download latest release
wget https://github.com/skylot/jadx/releases/download/v1.5.6/jadx-1.5.6.zip
unzip jadx-1.5.6.zip -d jadx
chmod +x jadx/bin/jadx jadx/bin/jadx-gui
```

### Verification

```bash
jadx --version
```

**Expected output:**
```
jadx 1.5.6
```

## 3. Basic Usage

### Decompiling an APK (CLI)

```bash
jadx -d output_dir target_app.apk
```

**Explanation:** Decompiles the APK and writes Java source code to `output_dir/`.

**Example output:**
```
INFO  - loading jadx args...
INFO  - loading class analysis...
INFO  - loading and decoding resources...
INFO  - processing class files... (245/245)
INFO  - processing data... done
INFO  - Total time: 3.2s
INFO  - Output directory: output_dir
```

**Result structure:**
```
output_dir/
├── sources/                    # Java source code
│   └── com/
│       └── example/
│           └── app/
│               ├── MainActivity.java
│               ├── ApiService.java
│               └── utils/
│                   └── CryptoUtils.java
└── resources/                  # Decoded resources
    ├── AndroidManifest.xml
    └── res/
        ├── layout/
        │   └── activity_main.xml
        └── values/
            └── strings.xml
```

### Decompiling a Single DEX File

```bash
jadx -d output_dir classes.dex
```

**Explanation:** Decompiles a standalone DEX file without resources.

### Decompiling with Resources Only

```bash
jadx -d output_dir --no-src target_app.apk
```

**Explanation:** Decodes only resources (manifest, layouts, strings) without decompiling Java code.

### Decompiling Source Only

```bash
jadx -d output_dir --no-res target_app.apk
```

**Explanation:** Decompiles only Java source code, skipping resource decoding.

### Launching the GUI

```bash
jadx-gui target_app.apk
```

**Explanation:** Opens the JADX GUI with the APK loaded for interactive browsing.

### Flags Reference

| Flag | Description |
|------|-------------|
| `-d DIR` | Output directory |
| `-ds DIR` | Output directory for sources only |
| `-dr DIR` | Output directory for resources only |
| `-r`, `--no-res` | Don't decode resources |
| `-s`, `--no-src` | Don't decompile source code |
| `-e`, `--export-gradle` | Export as Gradle project |
| `-j NUM` | Thread count (default: 16) |
| `--show-bad-code` | Show inconsistent/corrupted code |
| `--deobf` | Enable deobfuscation |
| `--deobf-min NUM` | Min name length for deobfuscation (default: 3) |
| `--deobf-max NUM` | Max name length for deobfuscation (default: 64) |
| `--cfg` | Save control flow graphs (dot files) |
| `--call-graph` | Save call graph (dot or json) |
| `-f`, `--fallback` | Use fallback decompilation mode |
| `-q`, `--quiet` | Suppress output |
| `-v`, `--verbose` | Verbose output |

## 4. Intermediate Usage

### Deobfuscation

```bash
jadx -d output_dir --deobf target_app.apk
```

**Explanation:** Enables JADX's deobfuscator, renaming obfuscated identifiers (e.g., `a.b.c` → `com.example.app.utils.Helper`).

### Custom Deobfuscation Thresholds

```bash
jadx -d output_dir --deobf --deobf-min 4 --deobf-max 100 target_app.apk
```

**Explanation:** Adjusts which identifier lengths trigger deobfuscation.

### Export as Gradle Project

```bash
jadx -d output_dir --export-gradle target_app.apk
```

**Explanation:** Exports the decompiled code as a Gradle project, importable into Android Studio.

### Single Class Decompilation

```bash
jadx -d output_dir --single-class com.example.app.MainActivity target_app.apk
```

**Explanation:** Decompiles only the specified class — faster for targeted analysis.

### JSON Output

```bash
jadx -d output_dir --output-format json target_app.apk
```

**Explanation:** Outputs decompiled code as JSON for programmatic analysis.

### Multi-threaded Processing

```bash
jadx -d output_dir -j 8 large_app.apk
```

**Explanation:** Uses 8 threads instead of the default 16, useful for memory-constrained systems.

### Batch Decompilation

```bash
#!/bin/bash
for apk in *.apk; do
    name=$(basename "$apk" .apk)
    jadx -d "output/$name" --deobf "$apk" 2>/dev/null
done
```

**Explanation:** Decompile multiple APKs in sequence.

## 5. Advanced Usage

### Call Graph Analysis

```bash
jadx -d output_dir --call-graph dot target_app.apk
# Render the graph
dot -Tpng output_dir/call-graph.dot -o call_graph.png
```

**Explanation:** Generates a call graph showing method relationships, useful for understanding app architecture.

### Control Flow Graph Analysis

```bash
jadx -d output_dir --cfg target_app.apk
# View CFG files
ls output_dir/sources/com/example/app/*.dot
```

**Explanation:** Saves method control flow graphs as GraphViz dot files for analysis.

### Searching Decompiled Code

```bash
# After decompilation
grep -rn "http://" output_dir/sources/
grep -rn "password\|secret\|api_key" output_dir/sources/
grep -rn "getDeviceId\|getSubscriberId" output_dir/sources/
grep -rn "SharedPreferences\|getSharedPreferences" output_dir/sources/
```

**Explanation:** Use standard Unix tools to search decompiled source for patterns of interest.

### Plugin System

```bash
# List installed plugins
jadx plugins --list

# Install a plugin
jadx plugins -i plugin-location-id

# Disable a plugin
jadx plugins --disable plugin-id
```

**Explanation:** JADX supports plugins for custom decompilation logic.

### Environment Variables

```bash
# Disable XML security checks (for malformed APKs)
JADX_DISABLE_XML_SECURITY=true jadx -d output_dir target.apk

# Increase zip entry limit
JADX_ZIP_MAX_ENTRIES_COUNT=200000 jadx -d output_dir target.apk
```

**Explanation:** Environment variables control JADX's security checks and resource limits.

### Using JADX as a Library

```java
// In your Java/Kotlin project
JadxArgs args = new JadxArgs();
args.setInputFiles(List.of(new File("target.apk")));
JadxDecompiler jadx = new JadxDecompiler(args);
jadx.load();
// Access classes, methods, fields programmatically
for (ClassNode cls : jadx.getClasses()) {
    System.out.println(cls.getName());
}
```

**Explanation:** JADX can be embedded in Java projects for programmatic decompilation.

## 6. Real-World Scenarios

### Scenario 1: Android App Security Audit

**Target:** Banking application

```bash
# Full decompile with deobfuscation
jadx -d bank_audit --deobf banking_app.apk

# Search for hardcoded secrets
grep -rn "api_key\|API_KEY\|apiKey\|secret\|password" bank_audit/sources/

# Check for insecure HTTP
grep -rn "http://" bank_audit/sources/

# Check for certificate pinning bypass
grep -rn "TrustManager\|X509TrustManager\|checkServerTrusted" bank_audit/sources/

# Check for debug mode
grep -rn "BuildConfig.DEBUG\|isDebuggable" bank_audit/sources/

# Audit permissions
cat bank_audit/resources/AndroidManifest.xml | grep "uses-permission"
```

### Scenario 2: Malware Analysis

**Target:** Suspicious APK sample

```bash
# Decompile
jadx -d malware_analysis --show-bad-code --deobf suspicious.apk

# Find C2 communication
grep -rn "http\|https\|socket\|WebSocket" malware_analysis/sources/

# Find data collection
grep -rn "getDeviceId\|getIMEI\|getAccounts\|getInstalledPackages" malware_analysis/sources/

# Find encryption routines
grep -rn "Cipher\|SecretKey\|AES\|RSA\|encrypt\|decrypt" malware_analysis/sources/

# Find dynamic code loading
grep -rn "DexClassLoader\|loadClass\|Runtime.exec\|ProcessBuilder" malware_analysis/sources/
```

### Scenario 3: CTF Android Challenge

**Context:** Find the flag in a custom Android app

```bash
jadx -d challenge --deobf challenge.apk

# Search for flag patterns
grep -rn "flag\|FLAG\|ctf\|CTF" challenge/sources/
grep -rn "base64\|Base64" challenge/sources/

# Check for hardcoded strings
grep -rn "string" challenge/sources/ | grep -i "key\|secret\|pass"

# Check native libraries
ls challenge/resources/lib/
```

### Scenario 4: API Endpoint Discovery

**Context:** Find all API endpoints used by an app

```bash
jadx -d api_audit target_app.apk

# Find all URL patterns
grep -roh "https://[a-zA-Z0-9./_?=&-]*" api_audit/sources/ | sort -u

# Find Retrofit/OkHttp endpoints
grep -rn "@GET\|@POST\|@PUT\|@DELETE\|@PATCH" api_audit/sources/

# Find Volley requests
grep -rn "RequestQueue\|StringRequest\|JsonObjectRequest" api_audit/sources/
```

## 7. Detection & Defense

### How Defenders Detect

- **App integrity verification**: Runtime signature checking
- **Obfuscation**: ProGuard/R8 makes decompiled code harder to read
- **String encryption**: Runtime decryption of sensitive strings
- **Anti-analysis**: Detection of debugging, emulation, or decompilation tools

### Mitigation (For Developers)

- Enable ProGuard/R8 obfuscation
- Use Android App Bundle for dynamic feature delivery
- Implement runtime integrity checks
- Encrypt sensitive strings at build time
- Use native code (NDK) for critical algorithms

### Blue Team Response

```bash
# Verify APK integrity
apksigner verify --verbose --print-certs target.apk
# Check for known malicious patterns
yara -r malware_rules/ suspicious.apk
```

## 8. Troubleshooting

### Common Errors

**"Java 11+ required"**
```
Error: A JNI error has occurred
```
**Fix:** Install Java 11+: `sudo apt install default-jdk`

**"Can't load converted smali"**
**Fix:** APK may use advanced DEX features. Try `--show-bad-code` flag.

**"Out of memory"**
```
java.lang.OutOfMemoryError: Java heap space
```
**Fix:** Increase heap: `jadx -Xmx4g -d output_dir target.apk`

**Incomplete decompilation**
JADX explicitly warns that 100% decompilation is not always possible. Some bytecode constructs cannot be represented in Java. Use `--show-bad-code` to see what JADX could recover.

### Performance Optimization

- Use `-j` to control thread count based on available CPU cores
- Use `--no-res` when only source code is needed
- Use `--no-src` when only resources are needed
- For large APKs, use a machine with 8+ GB RAM

### FAQ

**Q: JADX vs Apktool?**
A: JADX decompiles to Java source code (higher level). Apktool disassembles to smali (lower level) but supports APK rebuilding. Use both for complete analysis.

**Q: How accurate is JADX's decompilation?**
A: JADX handles ~95-99% of common Android bytecode correctly. Edge cases (complex switch statements, obfuscated control flow) may produce imperfect output.

**Q: Can JADX decompile R8/ProGuard code?**
A: Yes, but the output uses obfuscated names. Use `--deobf` to get better naming.

---

## Resources

- **GitHub**: https://github.com/skylot/jadx
- **Releases**: https://github.com/skylot/jadx/releases
- **Wiki**: https://github.com/skylot/jadx/wiki
- **Troubleshooting**: https://github.com/skylot/jadx/wiki/Troubleshooting-Q&A
- **JADX GUI Key Bindings**: https://github.com/skylot/jadx/wiki/JADX-GUI-Key-bindings
- **Android Developer Docs**: https://developer.android.com/guide
