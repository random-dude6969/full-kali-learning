---
tool_name: "wpprobe"
display_name: "WPProbe"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "web_scanning"
install_command: "go install github.com/chokeenworwor/wpprobe@latest"
version: "1.0"
github_repo: "https://github.com/chokeenworwor/wpprobe"
kali_tools_page: "https://www.kali.org/tools/wpprobe/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["wordpress", "plugin", "theme", "scanner", "enumeration", "detection"]
related_tools: ["wpscan", "whatweb", "wappalyzer", "nuclei"]
---

# WPProbe

## Chapter 1: Introduction & Overview

### What is WPProbe?
WPProbe is a WordPress plugin and theme scanner that detects installed plugins on a WordPress site by probing known plugin file paths. It uses a wordlist-based approach, checking for the existence of `readme.txt`, `style.css`, and `changelog.txt` files in `/wp-content/plugins/{plugin-name}/` directories. When a path returns HTTP 200, the plugin is confirmed as installed.

### History & Background
- **Created:** 2022 by chokeenworwor
- **License:** MIT License
- **Language:** Go
- **Purpose:** Fast WordPress plugin enumeration
- **GitHub Stars:** 200+
- **Performance:** Concurrent scanning with configurable thread count

### When to Use This Tool
- **WordPress reconnaissance:** Quickly identify installed plugins
- **Vulnerability assessment:** Map plugin attack surface before vulnerability checking
- **Penetration testing:** Enumerate plugins as part of WordPress security assessment
- **Bug bounty:** Find custom or lesser-known plugins that WPScan may miss
- **CTF competitions:** Identify WordPress plugins in challenges

### Legal and Ethical Considerations
- Scans publicly accessible plugin paths (HTTP requests)
- Semi-intrusive — makes many requests to target
- Use only on authorized targets
- May trigger WAF/IDS alerts with high thread counts

### Key Concepts
- **Plugin Enumeration:** Systematically discovering installed WordPress plugins
- **Wordlist-based Detection:** Probing known plugin paths against a wordlist
- **HTTP Status Code:** 200 = plugin exists, 404 = not found, 403 = forbidden
- **readme.txt:** Standard WordPress plugin file containing version info
- **style.css:** Theme file containing theme metadata (author, version, description)

---

## Chapter 2: Installation & Setup

### System Requirements
- Go 1.18+ (for `go install`)
- Internet access
- No special permissions needed

### Installation via Go
```bash
# Install Go (if not already installed)
sudo apt update
sudo apt install golang-go

# Install WPProbe
go install github.com/chokeenworwor/wpprobe@latest
```

**Sample output:**
```
go: downloading github.com/chokeenworwor/wpprobe v1.0.0
go: downloading github.com/valyala/fasthttp v1.44.0
go: downloading github.com/valyala/bytebufferpool v1.0.1
go: installing github.com/chokeenworwor/wpprobe@latest
```

### Binary Download
```bash
# Download latest release from GitHub
wget https://github.com/chokeenworwor/wpprobe/releases/download/v1.0/wpprobe_linux_amd64
chmod +x wpprobe_linux_amd64
sudo mv wpprobe_linux_amd64 /usr/local/bin/wpprobe
```

### From Source
```bash
git clone https://github.com/chokeenworwor/wpprobe.git
cd wpprobe
go build -o wpprobe .
sudo mv wpprobe /usr/local/bin/
```

### Verification
```bash
wpprobe --help
```
**Output:**
```
WPProbe - WordPress Plugin Scanner

Usage:
  wpprobe [flags]

Flags:
  -u, --url string       Target URL
  -t, --threads int      Number of threads (default 10)
  -w, --wordlist string  Plugin wordlist file
  -o, --output string    Output file
  -v, --verbose          Verbose output
  -h, --help             Help
```

### Default Wordlist Location
```bash
# WPProbe includes a built-in wordlist
# To use custom wordlist:
wpprobe -u https://target.com -w /path/to/custom_wordlist.txt
```

---

