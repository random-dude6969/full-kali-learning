# CuteCapt - Web Page Screenshot Tool

## Complete Educational Guide for Cybersecurity Professionals

---

## 1. Introduction

CuteCapt is a utility designed to capture web pages and save them as various image formats (PNG, SVG, PDF, etc.). For cybersecurity professionals, CuteCapt serves as an essential tool for documenting web application vulnerabilities, capturing evidence during penetration tests, and creating visual records of web-based security findings.

### 1.1 What is CuteCapt?

CuteCapt (also known as CutyCapt) is a command-line tool that uses a headless browser (QtWebKit) to render web pages and capture screenshots. It supports multiple output formats and can capture full-page screenshots, making it invaluable for security documentation.

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Author | Jan Lieskovsky (Red Hat Security) |
| License | GPL v2 |
| Type | Command-line utility |
| Browser Engine | QtWebKit / QtWebEngine |
| Input | URLs, HTML files |
| Output | PNG, SVG, PDF, JPEG, BMP, GIF, and more |

### 1.3 Why Use CuteCapt for Cybersecurity?

- **Evidence Capture:** Document vulnerabilities with visual proof
- **Before/After Screenshots:** Compare before and after remediation
- **Full-Page Capture:** Capture entire web pages, not just viewport
- **Batch Processing:** Capture multiple pages automatically
- **Automation:** Integrate into scripts and workflows
- **Multiple Formats:** Choose the best format for your needs

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install cutycapt

# Verify installation
cutycapt --help
```

### 2.2 Ubuntu/Debian

```bash
# Install dependencies
sudo apt install libqt4-dev qt4-qmake qmake-qt4 \
    libqtwebkit-dev xvfb

# Download source
wget http://cutycapt.sourceforge.net/cutycapt-2018-07-24.tar.gz
tar -xzf cutycapt-2018-07-24.tar.gz
cd cutycapt-2018-07-24

# Build
qmake
make

# Install
sudo cp CutyCapt /usr/local/bin/
```

### 2.3 Alternative: Using wkhtmltoimage

```bash
# Install alternative tool
sudo apt install wkhtmltopdf wkhtmltoimage

# Basic usage
wkhtmltoimage --quality 50 http://example.com screenshot.png
```

---

## 3. Basic Usage

### 3.1 Command Syntax

```bash
cutycapt --url=<URL> --out=<OUTPUT_FILE> [OPTIONS]
```

### 3.2 Simple Screenshot

```bash
# Capture a web page
cutycapt --url=http://example.com --out=screenshot.png

# Capture with specific format
cutycapt --url=http://example.com --out=screenshot.svg
cutycapt --url=http://example.com --out=screenshot.pdf
```

### 3.3 Common Options

```bash
# Full page capture (not just viewport)
cutycapt --url=http://example.com --out=full.png --full-page

# Custom viewport size
cutycapt --url=http://example.com --out=wide.png --min-width=1920 --min-height=1080

# Quality settings
cutycapt --url=http://example.com --out=high.jpg --jpeg-quality=95
cutycapt --url=http://example.com --out=low.jpg --jpeg-quality=50

