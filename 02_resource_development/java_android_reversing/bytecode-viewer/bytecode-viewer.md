---
tool_name: bytecode-viewer
display_name: Bytecode Viewer - Java/Android Reverse Engineering Suite
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: java_android_reversing
install_command: "sudo apt install bytecode-viewer"
version: "2.13.2"
github_repo: "https://github.com/Konloch/bytecode-viewer"
kali_tools_page: "https://www.kali.org/tools/bytecode-viewer/"
difficulty_level: beginner
requires_root: false
gui_available: true
tags: [java, android, decompiler, bytecode, reverse-engineering, multi-decompiler]
related_tools: [jadx, jd-gui, dex2jar, apktool]
---

# Bytecode Viewer - Java/Android Reverse Engineering Suite

## 1. Introduction & Overview

Bytecode Viewer (BCV) is a comprehensive, all-in-one Java and Android reverse engineering suite. Unlike single-purpose decompilers, BCV bundles six Java decompilers, three bytecode disassemblers, two APK converters, a Java compiler, and a plugin system into a single GUI application. It's designed to let you compare decompiler output side-by-side and edit bytecode in place.

**Created:** 2015 by Konloch
**License:** GPL v3.0
**GitHub Stars:** 15,600+

### What It Does

- Decompiles Java JAR, class files, Android APK/DEX/XAPK/APKM files
- Side-by-side comparison of 6 different decompilers
- Edit and recompile Java and bytecode directly
- Export as runnable JAR, ZIP, or APK
- Plugin system with Java and JavaScript scripting
- Malicious code scanning API
- Supports 30+ languages in the UI

### Supported File Formats

| Format | Description |
|--------|-------------|
| `.class` | Java compiled class |
| `.jar` | Java Archive |
| `.apk` | Android Package |
| `.dex` | Dalvik Executable |
| `.xapk` | Split APK bundle |
| `.apkm` | APK Mirror bundle |
| `.war` | Web Application Archive |
| `.jsp` | JavaServer Pages |

### When to Use

- **Quick decompilation**: Drag-and-drop a JAR or APK for instant decompilation
- **Decompiler comparison**: Compare output from 6 decompilers to find the most readable result
- **Java bytecode editing**: Modify bytecode without full decompilation
- **Educational**: Learn how different decompilers interpret the same bytecode
- **Quick triage**: Fast initial assessment of Java/Android applications

### When NOT to Use

- Deep Android resource analysis (use Apktool instead)
- Batch processing hundreds of files (use jadx CLI)
- High-performance decompilation of massive JARs (use dedicated CLI tools)

### Legal & Ethical

BCV is for authorized security research and educational purposes. Modifying and redistributing commercial software may violate copyright laws.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Decompiler** | Converts bytecode to Java source code |
| **Disassembler** | Converts bytecode to readable assembly |
| **Smali/BakSmali** | Dalvik bytecode assembly/disassembly |
| **Krakatau** | Java bytecode assembler/disassembler |
| **Enjarify** | Google's DEX-to-JAR converter |

## 2. Installation & Setup

### System Requirements

- Java 8+ (JRE)
- 4+ GB RAM recommended
- Display server (X11/Wayland) for GUI

### Installation

```bash
sudo apt install bytecode-viewer
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  bytecode-viewer
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 145 MB of archives.
After this operation, 280 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 bytecode-viewer all 2.13.2-0kali1 [145 MB]
Fetched 145 MB in 5s (28.2 MB/s)
Selecting previously unselected package bytecode-viewer.
Preparing to unpack .../bytecode-viewer_2.13.2-0kali1_all.deb ...
Unpacking bytecode-viewer (2.13.2-0kali1) ...
Setting up bytecode-viewer (2.13.2-0kali1) ...
```

### From GitHub Releases (Alternative)

```bash
wget https://github.com/Konloch/bytecode-viewer/releases/download/v2.13.2/Bytecode-Viewer-2.13.2.jar
java -jar Bytecode-Viewer-2.13.2.jar
```

### Building from Source

```bash
git clone https://github.com/Konloch/bytecode-viewer.git
cd bytecode-viewer
mvn package
# Run: java -jar target/Bytecode-Viewer-*.jar
```

### Verification