## Chapter 3: Basic Usage

### First Run
```bash
wpprobe -u https://target.com
```
**Sample output:**
```
[+] WPProbe - WordPress Plugin Scanner
[+] Target: https://target.com
[+] Threads: 10
[+] Loading wordlist...
[+] Scanning plugins...

[+] Found plugins:
 | https://target.com/wp-content/plugins/akismet/readme.txt (200)
 | https://target.com/wp-content/plugins/contact-form-7/readme.txt (200)
 | https://target.com/wp-content/plugins/jetpack/readme.txt (200)
 | https://target.com/wp-content/plugins/woocommerce/readme.txt (200)
 | https://target.com/wp-content/plugins/elementor/readme.txt (200)
 | https://target.com/wp-content/plugins/wordfence/readme.txt (200)

[+] Scan complete
[+] Plugins found: 6
[+] Requests made: 1500
[+] Time elapsed: 45s
```

### Verbose Mode
```bash
wpprobe -v -u https://target.com
```
**Output:**
```
[+] WPProbe - WordPress Plugin Scanner
[+] Target: https://target.com
[+] Threads: 10
[+] Wordlist: built-in (1000 entries)
[+] Starting scan...

[+] Probing: akismet -> 200 OK
[+] Probing: jetpack -> 200 OK
[+] Probing: contact-form-7 -> 200 OK
[~] Probing: wordfence -> 404 Not Found
[+] Probing: woocommerce -> 200 OK
[~] Probing: yoast-seo -> 404 Not Found
[+] Probing: elementor -> 200 OK
[+] Probing: wordfence -> 200 OK

[+] Found 6 plugins
```

### Essential Commands

#### Basic Plugin Scan
```bash
wpprobe -u https://target.com
```
**Explanation:** Scans for WordPress plugins using the built-in wordlist.

#### Custom Thread Count
```bash
wpprobe -t 50 -u https://target.com
```
**Explanation:** Uses 50 threads for faster scanning (may trigger rate limiting).

#### Custom Wordlist
```bash
wpprobe -w custom_plugins.txt -u https://target.com
```
**Explanation:** Scans using a custom plugin wordlist.

#### Save Results
```bash
wpprobe -o results.txt -u https://target.com
```
**Explanation:** Saves scan results to a file.

### Complete Flags Reference

#### Target Specification
| Flag | Description | Example |
|------|-------------|---------|
| `-u`, `--url` | Target URL (required) | `wpprobe -u https://target.com` |

#### Scan Options
| Flag | Description | Example |
|------|-------------|---------|
| `-t`, `--threads` | Thread count (default 10) | `wpprobe -t 20 -u https://target.com` |
| `-w`, `--wordlist` | Custom plugin wordlist | `wpprobe -w plugins.txt -u https://target.com` |
| `-v`, `--verbose` | Verbose output | `wpprobe -v -u https://target.com` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-o`, `--output` | Output file | `wpprobe -o results.txt -u https://target.com` |

---

## Chapter 4: Intermediate Usage

### Bash Automation
```bash
#!/bin/bash
# Multi-site WordPress plugin scanner
SITES=("https://site1.com" "https://site2.com" "https://site3.com")
REPORT_DIR="wpprobe_$(date +%Y%m%d)"
mkdir -p "$REPORT_DIR"

for site in "${SITES[@]}"; do
    domain=$(echo "$site" | sed 's|https\?://||;s|/.*||')
    echo "[*] Scanning: $site"
    
    wpprobe -t 20 -o "${REPORT_DIR}/${domain}.txt" "$site" 2>/dev/null
    
    # Count plugins found
    PLUGINS=$(grep -c "readme.txt" "${REPORT_DIR}/${domain}.txt" 2>/dev/null || echo "0")
    echo "[+] Found $PLUGINS plugins on $domain"
    echo ""
done

echo "[+] Scan complete. Reports in: $REPORT_DIR"
```