# Delay page load
cutycapt --url=http://example.com --out=loaded.png --delay=5000
```

---

## 4. Command-Line Options

### 4.1 Input Options

| Option | Description |
|--------|-------------|
| `--url=<URL>` | URL to capture |
| `--file=<FILE>` | Local HTML file to capture |

### 4.2 Output Options

| Option | Description |
|--------|-------------|
| `--out=<FILE>` | Output file path |
| `--format=<FMT>` | Output format (PNG, SVG, PDF, etc.) |

### 4.3 Viewport Options

| Option | Description |
|--------|-------------|
| `--min-width=<PX>` | Minimum viewport width |
| `--min-height=<PX>` | Minimum viewport height |
| `--max-width=<PX>` | Maximum viewport width |
| `--max-height=<PX>` | Maximum viewport height |

### 4.4 Timing Options

| Option | Description |
|--------|-------------|
| `--delay=<MS>` | Delay before capture (milliseconds) |
| `--timeout=<MS>` | Page load timeout (milliseconds) |

### 4.5 Quality Options

| Option | Description |
|--------|-------------|
| `--jpeg-quality=<N>` | JPEG quality (0-100) |
| `--zoom=<N>` | Zoom factor (1.0 = 100%) |

### 4.6 Other Options

| Option | Description |
|--------|-------------|
| `--full-page` | Capture entire page |
| `--header=<NAME:VALUE>` | Add HTTP header |
| `--cookie=<NAME=VALUE>` | Add cookie |
| `--javascript=<ON/OFF>` | Enable/disable JavaScript |
| `--plugins=<ON/OFF>` | Enable/disable plugins |
| `--user-agent=<AGENT>` | Set user agent string |
| `--proxy=<PROXY>` | Use HTTP proxy |

---

## 5. Output Formats

### 5.1 Supported Formats

| Format | Extension | Best For |
|--------|-----------|----------|
| PNG | .png | Screenshots, documentation |
| JPEG | .jpg | Smaller file size |
| SVG | .svg | Vector graphics, scaling |
| PDF | .pdf | Reports, sharing |
| BMP | .bmp | Legacy systems |
| GIF | .gif | Animations (not supported) |
| TIF | .tif | High quality, printing |

### 5.2 Format Selection

```bash
# PNG (default, lossless)
cutycapt --url=http://example.com --out=screenshot.png

# JPEG (compressed)
cutycapt --url=http://example.com --out=screenshot.jpg

# SVG (vector)
cutycapt --url=http://example.com --out=screenshot.svg

# PDF (document)
cutycapt --url=http://example.com --out=screenshot.pdf
```

---

## 6. Screenshot Strategies

### 6.1 Full-Page Capture

```bash
# Capture entire scrollable page
cutycapt --url=http://example.com --out=full_page.png --full-page
```

### 6.2 Specific Viewport

```bash
# Desktop viewport (1920x1080)
cutycapt --url=http://example.com --out=desktop.png \
    --min-width=1920 --min-height=1080

# Mobile viewport (375x812 - iPhone X)
cutycapt --url=http://example.com --out=mobile.png \
    --min-width=375 --min-height=812

# Tablet viewport (768x1024 - iPad)
cutycapt --url=http://example.com --out=tablet.png \
    --min-width=768 --min-height=1024
```

### 6.3 Login-Protected Pages

```bash
# With cookies
cutycapt --url=http://example.com/admin \
    --out=admin.png \
    --cookie="session=abc123; logged_in=true"

# With basic auth
cutycapt --url=http://user:pass@example.com/secure \
    --out=secure.png
```

### 6.4 Capturing Dynamic Content

```bash
# Wait for JavaScript to load
cutycapt --url=http://example.com --out=dynamic.png --delay=5000

# With JavaScript enabled
cutycapt --url=http://example.com --out=js.png --javascript=on
```

---

## 7. Batch Screenshotting

### 7.1 Basic Batch Script

```bash
#!/bin/bash
# batch_screenshot.sh

URLS=(
    "http://example.com"
    "http://example.com/about"
    "http://example.com/contact"
    "http://example.com/products"
)

OUTPUT_DIR="screenshots"
mkdir -p "$OUTPUT_DIR"

for url in "${URLS[@]}"; do
    # Generate filename from URL
    filename=$(echo "$url" | sed 's|http[s]*://||g; s|/|_|g; s|[^a-zA-Z0-9_.-]||g')
    cutycapt --url="$url" --out="$OUTPUT_DIR/${filename}.png" --full-page
    echo "Captured: $url -> $OUTPUT_DIR/${filename}.png"
    sleep 1  # Be polite
done

echo "Batch capture complete!"
```

### 7.2 Batch from File

```bash
#!/bin/bash
# capture_from_file.sh

INPUT_FILE="urls.txt"
OUTPUT_DIR="screenshots"
mkdir -p "$OUTPUT_DIR"