```bash
java -jar /usr/share/bytecode-viewer/bytecode-viewer.jar -help 2>/dev/null | head -5
```

## 3. Basic Usage

### Launching the GUI

```bash
bytecode-viewer
# Or:
java -jar /usr/share/bytecode-viewer/bytecode-viewer.jar
```

**Explanation:** Opens the BCV GUI. Drag-and-drop files into the window to begin decompilation.

### Command-Line Decompilation

```bash
java -jar bytecode-viewer.jar -decompiler procyon -i target.jar -o output.java
```

**Explanation:** Decompile using Procyon decompiler from command line.

### Command-Line Flags

| Flag | Description |
|------|-------------|
| `-help` | Display help menu |
| `-clean` | Delete BCV directory |
| `-english` | Force English UI |
| `-list` | List available decompilers |
| `-decompiler NAME` | Select decompiler (procyon, cfr, fernflower, jadx, jd-gui, krakatau) |
| `-i FILE` | Input file (JAR, class, APK, DEX) |
| `-o FILE` | Output file (Java or bytecode) |
| `-t CLASS` | Target class name or "all" for full decompile |
| `-nowait` | Don't wait for user input |

### GUI Workflow

1. **Drag-and-drop** a file into BCV
2. **Select decompiler** from View Pane menu (View 1, View 2, View 3)
3. **Navigate** the resource tree on the left
4. **Compare** decompiler output side-by-side
5. **Edit** Java/bytecode inline (if enabled)
6. **Export** as runnable JAR, ZIP, or APK

### Built-in Decompilers

| Decompiler | Strengths |
|------------|-----------|
| **CFR** | Modern, good annotation handling |
| **Procyon** | High accuracy, good for Java 8+ |
| **FernFlower** | IntelliJ's decompiler, very readable output |
| **JADX** | Android-focused, excellent for APK |
| **JD-GUI** | Classic, fast, reliable |
| **Krakatau** | Bytecode-focused, handles edge cases |

## 4. Intermediate Usage

### Side-by-Side Comparison

Open the same class in multiple decompiler views to identify the most readable output. Some decompilers handle specific bytecode patterns better than others.

### Java Editing

```java
// BCV allows direct Java editing (with compilation)
// Modify the decompiled code and recompile
public class TargetClass {
    public static void main(String[] args) {
        System.out.println("Modified!");
    }
}
```

**Explanation:** BCV's built-in compiler can recompile modified Java source back to bytecode.

### Bytecode Editing

BCV provides a bytecode editor for direct instruction manipulation when Java source is unavailable or too complex.

### Plugin System

BCV supports Java and JavaScript plugins:

```javascript
// Example: Find all string constants
var classes = api.getClasses();
for (var i = 0; i < classes.size(); i++) {
    var cls = classes.get(i);
    api.log("Class: " + cls.name);
}
```

**Explanation:** Plugins can programmatically analyze all loaded classes using BCV's API.

### Exporting

```bash
# Command-line export
java -jar bytecode-viewer.jar -i input.jar -o output_dir/ -t all -decompiler procyon
```

**Explanation:** Batch export all classes from a JAR file using Procyon decompiler.

### Handling Large Files

```bash
# Increase Java heap for large JARs
java -Xmx4G -jar bytecode-viewer.jar
```

**Explanation:** Large JARs (100+ MB) require increased memory allocation.

## 5. Advanced Usage

### Malicious Code Scanning

BCV includes an API for writing malicious code scanners:

```java
// Pseudo-code for a custom scanner
public class MaliciousCodeScanner implements BCVPlugin {
    public void execute(API api) {
        for (ClassNode cls : api.getClasses()) {
            for (MethodNode method : cls.methods) {
                // Check for Runtime.exec calls
                if (method.instructions.contains("java/lang/Runtime.exec")) {
                    api.log("SUSPICIOUS: " + cls.name + "." + method.name);
                }
            }
        }
    }
}
```

**Explanation:** Build custom scanners to detect malware patterns across loaded classes.

### Batch Processing

```bash
#!/bin/bash
for jar in *.jar; do
    name=$(basename "$jar" .jar)
    java -jar bytecode-viewer.jar -i "$jar" -o "output/$name/" -t all -decompiler cfr -nowait
done
```

**Explanation:** Automates decompilation of multiple JAR files.