### Tool Chaining
```bash
# WPProbe → WPScan for vulnerability checking
wpprobe -u https://target.com -o plugins.txt
# Extract plugin names
PLUGINS=$(grep -oP 'plugins/\K[^/]+(?=/readme)' plugins.txt)
# Run WPScan with discovered plugins
wpscan --url https://target.com -e vp --api-token TOKEN

# WPProbe → Nuclei for specific CVE detection
wpprobe -u https://target.com -o plugins.txt
grep "readme.txt" plugins.txt | awk -F'/' '{print $(NF-1)}' | \
    while read -r plugin; do
        echo "[*] Checking $plugin for vulnerabilities..."
        nuclei -u "https://target.com/wp-content/plugins/${plugin}/" \
            -t /usr/share/nuclei-templates/http/technologies/wordpress/
    done

# WPProbe → WhatWeb for tech stack
WHATWEB_OUT=$(whatweb -f json https://target.com | jq -r '.[0].plugins | keys[]')
if echo "$WHATWEB_OUT" | grep -qi "wordpress"; then
    echo "[+] WordPress detected, running WPProbe..."
    wpprobe -u https://target.com
fi
```

### Python Integration
```python
import subprocess
import re

def wpprobe_scan(url, threads=10, wordlist=None):
    cmd = ["wpprobe", "-u", url, "-t", str(threads)]
    if wordlist:
        cmd.extend(["-w", wordlist])
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout

def parse_plugins(output):
    plugins = []
    for line in output.split('\n'):
        if 'readme.txt' in line and '200' in line:
            match = re.search(r'plugins/([^/]+)/readme', line)
            if match:
                plugins.append(match.group(1))
    return plugins

# Scan and extract plugins
output = wpprobe_scan("https://target.com", threads=20)
plugins = parse_plugins(output)

print(f"[+] Found {len(plugins)} plugins:")
for plugin in plugins:
    print(f"  - {plugin}")
```

### Custom Wordlist Creation
```bash
# Create a custom plugin wordlist from WPScan's database
# Download WPScan's plugin list
curl -s "https://raw.githubusercontent.com/wpscanteam/wpscan/master/data/plugins.txt" > wp_plugins.txt

# Or create manually
cat > custom_plugins.txt << EOF
akismet
jetpack
woocommerce
elementor
wordfence
contact-form-7
yoast-seo
advanced-custom-fields
wpforms-lite
classic-editor
ewww-image-optimizer
redirection
really-simple-ssl
updraftplus
google-site-kit
EOF

# Use custom wordlist
wpprobe -w custom_plugins.txt -u https://target.com
```

### Batch Processing
```bash
#!/bin/bash
# Batch WordPress plugin scan
INPUT_FILE="wordpress_targets.txt"
OUTPUT_DIR="batch_wpprobe"
mkdir -p "$OUTPUT_DIR"

cat "$INPUT_FILE" | while read -r url; do
    domain=$(echo "$url" | sed 's|https\?://||;s|/.*||')
    echo "[*] Scanning: $url"
    
    wpprobe -t 20 -o "${OUTPUT_DIR}/${domain}.txt" "$url" 2>/dev/null
    
    # Extract plugin list
    PLUGINS=$(grep -oP 'plugins/\K[^/]+(?=/readme)' "${OUTPUT_DIR}/${domain}.txt" 2>/dev/null)
    
    if [[ -n "$PLUGINS" ]]; then
        echo "[+] Plugins found:"
        echo "$PLUGINS" | while read -r p; do echo "    - $p"; done
    else
        echo "[!] No plugins found"
    fi
    echo ""
done
```

---

## Chapter 5: Advanced Usage

### Detection Methods
WPProbe uses multiple file patterns for detection:
```bash
# Default detection paths
/target.com/wp-content/plugins/{plugin}/readme.txt
/target.com/wp-content/plugins/{plugin}/style.css
/target.com/wp-content/plugins/{plugin}/changelog.txt
/target.com/wp-content/plugins/{plugin}/plugin.php
```

