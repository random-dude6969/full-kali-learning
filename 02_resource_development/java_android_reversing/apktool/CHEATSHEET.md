# Apktool Cheatsheet

## Quick Commands

```bash
apktool d target.apk -o output/           # Decode APK
apktool b output/ -o modified.apk         # Rebuild APK
apktool --version                          # Show version
```

## Decode

```bash
apktool d target.apk                       # Decode to target/
apktool d target.apk -o decoded/           # Decode to specific dir
apktool d -r target.apk                    # Resources only (no smali)
apktool d -s target.apk                    # Smali only (no resources)
apktool d target.apk -f                    # Force overwrite
```

## Rebuild

```bash
apktool b decoded/                         # Build from dir
apktool b decoded/ -o modified.apk         # Build with name
apktool b -f decoded/                      # Force overwrite
apktool b -d decoded/                      # Debug mode
```

## Framework Management

```bash
apktool if framework-res.apk              # Install framework
apktool if framework-res.apk -t 23        # Install with tag
apktool list-frameworks                    # List installed
```

## Signing

```bash
# Generate keystore
keytool -genkey -v -keystore key.keystore -alias key \
    -keyalg RSA -keysize 2048 -validity 10000

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
    -keystore key.keystore modified.apk key

# Or use apksigner
apksigner sign --ks key.keystore modified.apk
```

## APK Structure

```
target.apk/
├── AndroidManifest.xml     # Decoded XML
├── classes.dex             # Dalvik bytecode
├── res/                    # Decoded resources
├── lib/                    # Native libraries
├── assets/                 # Raw assets
├── META-INF/               # Signing info
├── original/               # Original files
└── apktool.yml             # Metadata
```

## Smali Quick Reference

```smali
.class public Lcom/example/MyClass;    # Class declaration
.super Landroid/app/Activity;          # Parent class

.method public onCreate(Landroid/os/Bundle;)V
    invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V
    const-string v0, "Hello"
    invoke-static {v0}, Landroid/util/Log;->d(Ljava/lang/String;)I
    return-void
.end method
```

## Security Audit Commands

```bash
# Extract permissions
grep "uses-permission" decoded/AndroidManifest.xml

# Find hardcoded URLs
grep -roh "http[s]*://[^\"']*" decoded/smali/

# Find dangerous APIs
grep -r "getDeviceId\|getSubscriberId\|exec" decoded/smali/

# Check debug mode
grep "debuggable" decoded/AndroidManifest.xml

# Check backup
grep "allowBackup" decoded/AndroidManifest.xml
```

## Flags

| Flag | Description |
|------|-------------|
| `-d`, `decode` | Decode APK |
| `-b`, `build` | Build APK |
| `-o FILE` | Output path |
| `-f`, `force` | Force overwrite |
| `-s`, `no-src` | Skip DEX |
| `-r`, `no-res` | Skip resources |
| `-d`, `debug` | Debug build |
| `-p DIR` | Framework dir |

## Tips

- Use `-r` for faster resource-only analysis
- Use `-s` for faster smali-only analysis
- Install frameworks with `apktool if` for apps referencing custom frameworks
- Always resign APKs after rebuilding
- Use alongside jadx for full analysis
