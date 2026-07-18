# JavaSnoop Cheatsheet

## Quick Start

```bash
javasnoop                                    # Launch GUI
```

## Workflow

1. Launch: `javasnoop`
2. Select running Java process
3. Load JAR/class files
4. Select methods to hook
5. Write interceptors
6. Activate hooks

## GUI Features

| Feature | Description |
|---------|-------------|
| Process List | Running Java processes |
| Method Browser | Classes and methods |
| Hook Editor | Before/after logic |
| Call Monitor | Intercepted calls |
| Field Inspector | Object fields |

## Hook Syntax

```javascript
// Before advice
before: {
    log("Method called: " + $method.getName());
    log("Args: " + java.util.Arrays.toString($args));
}

// After advice
after: {
    log("Result: " + $ret);
}

// Modify return value
after: {
    return true;
}
```

## Common Hooks

```javascript
// Bypass authentication
after: {
    return true;
}

// Log all HTTP requests
before: {
    log("URL: " + $args[0]);
}

// Capture encrypted data
before: {
    log("Plaintext: " + new String($args[0]));
}
after: {
    log("Ciphertext: " + bytesToHex($ret));
}
```

## Variables

| Variable | Description |
|----------|-------------|
| `$method` | Method object |
| `$args` | Arguments array |
| `$ret` | Return value |
| `$this` | Object instance |

## Class-Level Hooks

```javascript
class: com.example.Auth
method: *
before: {
    log("Auth method: " + $method.getName());
}
```

## Tips

- Hook specific methods, not all methods
- Use conditional hooks to reduce noise
- Combine with jadx for static analysis
- Use Frida for modern Java/Android hooking
- Check jps for running Java processes
