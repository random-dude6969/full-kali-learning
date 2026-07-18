# bloodhound CHEATSHEET

## Quick Reference

### Setup
```bash
sudo bloodhound-setup          # Initial setup
sudo bloodhound-start          # Start services
sudo bloodhound-stop           # Stop services
```

### Access
```bash
http://localhost:7474           # Neo4j browser
http://localhost:8080           # BloodHound GUI
```

### Default Credentials
```
Neo4j: neo4j/neo4j (change immediately)
BloodHound: admin/admin (change on first login)
```

### Reset Admin Password
```bash
sudo env bhe_recreate_default_admin=true bloodhound
```

### Update Neo4j Password
```bash
sudo vim /etc/bhapi/bhapi.json
# Update "secret" field
```

### SharpHound (Windows)
```powershell
.\SharpHound.exe -c All                            # Collect all
.\SharpHound.exe -c Users,Groups,ACLs              # Specific
.\SharpHound.exe -c All --stealth                   # Stealth
.\SharpHound.exe -c All --loop --loopduration 02:00:00  # Loop
```

### bloodhound-python (Linux)
```bash
bloodhound-python -u user -p pass -d domain.local -ns 192.168.1.1 -c All
bloodhound-python -u user -H aad3b435b51404eeaad3b435b51404ee:da76f... -d domain.local -ns 192.168.1.1 -c All
```

### Common Queries
```cypher
// Shortest path to DA
MATCH p=shortestPath((n:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@"})) RETURN p

// All Domain Admins
MATCH (g:Group {name:"DOMAIN ADMINS@"})<-[:MemberOf]-(u:User) RETURN u

// Kerberoastable users
MATCH (u:User {hasspn:true}) RETURN u

// AS-REP Roastable
MATCH (u:User {dontreqpreauth:true}) RETURN u
```

---

## Target: 192.168.1.x

```bash
# Linux collection
bloodhound-python -u user@domain.local -p 'Password' -d domain.local -ns 192.168.1.1 -c All

# Import to BloodHound GUI
```