while IFS= read -r url; do
    # Skip empty lines and comments
    [[ -z "$url" || "$url" =~ ^# ]] && continue
    
    filename=$(echo "$url" | sed 's|http[s]*://||g; s|/|_|g; s|[^a-zA-Z0-9_.-]||g')
    cutycapt --url="$url" --out="$OUTPUT_DIR/${filename}.png" --full-page --delay=2000
    echo "Captured: $url"
done < "$INPUT_FILE"
```

### 7.3 URLs File Format

```text
# urls.txt
http://example.com
http://example.com/login
http://example.com/admin
https://target.com
https://target.com/api/docs
```

---

## 8. Integration with Pentest Workflows

### 8.1 Reconnaissance Phase

```bash
# Capture target website
cutycapt --url=http://target.com --out=target_home.png --full-page

# Capture subdomains
for sub in www mail ftp vpn; do
    cutycapt --url="http://${sub}.target.com" --out="${sub}.png" --full-page
done
```

### 8.2 Vulnerability Documentation

```bash
# Before exploitation
cutycapt --url="http://target.com/vuln?id=1" --out=vuln_before.png

# After exploitation (with evidence)
cutycapt --url="http://target.com/vuln?id=1'" --out=vuln_error.png
```

### 8.3 Reporting Phase

```bash
# Capture all findings for report
while IFS= read -r url; do
    cutycapt --url="$url" --out="report_images/$(basename $url).png" --full-page
done < finding_urls.txt
```

---

## 9. URL Handling

### 9.1 HTTP/HTTPS

```bash
# HTTP
cutycapt --url=http://example.com --out=site.png

# HTTPS
cutycapt --url=https://example.com --out=site.png

# Auto-detect (use HTTPS if available)
cutycapt --url=example.com --out=site.png
```

### 9.2 Local Files

```bash
# Local HTML file
cutycapt --file:///path/to/local.html --out=local.png

# File with query string
cutycapt --file:///path/to/page.html?id=1 --out=local.png
```

### 9.3 Authentication

```bash
# Basic authentication
cutycapt --url=http://user:password@target.com --out=auth.png

# Cookie-based authentication
cutycapt --url=http://target.com/admin \
    --out=admin.png \
    --cookie="session_id=abc123; user=admin"
```

---

## 10. Timeouts and Page Load

### 10.1 Setting Timeouts

```bash
# Short timeout (5 seconds)
cutycapt --url=http://example.com --out=quick.png --timeout=5000

# Long timeout (30 seconds)
cutycapt --url=http://example.com --out=slow.png --timeout=30000
```

### 10.2 Delay Settings

```bash
# Wait 2 seconds for JavaScript
cutycapt --url=http://example.com --out=js.png --delay=2000

# Wait 10 seconds for heavy pages
cutycapt --url=http://example.com --out=heavy.png --delay=10000
```

---

## 11. Scripting with CuteCapt

### 11.1 Automated Screenshot Script

```bash
#!/bin/bash
# auto_screenshot.sh

URL=$1
OUTPUT=$2
DELAY=${3:-3000}

if [ -z "$URL" ] || [ -z "$OUTPUT" ]; then
    echo "Usage: $0 <URL> <OUTPUT> [DELAY_MS]"
    exit 1
fi

cutycapt \
    --url="$URL" \
    --out="$OUTPUT" \
    --full-page \
    --delay="$DELAY" \
    --min-width=1920 \
    --min-height=1080 \
    --javascript=on \
    --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

echo "Screenshot saved: $OUTPUT"
```

### 11.2 Parallel Capture Script

```bash
#!/bin/bash
# parallel_capture.sh

URLS_FILE=$1
MAX_PARALLEL=5

cat "$URLS_FILE" | xargs -P $MAX_PARALLEL -I {} bash -c '
    url={}
    filename=$(echo "$url" | sed "s|http[s]*://||g; s|/|_|g")
    cutycapt --url="$url" --out="screenshots/${filename}.png" --full-page
    echo "Captured: $url"
'
```

### 11.3定时截图脚本

```bash
#!/bin/bash
# monitor_screenshot.sh

URL="http://target.com"
OUTPUT_DIR="monitor_$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

while true; do
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    cutycapt --url="$URL" --out="$OUTPUT_DIR/${TIMESTAMP}.png" --full-page
    echo "Captured at $TIMESTAMP"
    sleep 3600  # Every hour
done
```

---

## 12. Comparison with Alternatives

### 12.1 CuteCapt vs wkhtmltoimage

| Feature | CuteCapt | wkhtmltoimage |
|---------|----------|---------------|
| Browser Engine | QtWebKit | WebKit |
| Full Page | Yes | Yes |
| Quality | Good | Better |
| Speed | Medium | Faster |
| Dependencies | Qt | Minimal |
| Active Development | No | Yes |

### 12.2 CuteCapt vs PhantomJS

| Feature | CuteCapt | PhantomJS |
|---------|----------|-----------|
| Scripting | Limited | Full JS |
| Flexibility | Medium | High |
| Ease of Use | Easier | Complex |
| Maintenance | Deprecated | Deprecated |

### 12.3 Modern Alternatives

```bash
# Puppeteer (Node.js)
npx puppeteer screenshot http://example.com screenshot.png

# Playwright (Node.js/Python)
npx playwright screenshot http://example.com screenshot.png

# GoScreenshot (Go)
gScreenshot http://example.com screenshot.png
```

---

## 13. Combining with Reporting Tools

### 13.1 CherryTree Integration

```bash
# Capture and import to CherryTree
cutycapt --url=http://example.com --out=screenshot.png
# Then drag screenshot.png into CherryTree node
```

### 13.2 Markdown Reports

```bash
# Generate Markdown with embedded images
cutycapt --url=http://example.com --out=finding1.png
echo "![Finding 1](finding1.png)" >> report.md
```

### 13.3 PDF Reports

```bash
# Capture directly as PDF
cutycapt --url=http://example.com --out=finding.pdf

# Or convert PNG to PDF
convert finding.png finding.pdf
```

---

## 14. Best Practices

### 14.1 Naming Conventions

```
screenshots/
├── 2024-01-15_target.com_home.png
├── 2024-01-15_target.com_login.png
├── 2024-01-15_target.com_admin.png
├── 2024-01-15_vuln_sqli.png
└── 2024-01-15_vuln_xss.png
```

### 14.2 Documentation Workflow

1. Capture screenshots during the assessment
2. Use consistent naming conventions
3. Add timestamps for audit trail
4. Include context in filenames
5. Store in organized directory structure

### 14.3 Quality Settings

```bash
# High quality for reports
cutycapt --url=http://example.com --out=report.png --jpeg-quality=100

# Medium quality for email
cutycapt --url=http://example.com --out=email.png --jpeg-quality=75

# Low quality for quick preview
cutycapt --url=http://example.com --out=preview.png --jpeg-quality=50
```

---

## 15. Troubleshooting

### Common Issues

**Problem:** CuteCapt won't start

```bash
# Check dependencies
ldd /usr/bin/cutycapt

# Install missing dependencies
sudo apt install libqtwebkit4 xvfb
```

**Problem:** Blank screenshots

```bash
# Add delay for JavaScript pages
cutycapt --url=http://example.com --out=blank.png --delay=5000

# Enable JavaScript
cutycapt --url=http://example.com --out=js.png --javascript=on
```

**Problem:** Partial page capture

```bash
# Use full-page option
cutycapt --url=http://example.com --out=full.png --full-page

# Increase viewport size
cutycapt --url=http://example.com --out=large.png --min-width=1920
```

**Problem:** Timeout errors

```bash
# Increase timeout
cutycapt --url=http://example.com --out=timeout.png --timeout=30000
```

---

## References

- [CuteCapt Documentation](http://cutycapt.sourceforge.net/)
- [CuteCapt Manual](http://cutycapt.sourceforge.net/cutycapt-manual.html)
- [Kali Linux CuteCapt Package](https://www.kali.org/tools/cutycapt/)
- [wkhtmltoimage Documentation](https://wkhtmltopdf.org/)
