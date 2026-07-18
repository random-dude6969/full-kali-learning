---
tool: "instaloader"
category: "OSINT / Social Media Reconnaissance"
---

# Instaloader Cheatsheet

## Installation

```bash
pip3 install instaloader              # Install via pip
sudo apt install instaloader          # Install on Kali
instaloader --version                 # Check version
```

## Authentication

```bash
instaloader --login USERNAME                          # Interactive login
instaloader --login USERNAME --password PASS          # Non-interactive (avoid)
instaloader --session-file /path/to/session.file      # Use session file
rm -f ~/.config/instaloader/USERNAME.session          # Clear cached session
```

## Profile Download

```bash
instaloader TARGET_USER                                # Download full profile
instaloader --count 50 TARGET_USER                     # Last 50 posts only
instaloader --metadata-json TARGET_USER                # With JSON metadata
instaloader --compress-json TARGET_USER                # Compress metadata (xz)
instaloader -d ./downloads TARGET_USER                 # Custom output dir
instaloader --count 0 TARGET_USER                      # Profile picture only
```

## Content Filtering

```bash
instaloader --no-pictures TARGET_USER                  # Videos only
instaloader --no-videos TARGET_USER                    # Images only
instaloader --no-video-thumbnails TARGET_USER          # Skip thumbnails
instaloader --no-captions TARGET_USER                  # Skip captions
instaloader --no-metadata-json TARGET_USER             # No metadata files
```

## Specific Content

```bash
instaloader -- SHORTCODE                              # Post by shortcode
instaloader https://www.instagram.com/p/SHORTCODE/     # Post by URL
instaloader --id USERID                                # Profile by numeric ID
```

## Rate Limiting & Evasion

```bash
instaloader --sleep 60 TARGET_USER                     # 60s delay between posts
instaloader --sleep 120 --max-connection-attempts 5 T  # Slow + retries
instaloader --request-timeout 30 TARGET_USER           # Custom timeout
instaloader --proxy http://proxy:8080 TARGET_USER      # HTTP proxy
instaloader --proxy socks5://proxy:1080 TARGET_USER    # SOCKS5 proxy
```

## Output Patterns

```bash
--dirname-pattern './dl/{profile}'          # Custom directory layout
--filename-pattern '{date_utc}_{shortcode}' # Custom filenames
```

**Filename variables:** `{profile}`, `{date_utc}`, `{date}`, `{shortcode}`, `{mediaid}`, `{caption}`, `{likes}`, `{comments}`, `{typename}`

## Batch Download (Bash)

```bash
#!/bin/bash
# Download profiles from file with random delays
while read -r user; do
    delay=$(( RANDOM % 121 + 60 ))
    instaloader --count 50 --sleep "$delay" --metadata-json "$user"
done < targets.txt
```

## Python API — One-Liners

```python
import instaloader; L = instaloader.Instaloader()
p = instaloader.Profile.from_username(L.context, "user")
print(p.username, p.followers, p.mediacount, p.is_private)
```

## Python API — Download Profile

```python
L = instaloader.Instaloader()
p = instaloader.Profile.from_username(L.context, "target")
for post in p.get_posts():
    L.download_post(post, target="target")
```

## Python API — Specific Post

```python
post = instaloader.Post.from_shortcode(L.context, "CwBQAEoF")
print(post.caption, post.likes, post.date_utc, post.is_video)
L.download_post(post)
```

## Python API — Stories (Authenticated)

```python
L.login("you", "pass")
p = instaloader.Profile.from_username(L.context, "target")
for story in L.get_stories(userids=[p.userid]):
    for item in story.get_items():
        L.download_storyitem(item, target="target/stories")
```

## Python API — Highlights

```python
L.login("you", "pass")
p = instaloader.Profile.from_username(L.context, "target")
for hl in L.get_highlights(p):
    for item in hl.get_items():
        L.download_storyitem(item, target=f"target/highlights/{hl.title}")
```

## Forensics Workflow

```bash
# Archive with chain of custody
echo "Target: TARGET | Operator: $(whoami) | Time: $(date -u)" > custody.txt
instaloader -d evidence/ --metadata-json --compress-json --count 1000 TARGET
find evidence/ -type f -exec sha256sum {} \; > checksums.sha256
```

## OSINT Pipeline

```bash
# Sherlock → Instaloader
sherlock TARGET --site instagram
grep -rhoE 'instagram\.com/[a-zA-Z0-9._]+' sherlock_results/ | \
    cut -d/ -f3 | sort -u | while read u; do
        instaloader --count 100 --metadata-json "$u"
    done
```

## Troubleshooting

| Error | Fix |
|-------|-----|
| `Login required` | `instaloader --login USER TARGET` |
| `Rate limit exceeded` | Add `--sleep 120`, wait 15-30 min |
| `Profile not found` | Verify username: `curl -s -o /dev/null -w "%{http_code}" https://instagram.com/USER/` |
| `Challenge required` | Delete session, re-login interactively |
| `401 Unauthorized` | Re-authenticate: delete `.session` file, `--login` again |
| `Connection timeout` | Increase `--request-timeout 60 --max-connection-attempts 5` |
| Nothing downloaded | Check `--no-pictures` / `--no-videos` flags aren't both set |

## Detection Indicators

- API requests to `web_profile_info` or `graphql/query`
- Bulk downloads from `cdninstagram.com`
- Sequential username queries in short timeframes
- Missing browser artifacts (no JS execution)
- `python-requests` User-Agent string
