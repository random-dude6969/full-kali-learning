# JD-GUI Cheatsheet

## Quick Start

```bash
jd-gui                                       # Launch GUI
jd-gui target.jar                            # Open JAR directly
jd-gui -export target.jar                    # Export sources
```

## GUI Navigation

| Action | Shortcut |
|--------|----------|
| Open file | Ctrl+O |
| Search | Ctrl+F |
| Go to class | Ctrl+G |
| Export all | File → Save All Sources |
| Close tab | Ctrl+W |

## CLI Export

```bash
jd-gui -export target.jar                    # Export to ZIP
jd-gui target.jar                            # Open and browse
```

## Common Searches

```bash
# In JD-GUI search bar:
Statement.execute                              # SQL injection
ObjectInputStream                             # Deserialization
FileInputStream                                # File operations
HttpURLConnection                              # Network calls
password                                       # Hardcoded secrets
Cipher.init                                    # Crypto operations
Runtime.exec                                   # Command execution
```

## JAR Analysis

```
target.jar/
├── META-INF/
│   └── MANIFEST.MF              # Version info
├── com/example/app/
│   ├── Main.class               # Entry point
│   ├── AuthService.class        # Authentication
│   └── DatabaseHelper.class     # Database
```

## Integration with dex2jar

```bash
# Android APK → JAR → JD-GUI
d2j-dex2jar.sh target.apk
jd-gui target-dex2jar.jar

# Or use jadx instead (better for Android)
jadx -d output/ target.apk
```

## Batch Export

```bash
#!/bin/bash
for jar in *.jar; do
    name=$(basename "$jar" .jar)
    jd-gui -export "$jar" 2>/dev/null
    mv sources.zip "exported_${name}.zip"
done
```

## Decompiler Comparison

| Tool | Strengths |
|------|-----------|
| JD-GUI | Fast, simple, reliable |
| CFR | Modern Java support |
| Procyon | Java 8+ accuracy |
| FernFlower | Readable output |
| JADX | Android, deobfuscation |

## Tips

- Fastest way to browse a JAR
- Use search for quick vulnerability scanning
- Export for grep-based analysis
- Check MANIFEST.MF for version info
- Use alongside jadx for comprehensive analysis
- Increase heap for large JARs: `java -Xmx2g -jar jd-gui.jar`
