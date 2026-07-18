---
tool_name: jd-gui
display_name: JD-GUI - Java Decompiler GUI
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: java_android_reversing
install_command: "sudo apt install jd-gui"
version: "1.6.6"
github_repo: "https://github.com/java-decompiler/jd-gui"
kali_tools_page: "https://www.kali.org/tools/jd-gui/"
difficulty_level: beginner
requires_root: false
gui_available: true
tags: [java, decompiler, gui, jar, class, bytecode, reverse-engineering]
related_tools: [jadx, bytecode-viewer, dex2jar, cfr, procyon]
---

# JD-GUI - Java Decompiler GUI

## 1. Introduction & Overview

JD-GUI is one of the most well-known and widely used Java decompilers with a graphical interface. It reads compiled Java `.class` files and `.jar` archives and displays the reconstructed Java source code with syntax highlighting. It's fast, reliable, and has been a staple in the Java reverse engineering community for over a decade.

**Created:** 2008 by Emmanuel Dupuy
**License:** GNU GPL v3 (with exceptions)
**GitHub Stars:** 9,800+

### What It Does

- Decompiles `.class` files to readable Java source code
- Opens `.jar` archives and displays the full class hierarchy
- Syntax highlighting with import organization
- Search across all decompiled classes
- Export decompiled source as ZIP

### When to Use

- **Quick JAR inspection**: Fastest way to browse a JAR file's contents
- **Java application analysis**: Understand third-party library behavior
- **Legacy code analysis**: View decompiled code from old/deprecated libraries
- **Quick triage**: Rapid initial assessment before deeper analysis with JADX

### When NOT to Use

- Android APK/DEX files (use JADX or dex2jar first)
- Need to modify and rebuild code (use Apktool or bytecode-viewer)
- Need deobfuscation (use JADX with --deobf flag)
- Batch processing (use jadx CLI)

### Legal & Ethical

JD-GUI is for authorized security research and education. Decompiling proprietary software may violate terms of service.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Class file** | Compiled Java bytecode (`.class`) |
| **JAR** | Java Archive — ZIP containing class files |
| **Bytecode** | Intermediate representation between source and machine code |
| **Decompilation** | Converting bytecode back to source code (lossy) |

## 2. Installation & Setup

### System Requirements

- Java 8+ (JRE)
- ~30 MB disk space
- Display server (X11/Wayland)

### Installation

```bash
sudo apt install jd-gui
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  jd-gui
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 8.4 MB of archives.
After this operation, 30.2 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 jd-gui all 1.6.6-0kali1 [8.4 MB]
Fetched 8.4 MB in 1s (6.8 MB/s)
Selecting previously unselected package jd-gui.
Preparing to unpack .../jd-gui_1.6.6-0kali1_all.deb ...
Unpacking jd-gui (1.6.6-0kali1) ...
Setting up jd-gui (1.6.6-0kali1) ...
```

### From GitHub Releases (Alternative)

```bash
wget https://github.com/java-decompiler/jd-gui/releases/download/v1.6.6/jd-gui-1.6.6.jar
java -jar jd-gui-1.6.6.jar
```

### Verification

```bash
jd-gui --version 2>/dev/null || dpkg -L jd-gui | grep "\.jar"
```

## 3. Basic Usage

### Launching the GUI

```bash
jd-gui
```

**Explanation:** Opens JD-GUI. Use File → Open to load a JAR or class file.

### Opening a JAR File

```bash
jd-gui target_app.jar
```

**Explanation:** Opens the JAR file directly in JD-GUI for browsing.

**GUI layout:**
- **Left panel**: Class hierarchy tree (packages → classes → methods)
- **Right panel**: Decompiled Java source code with syntax highlighting
- **Bottom panel**: Search results

### Navigation

- **Click** a class in the tree to view its decompiled source
- **Double-click** a method to jump to its implementation
- **Ctrl+F** to search across all loaded classes
- **Ctrl+G** to go to a specific class

### Exporting Source

```bash
# File → Save All Sources → export as ZIP
# Or use command line:
jd-gui -export target_app.jar
```

**Explanation:** Exports all decompiled source code as a ZIP file for offline analysis.

### Command-Line Options

| Option | Description |
|--------|-------------|
| `-export` | Export all sources to directory |
| `-no النوافذ` | No GUI mode (CLI export only) |

## 4. Intermediate Usage

### Batch Export

```bash
#!/bin/bash
for jar in *.jar; do
    name=$(basename "$jar" .jar)
    jd-gui -export "$jar" 2>/dev/null
    mv sources.zip "exported_${name}.zip"
done
```

**Explanation:** Automates decompilation and export of multiple JAR files.

### Searching for Vulnerabilities