### Comparing Decompiler Accuracy

```bash
#!/bin/bash
# Compare all decompilers on a single class
for decompiler in procyon cfr fernflower jadx jd-gui krakatau; do
    java -jar bytecode-viewer.jar -decompiler $decompiler -i target.jar -o "output_${decompiler}/" -t "com.example.Main" -nowait
done
```

**Explanation:** Generates decompiled output from all available decompilers for comparison.

## 6. Real-World Scenarios

### Scenario 1: Java Application Security Audit

**Target:** Enterprise Java application (JAR)

```bash
# Open in BCV
java -jar bytecode-viewer.jar app.jar

# Look for SQL injection patterns
# In the search panel, search for: Statement.execute, PreparedStatement

# Look for hardcoded credentials
# Search for: password, secret, apiKey, Connection

# Look for insecure deserialization
# Search for: ObjectInputStream, readObject, deserialize
```

### Scenario 2: Android APK Triage

```bash
# Convert APK to DEX first (or open APK directly in BCV)
# BCV uses Dex2Jar/Enjarify internally
java -jar bytecode-viewer.jar suspicious.apk

# Browse decompiled code for:
# - C2 URLs
# - Data collection APIs
# - Encryption routines
```

### Scenario 3: Java Malware Analysis

**Target:** Malicious JAR file

```bash
java -jar bytecode-viewer.jar malware.jar

# Search for:
# - Process execution: Runtime.exec, ProcessBuilder
# - Network: HttpURLConnection, Socket, URL
# - File operations: FileOutputStream, RandomAccessFile
# - Reflection: Class.forName, Method.invoke
```

### Scenario 4: Code Comparison

**Context:** Compare original and obfuscated versions

```bash
# Decompile both versions
java -jar bytecode-viewer.jar -decompiler cfr -i original.jar -o orig/ -t all -nowait
java -jar bytecode-viewer.jar -decompiler cfr -i obfuscated.jar -o obf/ -t all -nowait

# Diff the output
diff -r orig/ obf/
```

## 7. Detection & Defense

### How Defenders Detect

- **File analysis**: Java JARs/APKs are easily decompiled
- **String extraction**: Hardcoded values are trivially extracted
- **API analysis**: Dangerous APIs are detectable through static analysis

### Mitigation (For Developers)

- Use ProGuard/R8 for obfuscation
- Implement runtime integrity checks
- Encrypt sensitive strings at build time
- Use native code for critical algorithms

### Blue Team Response

```bash
# Quick triage of suspicious JAR
java -jar bytecode-viewer.jar -list  # Check available decompilers
java -jar bytecode-viewer.jar -decompiler cfr -i suspect.jar -o analysis/ -t all
grep -r "Runtime.exec\|ProcessBuilder" analysis/
```

## 8. Troubleshooting

### Common Errors

**"java.lang.OutOfMemoryError"**
```bash
java -Xmx4G -jar bytecode-viewer.jar
```
**Fix:** Increase Java heap size for large files.

**"File permission denied"**
Run as administrator or change file permissions: `chmod +r target.jar`

**UI lag**
Change theme: View → Visual Settings → Window Theme → System Theme

**APK loading fails**
Ensure the APK is not corrupted: `file target.apk && unzip -t target.apk`

### Performance Optimization

- Use the System Theme for better UI performance
- Increase heap with `-Xmx4G` or higher
- Use command-line mode for batch operations
- Close other Java applications to free memory

### FAQ

**Q: BCV vs JADX?**
A: BCV is an all-in-one suite with 6 decompilers for comparison. JADX is a single, highly optimized decompiler with better accuracy. Use BCV for exploration, JADX for depth.

**Q: Can BCV recompile modified Java?**
A: Yes, BCV includes a built-in Java compiler for editing and recompiling class files.

**Q: Which decompiler should I use?**
A: Start with Procyon or CFR for general Java. Use JADX for Android-specific code. Compare multiple decompilers when output seems incorrect.

---

## Resources

- **GitHub**: https://github.com/Konloch/bytecode-viewer
- **Website**: https://bytecodeviewer.com
- **Releases**: https://github.com/Konloch/bytecode-viewer/releases
- **Discord**: https://discord.gg/aexsYpfMEf
- **The Bytecode Club**: https://the.bytecode.club