### Custom Detection Patterns
```bash
# Create custom detection script
#!/bin/bash
PLUGIN=$1
TARGET=$2

# Check multiple paths
for file in readme.txt style.css changelog.txt plugin.php index.php; do
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
        "${TARGET}/wp-content/plugins/${PLUGIN}/${file}")
    if [[ "$HTTP_CODE" == "200" ]]; then
        echo "[+] ${PLUGIN}/${file} found (HTTP ${HTTP_CODE})"
    fi
done
```

### Evasion Techniques
```bash
# Reduce thread count to avoid rate limiting
wpprobe -t 5 -u https://target.com

# Use custom User-Agent
curl -s -H "User-Agent: Mozilla/5.0" \
    "https://target.com/wp-content/plugins/akismet/readme.txt"

# Delay between requests
for plugin in $(cat plugins.txt); do
    curl -s "https://target.com/wp-content/plugins/${plugin}/readme.txt" -o /dev/null
    sleep 1
done
```

### Workflow Integration
```bash
#!/bin/bash
# Full WordPress reconnaissance pipeline
TARGET=$1

echo "[+] Phase 1: Technology Detection"
WHATWEB=$(whatweb -f json "$TARGET" 2>/dev/null)
if echo "$WHATWEB" | grep -qi "wordpress"; then
    echo "[+] WordPress detected"
else
    echo "[!] Not WordPress, exiting"
    exit 1
fi

echo "[+] Phase 2: Plugin Enumeration (WPProbe)"
wpprobe -t 20 -o wpprobe_results.txt "$TARGET"

echo "[+] Phase 3: Vulnerability Scanning (WPScan)"
PLUGINS=$(grep -oP 'plugins/\K[^/]+(?=/readme)' wpprobe_results.txt 2>/dev/null)
if [[ -n "$PLUGINS" ]]; then
    echo "[+] Found plugins: $PLUGINS"
    wpscan --url "$TARGET" -e vp --api-token TOKEN -o wpscan_vulns.json
else
    echo "[!] No plugins found"
fi

echo "[+] Phase 4: User Enumeration"
wpscan --url "$TARGET" -e u --api-token TOKEN -o wpscan_users.json

echo "[+] Pipeline complete"
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: WordPress Security Audit — Plugin Discovery
**Objective:** Identify all plugins installed on a WordPress site for security assessment.

```bash
# Step 1: Confirm WordPress is running
curl -s https://target.com | grep -i "wp-content"
# Confirmed: WordPress detected

# Step 2: Run WPProbe with high thread count
wpprobe -t 50 -v -o wpprobe_results.txt https://target.com
# Output reveals 15 plugins installed

# Step 3: Extract plugin names
PLUGINS=$(grep -oP 'plugins/\K[^/]+(?=/readme)' wpprobe_results.txt)
echo "[+] Plugins found:"
echo "$PLUGINS"

# Step 4: Cross-reference with WPScan
wpscan --url https://target.com -e vp --api-token TOKEN -o wpscan_vulns.json

# Step 5: Check for outdated plugins
for plugin in $PLUGINS; do
    echo "[*] Checking: $plugin"
    curl -s "https://target.com/wp-content/plugins/${plugin}/readme.txt" | \
        grep -i "stable tag" | head -1
done
```

### Scenario 2: CTF Challenge — Find the Vulnerable Plugin
**Objective:** Identify a vulnerable WordPress plugin in a CTF challenge.

```bash
# Step 1: Basic WPProbe scan
wpprobe -u http://ctf-target.com
# Output reveals: flavor (custom plugin)

# Step 2: Investigate the custom plugin
curl -s http://ctf-target.com/wp-content/plugins/flavor/readme.txt
# Output: Flavor Plugin v1.0 - Custom file manager

# Step 3: Test for file upload vulnerability
curl -s http://ctf-target.com/wp-content/plugins/flavor/upload.php
# Output: File upload form found

# Step 4: Upload a test file
echo "<?php echo 'FLAG{flavor_plugin_exploited}'; ?>" > shell.php
curl -s -F "file=@shell.php" \
    -F "submit=Upload" \
    http://ctf-target.com/wp-content/plugins/flavor/upload.php

