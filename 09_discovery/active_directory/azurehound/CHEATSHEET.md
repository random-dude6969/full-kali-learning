# azurehound CHEATSHEET

## Quick Reference

### Authentication
```bash
az login                                        # Azure CLI login
az login --service-principal -u APP -p SECRET --tenant TENANT
```

### List Tenants
```bash
azurehound -l
```

### Enumerate Tenant
```bash
azurehound -t TENANT_ID -o output.json
```

### Options
```bash
-t tenant          # Tenant ID
-o output          # Output file
-v                 # Verbose
--all              # Enumerate all
--users            # Users only
--groups           # Groups only
--applications     # Applications
--service-principals  # Service principals
--role-assignments    # Role assignments
```

### Output Formats
```bash
-o output.json     # JSON (default)
--format csv -o output.csv  # CSV
```

---

## Target: 192.168.1.x

```bash
az login
azurehound -l
azurehound -t TENANT_ID -o azure_data.json
```