```
# In JD-GUI search bar:
# SQL: "Statement.execute"
# Crypto: "Cipher.init"
# Network: "HttpURLConnection"
# Secrets: "password"
```

**Explanation:** Use JD-GUI's search to quickly locate potentially vulnerable code patterns.

### Comparing Library Versions

```bash
# Open both versions side-by-side
jd-gui library_v1.jar &
jd-gui library_v2.jar &
# Compare decompiled output manually
```

### Integration with dex2jar

```bash
# Convert Android APK to JAR first
d2j-dex2jar.sh target_app.apk
# Open converted JAR in JD-GUI
jd-gui target_app-dex2jar.jar
```

**Explanation:** JD-GUI can analyze Android apps after DEX-to-JAR conversion.

## 5. Advanced Usage

### Static Analysis Patterns

```java
// Search in JD-GUI for these patterns:

// SQL Injection
"Statement.execute"
"PreparedStatement"
"createQuery"

// Insecure Deserialization
"ObjectInputStream"
"readObject"

// Path Traversal
"FileInputStream"
"new File("

// Hardcoded Credentials
"password"
"secret"
"api_key"
"getConnection("
```

### Export for Further Analysis

```bash
# Export sources
jd-gui -export target.jar
# Grep exported source
grep -r "password" sources/ | grep -v "\.class"
grep -rn "http://" sources/
```

## 6. Real-World Scenarios

### Scenario 1: Third-Party Library Audit

**Target:** Log4j JAR file

```bash
jd-gui log4j-core-2.14.1.jar
# Search for JNDI lookup patterns
# Ctrl+F → "JndiLookup"
# Review the vulnerable class
```

### Scenario 2: Java Application Security Test

```bash
jd-gui target_app.jar
# Search for:
# - Hardcoded passwords
# - Insecure HTTP connections
# - Unsafe deserialization
# - File operations without validation
```

### Scenario 3: CTF Java Challenge

**Target:** Custom Java challenge JAR

```bash
jd-gui challenge.jar
# Browse class hierarchy for main class
# Read decompiled source to understand logic
# Look for hidden strings or flag patterns
# Ctrl+F → "flag" or "FLAG"
```

### Scenario 4: Library Vulnerability Check

```bash
# Identify library version
jd-gui target.jar
# Check META-INF/MANIFEST.MF for version info
# Cross-reference with CVE databases
searchsploit --cve CVE-XXXX-XXXXX
```

## 7. Detection & Defense

### How Defenders Detect

- **JAR files are trivially decompilable** — JD-GUI can read any standard JAR
- **No protection against static analysis** — obfuscation is the only defense

### Mitigation

- Use ProGuard/R8 for code obfuscation
- Encrypt/obfuscate sensitive strings
- Use native code (JNI) for critical algorithms
- Implement code signing and integrity checks

### Blue Team Response

```bash
# Quick triage
jd-gui -export suspicious.jar
grep -r "Runtime.exec\|ProcessBuilder" sources/
grep -r "socket\|Socket\|ServerSocket" sources/
```

## 8. Troubleshooting

### Common Errors

**"Could not find or load main class"**
**Fix:** Ensure Java is installed: `java -version`

**"Out of memory"**
```bash
java -Xmx2g -jar jd-gui.jar
```
**Fix:** Increase Java heap for large JAR files.

**"File format not recognized"**
**Fix:** The file may not be a valid JAR/class file. Try: `file target.jar`

**Empty decompilation output**
Some obfuscated or heavily optimized JARs may produce incomplete decompilation. Try CFR or JADX as alternatives.

### Performance Optimization

- Close unused JAR tabs to free memory
- Use search instead of manual browsing for large JARs
- Increase heap with `-Xmx2g` for analysis of large applications

### FAQ

**Q: JD-GUI vs JADX?**
A: JD-GUI is simpler and faster for JAR files. JADX has deobfuscation, Android support, and more features. Use JD-GUI for quick inspection, JADX for depth.

**Q: Can JD-GUI handle obfuscated code?**
A: JD-GUI decompiles bytecode correctly but displays obfuscated names as-is (e.g., `a.b.c()`). Use JADX with `--deobf` for better naming.

**Q: Is JD-GUI maintained?**
A: The GitHub repo receives occasional updates. It remains functional for most Java versions.

---

## Resources

- **GitHub**: https://github.com/java-decompiler/jd-gui
- **Releases**: https://github.com/java-decompiler/jd-gui/releases
- **Java Decompiler Home**: http://java-decompiler.github.io/
- **CFR (Alternative)**: https://github.com/leibnitz27/cfr
- **Procyon (Alternative)**: https://github.com/mstrobel/procyon