# Step 5: Access uploaded file
curl -s http://ctf-target.com/wp-content/uploads/shell.php
```

### Scenario 3: Bug Bounty — Plugin Enumeration for Attack Surface
**Objective:** Map all plugins on a bug bounty WordPress target.

```bash
# Step 1: Quick WPProbe scan
wpprobe -t 30 -o plugins.txt https://bounty-target.com

# Step 2: Extract and categorize plugins
echo "=== Plugin Analysis ==="
while read -r line; do
    if [[ "$line" == *"readme.txt"* && "$line" == *"200"* ]]; then
        PLUGIN=$(echo "$line" | grep -oP 'plugins/\K[^/]+')
        
        # Check readme for version
        VERSION=$(curl -s "https://bounty-target.com/wp-content/plugins/${PLUGIN}/readme.txt" | \
            grep -i "stable tag" | awk '{print $3}' | head -1)
        
        echo "[+] ${PLUGIN} (version: ${VERSION:-unknown})"
        
        # Check for known vulnerabilities
        searchsploit "$PLUGIN" 2>/dev/null | head -3
    fi
done < plugins.txt

# Step 3: Test each plugin for vulnerabilities
grep "readme.txt" plugins.txt | awk -F'/' '{print $(NF-1)}' | while read -r plugin; do
    echo "[*] Testing: $plugin"
    
    # Check for common vulnerable files
    for file in upload.php admin.php settings.php options.php config.php; do
        HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
            "https://bounty-target.com/wp-content/plugins/${plugin}/${file}")
        if [[ "$HTTP_CODE" != "404" ]]; then
            echo "  [!] ${file} found (HTTP ${HTTP_CODE})"
        fi
    done
done
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect WPProbe

#### Request Pattern Detection
WPProbe makes sequential requests to `/wp-content/plugins/{plugin}/readme.txt`:
```
GET /wp-content/plugins/akismet/readme.txt 200
GET /wp-content/plugins/jetpack/readme.txt 200
GET /wp-content/plugins/contact-form-7/readme.txt 200
GET /wp-content/plugins/woocommerce/readme.txt 200
...
```
This pattern is easily detectable.

#### Log Analysis
```bash
# Detect WPProbe in access logs
grep "wp-content/plugins/" access.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Find readme.txt probes
grep "readme.txt" access.log | \
    awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

### IDS/IPS Signatures

#### Snort Rule
```
alert http any any -> any any (msg:"WPProbe Plugin Enumeration"; \
  content:"/wp-content/plugins/"; http_uri; \
  content:"readme.txt"; http_uri; \
  flow:to_server,established; \
  threshold:type both, track by_src, count 20, seconds 60; \
  classtype:web-application-attack; \
  sid:1000030; rev:1;)
```

#### Suricata Rule
```
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"WordPress Plugin Path Probing"; \
  http.method; content:"GET"; \
  http.uri; pcre:"/\/wp-content\/plugins\/[^\/]+\/(readme|style|changelog)\.txt/"; \
  flow:to_server,established; \
  threshold:type both, track by_src, count 15, seconds 30; \
  classtype:web-application-attack; \
  sid:2000030; rev:1;)
```

### Mitigation Strategies

#### 1. Hide Plugin Directory
```apache
# Apache .htaccess — block directory listing
Options -Indexes

# Block readme.txt access
<Files readme.txt>
    Order Allow,Deny
    Deny from all
</Files>

<Files style.css>
    Order Allow,Deny
    Deny from all
</Files>
```

```nginx
# Nginx config
location ~* /wp-content/plugins/.*readme\.txt$ {
    deny all;
}

location ~* /wp-content/plugins/.*style\.css$ {
    deny all;
}
```

#### 2. Rate Limiting
```nginx
# Nginx rate limiting
limit_req_zone $binary_remote_addr zone=plugins:10m rate=10r/s;

