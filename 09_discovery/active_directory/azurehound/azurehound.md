---
title: azurehound
category: discovery
subcategory: active_directory
phase: 09
mitre_attack: TA0007
tags:
  - azure
  - active_directory
  - cloud
  - enumeration
  - attack_paths
difficulty: advanced
os: linux
install: sudo apt install azurehound
github: https://github.com/BloodHoundAD/AzureHound
related:
  - bloodhound
  - sharphound
  - azure-cli
---

# azurehound

## Chapter 1: Overview

**AzureHound** is a data collector for BloodHound that enumerates Azure Active Directory (Azure AD) and Azure resources. It identifies attack paths in hybrid and cloud-only Azure environments.

### Key Features

- **Azure AD enumeration**: Users, groups, applications, roles
- **Azure resource discovery**: VMs, storage, key vaults
- **Hybrid AD support**: On-prem + cloud relationships
- **Attack path analysis**: Identify cloud-based attack paths
- **BloodHound integration**: Import into BloodHound GUI

### Azure Resources Enumerated

| Resource | Description |
|----------|-------------|
| Azure AD Users | User accounts and properties |
| Azure AD Groups | Group memberships |
| Azure AD Applications | App registrations and permissions |
| Azure AD Service Principals | Service accounts |
| Azure Subscriptions | Subscription roles |
| Azure Resources | VMs, Storage, Key Vault |

---

## Chapter 2: Installation & Setup

### Install AzureHound

```bash
# Download from GitHub releases
wget https://github.com/BloodHoundAD/AzureHound/releases/latest/download/azurehound-linux-amd64
chmod +x azurehound-linux-amd64
sudo mv azurehound-linux-amd64 /usr/local/bin/azurehound
```

### Via Package Manager

```bash
sudo apt update && sudo apt install azurehound
```

### Azure CLI Login

```bash
az login
```

### Verify Installation

```bash
azurehound --help
```

---

## Chapter 3: Authentication

### Azure CLI Authentication

```bash
az login
azurehound -l
```

### Service Principal

```bash
az login --service-principal -u APP_ID -p SECRET --tenant TENANT_ID
azurehound -l
```

### Managed Identity

```bash
# Run on Azure VM with managed identity
azurehound -l
```

### Username/Password

```bash
az login -u user@domain.com -p 'Password'
```

---

## Chapter 4: Basic Usage

### List Tenants

```bash
azurehound -l
```

### Enumerate Tenant

```bash
azurehound -t TENANT_ID -o output.json
```

### Export for BloodHound

```bash
azurehound -t TENANT_ID -o azure_data.json
```

### Verbose Output

```bash
azurehound -t TENANT_ID -v
```

---

## Chapter 5: Collection Options

### Collect All Data

```bash
azurehound -t TENANT_ID --all -o output.json
```

### Collect Specific Resources

```bash
azurehound -t TENANT_ID --users --groups -o output.json
```

### Available Options

```bash
--users          Enumerate users
--groups         Enumerate groups
--applications   Enumerate applications
--service-principals  Enumerate service principals
--role-assignments   Enumerate role assignments
--all            Enumerate everything
```

---

## Chapter 6: Advanced Features

### Proxy Support

```bash
azurehound -t TENANT_ID --proxy http://127.0.0.1:8080 -o output.json
```

### Filter by OU

```bash
azurehound -t TENANT_ID --filter "OU=IT" -o output.json
```

### Output Formats

```bash
# JSON (default)
azurehound -t TENANT_ID -o output.json

# CSV
azurehound -t TENANT_ID --format csv -o output.csv
```

---

## Chapter 7: Real-World Scenarios

### Scenario 1: Quick Azure AD Assessment

```bash
az login
azurehound -l  # List tenants
azurehound -t TENANT_ID -o azure_assessment.json
```

### Scenario 2: Hybrid AD Enumeration

```bash
# On-prem enumeration
bloodhound-python -u user -p pass -d corp.local -ns 192.168.1.1 -c All

# Azure enumeration
az login
azurehound -t TENANT_ID -o azure_data.json

# Import both to BloodHound
```

### Scenario 3: Service Principal Assessment

```bash
az login --service-principal -u APP_ID -p SECRET --tenant TENANT_ID
azurehound -t TENANT_ID --service-principals --applications -o sp_audit.json
```

---

## Chapter 8: Mitigation & Defense

**For cloud administrators:**

- **Least privilege**: Minimize Azure AD roles
- **Conditional Access**: Enforce MFA and location
- **Monitor enumeration**: Alert on AzureHound patterns
- **Protect service principals**: Rotate credentials
- **Audit role assignments**: Review privileged access
- **Network restrictions**: Limit management access
- **Azure AD Protection**: Enable P2 features

### Detection Queries

```kusto
// Azure AD Audit Logs
AuditLogs | where OperationName == "List users"
```

### Conditional Access Rules

```json
{
  "conditions": {
    "users": {
      "includeUsers": ["All"]
    },
    "applications": {
      "includeApplications": ["All"]
    }
  },
  "grantControls": {
    "builtInControls": ["mfa"]
  }
}
```

---

## Resources

- [AzureHound GitHub](https://github.com/BloodHoundAD/AzureHound)
- [Azure AD Security](https://docs.microsoft.com/en-us/azure/active-directory/enterprise-identities-open-report)
- [OffSec PEN-200 Course](https://www.offsec.com/courses/pen-200/)
- [Azure Security](https://azure.microsoft.com/en-us/services/security-center/)

---

*Generated for educational purposes in authorized security testing environments only.*
