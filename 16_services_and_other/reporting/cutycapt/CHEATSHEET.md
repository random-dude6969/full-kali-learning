# CuteCapt - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Basic screenshot
cutycapt --url=http://example.com --out=screenshot.png

# Full page
cutycapt --url=http://example.com --out=full.png --full-page

# With delay
cutycapt --url=http://example.com --out=loaded.png --delay=3000
```

---

## Command-Line Options

### Input/Output
| Option | Description |
|--------|-------------|
| `--url=<URL>` | URL to capture |
| `--file=<FILE>` | Local HTML file |
| `--out=<FILE>` | Output file |
| `--format=<FMT>` | PNG, SVG, PDF, JPG, BMP |

### Viewport
| Option | Description |
|--------|-------------|
| `--min-width=<PX>` | Min viewport width |
| `--min-height=<PX>` | Min viewport height |
| `--max-width=<PX>` | Max viewport width |
| `--max-height=<PX>` | Max viewport height |

### Timing
| Option | Description |
|--------|-------------|
| `--delay=<MS>` | Delay before capture |
| `--timeout=<MS>` | Page load timeout |

### Quality
| Option | Description |
|--------|-------------|
| `--jpeg-quality=<N>` | JPEG quality (0-100) |
| `--zoom=<N>` | Zoom factor |
| `--full-page` | Capture full page |

### Authentication
| Option | Description |
|--------|-------------|
| `--cookie=<NAME=VALUE>` | Add cookie |
| `--header=<NAME:VALUE>` | Add HTTP header |
| `--user-agent=<AGENT>` | Set user agent |
| `--proxy=<PROXY>` | Use HTTP proxy |

---

## Common Use Cases

```bash
# Desktop screenshot (1920x1080)
cutycapt --url=http://example.com --out=desktop.png \
    --min-width=1920 --min-height=1080

# Mobile screenshot (375x812)
cutycapt --url=http://example.com --out=mobile.png \
    --min-width=375 --min-height=812

# Login-protected page
cutycapt --url=http://target.com/admin --out=admin.png \
    --cookie="session=abc123; logged_in=true"

# Basic auth
cutycapt --url=http://user:pass@target.com --out=auth.png

# Local HTML file
cutycapt --file:///path/to/page.html --out=local.png
```

---

## Batch Script

```bash
#!/bin/bash
# batch_screenshot.sh

URLS=(
    "http://example.com"
    "http://example.com/about"
    "http://example.com/contact"
)

OUTPUT_DIR="screenshots"
mkdir -p "$OUTPUT_DIR"

for url in "${URLS[@]}"; do
    filename=$(echo "$url" | sed 's|http[s]*://||g; s|/|_|g')
    cutycapt --url="$url" --out="$OUTPUT_DIR/${filename}.png" --full-page
    echo "Captured: $url"
    sleep 1
done
```

---

## Capture from URL File

```bash
#!/bin/bash
while IFS= read -r url; do
    [[ -z "$url" || "$url" =~ ^# ]] && continue
    filename=$(echo "$url" | sed 's|http[s]*://||g; s|/|_|g')
    cutycapt --url="$url" --out="screenshots/${filename}.png" --full-page --delay=2000
done < urls.txt
```

---

## Output Formats

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

## Alternative: wkhtmltoimage

```bash
# Install
sudo apt install wkhtmltopdf wkhtmltoimage

# Basic usage
wkhtmltoimage http://example.com screenshot.png

# With options
wkhtmltoimage --quality 50 --width 1920 http://example.com screenshot.png

# Full page
wkhtmltoimage --fullpage http://example.com full.png
```

---

## Troubleshooting

```bash
# Check dependencies
ldd /usr/bin/cutycapt

# Install missing deps
sudo apt install libqtwebkit4 xvfb

# Fix blank screenshots (add delay)
cutycapt --url=http://example.com --out=fixed.png --delay=5000

# Fix timeout
cutycapt --url=http://example.com --out=timeout.png --timeout=30000

# Run with Xvfb (headless)
xvfb-run cutycapt --url=http://example.com --out=screenshot.png
```

---

## Integration Examples

```bash
# Capture and convert to PDF for report
cutycapt --url=http://example.com --out=finding.png
convert finding.png finding.pdf

# Capture with custom user agent
cutycapt --url=http://example.com --out=ua.png \
    --user-agent="Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1)"

# Capture with proxy
cutycapt --url=http://example.com --out=proxy.png \
    --proxy=http://127.0.0.1:8080
```