location ~* /wp-content/plugins/ {
    limit_req zone=plugins burst=20 nodelay;
}
```

#### 3. WAF Rules (ModSecurity)
```apache
# Block plugin enumeration bursts
SecRule REQUEST_URI "@rx /wp-content/plugins/[^/]+/(readme|style|changelog)\.txt" \
  "id:1000031,phase:1,pass,nolog, \
   setvar:ip.plugin_probe_count=+1, \
   expirevar:ip.plugin_probe_count=60"

SecRule IP:PLUGIN_PROBE_COUNT "@gt 20" \
  "id:1000032,phase:1,deny,status:403,log,msg:'Plugin Enumeration Blocked'"
```

#### 4. Security Plugins
- **Wordfence:** Blocks plugin enumeration attempts
- **Sucuri:** WAF with plugin scanning protection
- **iThemes Security:** Hides plugin versions

#### 5. Remove Version Information
```php
// functions.php
function remove_plugin_versions() {
    // Remove generator tag
    remove_action('wp_head', 'wp_generator');
    
    // Remove RSD link
    remove_action('wp_head', 'rsd_link');
    
    // Remove wlwmanifest link
    remove_action('wp_head', 'wlwmanifest_link');
}
add_action('init', 'remove_plugin_versions');
```

### Blue Team Response Playbook
1. **Alert triggered:** Plugin enumeration burst detected
2. **Verify:** Check if scanning is authorized
3. **Document:** Log source IP, timestamp, plugins probed
4. **Block:** Add IP to firewall blocklist if unauthorized
5. **Harden:** Hide plugin versions, disable readme.txt access
6. **Update:** Patch vulnerable plugins immediately
7. **Report:** Document incident and response actions

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Not WordPress"
**Cause:** Target is not running WordPress.
**Solution:**
```bash
# Verify WordPress
curl -s https://target.com | grep -i "wp-content"
# If not WordPress, WPProbe won't find anything
```

#### "No plugins found"
**Cause:** WordPress site has no plugins, or wordlist is incomplete.
**Solution:**
```bash
# Try with custom wordlist
wpprobe -w /path/to/larger_wordlist.txt -u https://target.com

# Or use WPScan for comparison
wpscan --url https://target.com -e p
```

#### "Connection timeout"
**Cause:** Target is slow or blocking requests.
**Solution:**
```bash
# Reduce thread count
wpprobe -t 5 -u https://target.com
```

#### "Too many requests / 429 errors"
**Cause:** Target rate-limiting requests.
**Solution:**
```bash
# Reduce threads
wpprobe -t 3 -u https://target.com
# Or use custom User-Agent
curl -H "User-Agent: Mozilla/5.0" "https://target.com/wp-content/plugins/akismet/readme.txt"
```

### Performance Optimization
```bash
# Fast scan (low thread count)
wpprobe -t 5 -u https://target.com

# Thorough scan (high thread count)
wpprobe -t 100 -u https://target.com

# With custom wordlist for specific plugins
wpprobe -w targeted_plugins.txt -u https://target.com
```

### FAQ

**Q: How is WPProbe different from WPScan?**
A: WPProbe is faster for plugin enumeration only. WPScan does full vulnerability scanning, user enumeration, and brute-force attacks.

**Q: Can WPProbe detect plugin versions?**
A: Not directly. Use WPScan with `--api-token` for version and vulnerability data.

**Q: Does WPProbe work with non-WordPress sites?**
A: No, it specifically targets WordPress plugin paths.

**Q: How do I create a custom plugin wordlist?**
A: Extract plugin names from WPScan's database or create based on known WordPress plugins.

---

## Resources

- [WPProbe GitHub](https://github.com/chokeenworwor/wpprobe)
- [WPScan Plugin Database](https://wpscan.com/plugins)
- [WordPress Plugin Directory](https://wordpress.org/plugins/)
- [MITRE ATT&CK TA0043](https://attack.mitre.org/tactics/TA0043/)
- [OWASP WordPress Security](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing)
