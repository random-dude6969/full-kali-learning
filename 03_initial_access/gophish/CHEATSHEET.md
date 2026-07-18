# Gophish Quick Reference

## Service Management

```bash
# Start
sudo gophish-start

# Stop
sudo gophish-stop

# Status (via gophish-stop output)

# Logs
sudo journalctl -u gophish -f
```

## Default Access

| Setting | Value |
|---------|-------|
| Admin URL | `https://127.0.0.1:3333` |
| Landing Pages | `http://127.0.0.1:8080` |
| Username (Kali) | `admin` |
| Password (Kali) | `kali-gophish` |

## Installation

```bash
# Kali package
sudo apt install gophish

# From source
git clone https://github.com/gophish/gophish.git
cd gophish && go build

# Docker
docker run -p 3333:3333 -p 8080:8080 gophish/gophish
```

## API Quick Reference

```bash
# Set API key variable
export GOPHISH_KEY="YOUR_API_KEY"
export GOPHISH_URL="https://127.0.0.1:3333"

# List campaigns
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/campaigns/"

# Get campaign details
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/campaigns/1"

# Get campaign results
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/campaigns/1/results"

# List email templates
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/templates/"

# List groups
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/groups/"

# List landing pages
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/pages/"

# List sending profiles
curl -k -H "Authorization: $GOPHISH_KEY" "$GOPHISH_URL/api/smtp/"

# Create campaign
curl -k -X POST -H "Authorization: $GOPHISH_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Test","template":{"id":1},"page":{"id":1},"smtp":{"id":1},"groups":[{"id":1}]}' \
  "$GOPHISH_URL/api/campaigns/"

# Import targets via CSV
curl -k -H "Authorization: $GOPHISH_KEY" -F "file=@targets.csv" \
  "$GOPHISH_URL/api/import/group"
```

## Email Template Variables

| Variable | Description |
|----------|-------------|
| `{{.FirstName}}` | Target first name |
| `{{.LastName}}` | Target last name |
| `{{.Email}}` | Target email |
| `{{.Position}}` | Job position |
| `{{.URL}}` | Tracking URL |
| `{{.From}}` | Sender email |
| `{{.RID}}` | Tracking ID |

## Python Client

```python
# Install
pip3 install gophish

# Connect
from gophish import Gophish
api = Gophish('YOUR_API_KEY', host='https://127.0.0.1:3333', verify=False)

# List campaigns
campaigns = api.campaigns.get()

# Create campaign
from gophish.models import Campaign, Template, Page, SMTP, Group
c = Campaign(
    name="Test",
    template=Template(id=1),
    page=Page(id=1),
    smtp=SMTP(id=1),
    groups=[Group(id=1)]
)
api.campaigns.post(c)
```

## Export Results Script

```bash
#!/bin/bash
# Export campaign results to CSV
curl -k -s -H "Authorization: $GOPHISH_KEY" \
  "$GOPHISH_URL/api/campaigns/$1/results" | \
  python3 -c "
import json, sys, csv
results = json.load(sys.stdin)
w = csv.writer(sys.stdout)
w.writerow(['Email','Status','Submitted','Data'])
for r in results:
    w.writerow([r['email'], r['status'], r.get('submitted',False), r.get('submitted_data',{})])
" > "campaign_$1_results.csv"
echo "Exported to campaign_$1_results.csv"
```

## Metrics Calculator

```bash
#!/bin/bash
# Calculate campaign metrics
curl -k -s -H "Authorization: $GOPHISH_KEY" \
  "$GOPHISH_URL/api/campaigns/$1/results" | \
  python3 -c "
import json, sys
results = json.load(sys.stdin)
total = len(results)
sent = sum(1 for r in results if r['status'] in ['Email Sent','Email Opened','Clicked Link','Submitted Data'])
opened = sum(1 for r in results if r['status'] in ['Email Opened','Clicked Link','Submitted Data'])
clicked = sum(1 for r in results if r['status'] in ['Clicked Link','Submitted Data'])
submitted = sum(1 for r in results if r['status'] == 'Submitted Data')
print(f'Campaign Results (Total: {total})')
print(f'  Sent: {sent}/{total} ({sent/total*100:.1f}%)')
print(f'  Opened: {opened}/{total} ({opened/total*100:.1f}%)')
print(f'  Clicked: {clicked}/{total} ({clicked/total*100:.1f}%)')
print(f'  Submitted: {submitted}/{total} ({submitted/total*100:.1f}%)')
"
```

## SMTP Configuration Reference

| Provider | Host | Port | TLS |
|----------|------|------|-----|
| Office 365 | smtp.office365.com | 587 | STARTTLS |
| Gmail | smtp.gmail.com | 587 | STARTTLS |
| AWS SES | email-smtp.us-east-1.amazonaws.com | 587 | STARTTLS |
| SendGrid | smtp.sendgrid.net | 587 | STARTTLS |
| Mailgun | smtp.mailgun.org | 587 | STARTTLS |

## Campaign Best Practices

- **Best send times:** Tue-Thu, 9-11 AM
- **Avoid:** Monday mornings, Friday afternoons
- **Phases:** Baseline → Training → Re-test → Measure improvement
- **Track:** Open rate, click rate, submission rate
- **Credentials:** Never use real employee credentials for testing

## Docker Compose

```yaml
version: '3'
services:
  gophish:
    image: gophish/gophish
    ports:
      - "3333:3333"
      - "8080:8080"
    volumes:
      - ./data:/opt/gophish/data
    restart: unless-stopped
```

## Quick Campaign Workflow

1. **Create Group** → Import target CSV
2. **Create Template** → Write phishing email with `{{.URL}}`
3. **Create Landing Page** → Clone real login page, enable capture
4. **Configure SMTP** → Add sending profile
5. **Launch Campaign** → Set template, page, SMTP, group, schedule
6. **Monitor Results** → Dashboard shows real-time metrics
7. **Export Report** → Download results for documentation
