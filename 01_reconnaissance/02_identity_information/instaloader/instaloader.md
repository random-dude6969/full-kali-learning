---
tool_name: "instaloader"
display_name: "Instaloader"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "identity_information"
install_command: "pip3 install instaloader"
version: "4.10.1"
github_repo: "https://github.com/instaloader/instaloader"
kali_tools_page: "https://www.kali.org/tools/instaloader/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["instagram", "osint", "social-media", "scraping", "photos", "metadata", "profiles", "stories"]
related_tools: ["sherlock", "photon", "holmes", "maigret"]
---

# Instaloader — Instagram Data Downloader

## What This Tool Does (in 30 seconds)

Instaloader is a Python tool and command-line utility for downloading Instagram content. It can fetch profiles, posts, stories, highlights, and metadata from public accounts — and from private accounts when authenticated. It provides both media files (photos, videos) and structured metadata (JSON) for OSINT analysis, digital forensics, or research.

---

## Chapter 1: Introduction & Overview

### What is Instaloader?

Instaloader is a Python library and command-line tool for downloading content from Instagram. It communicates with Instagram's internal API to retrieve profile information, posts (photos and videos), stories, highlights, and associated metadata. It can download content from public profiles without any authentication and from private profiles when an authenticated session is available.

The tool is written in Python and distributed via pip. It is also available in Kali Linux repositories. Instaloader has been actively maintained since 2017 and is widely used in OSINT, digital forensics, and social media research workflows.

### Core Capabilities

| Capability | Description |
|------------|-------------|
| Profile download | Download all posts, profile picture, and metadata for a given username |
| Post download | Download a specific post by shortcode or URL |
| Stories & highlights | Download stories and highlight reels (requires authentication) |
| Metadata export | Save post and profile metadata as structured JSON |
| Resumable downloads | Resume interrupted downloads without re-downloading existing content |
| Rate limiting | Built-in delays and retry logic to avoid Instagram blocks |
| Python API | Full programmatic access for custom scripts and automation |
| Batch downloading | Download multiple profiles in sequence from a list |

### Use Cases in Security & OSINT

- **OSINT investigations**: Gather intelligence from Instagram profiles during investigations
- **Digital forensics**: Preserve social media evidence with timestamps and metadata for chain of custody
- **Threat intelligence**: Monitor threat actors who use Instagram for communication or recruitment
- **Brand monitoring**: Track brand mentions, logos, and visual content across Instagram
- **Research**: Analyze social media posting patterns, hashtag usage, and engagement metrics
- **Background checks**: Supplement background investigations with publicly available Instagram data

### How It Works Under the Hood

Instaloader communicates with Instagram's internal GraphQL API. When you request a profile, it:

1. Resolves the username to an internal user ID via the `web_profile_info` endpoint
2. Queries the GraphQL API for profile metadata (bio, follower count, post count, etc.)
3. Iterates through post edges to download media files and captions
4. Saves media to disk in organized directory structures
5. Optionally exports all metadata as JSON for later analysis

For authenticated sessions, Instaloader can also access:
- Private profile content
- Stories and highlights
- Post like counts (when visible)
- Follower/following lists (limited)

### Legal and Ethical Considerations

- Only download from **public** profiles without authentication unless you have explicit authorization
- Respect Instagram's Terms of Service — automated downloading may violate their ToS
- Use for legitimate research, investigation, or personal archival purposes only
- Do not redistribute downloaded content without the content owner's permission
- Be aware of local privacy laws (GDPR, CCPA, etc.) when collecting personal data
- Rate limiting exists for a reason — excessive scraping can result in IP bans or legal action
- This tool is provided for educational and authorized security testing purposes

---

## Chapter 2: Installation & Setup

### Installation Methods

#### Method 1: pip (Recommended)

```bash
pip3 install instaloader
```

Expected output:

```
Collecting instaloader
  Downloading instaloader-4.10.1-py3-none-any.whl (54 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 54.2/54.2 kB 1.2 MB/s eta 0:00:00
Collecting requests>=2.20.0 (from instaloader)
  Using cached requests-2.31.0-py3-none-any.whl (62 kB)
Collecting urllib3>=1.26.0 (from instaloader)
  Using cached urllib3-2.0.7-py3-none-any.whl (104 kB)
Collecting certifi>=2017.4.17 (from instaloader)
  Using cached certifi-2023.7.22-py3-none-any.whl (154 kB)
Collecting charset-normalizer<4,>=2 (from requests>=2.20.0->instaloader)
  Using cached charset_normalizer-3.2.0-cp310-cp310-manylinux_2_17_x86_64.whl (142 kB)
Installing collected packages: urllib3, charset-normalizer, certifi, requests, instaloader
Successfully installed certifi-2023.7.22 charset-normalizer-3.2.0 instaloader-4.10.1 requests-2.31.0 urllib3-2.0.7
```

#### Method 2: Kali Linux (Pre-installed)

```bash
sudo apt update && sudo apt install instaloader
```

Expected output:

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  instaloader
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 102 kB of archives.
After this operation, 395 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 instaloader all 4.10.1-1kali1 [102 kB]
Fetched 102 kB in 1s (102 kB/s)
Selecting previously unselected package instaloader.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../instaloader_4.10.1-1kali1_all.deb ...
Unpacking instaloader (4.10.1-1kali1) ...
Setting up instaloader (4.10.1-1kali1) ...
```

#### Method 3: From Source (Latest Development)

```bash
git clone https://github.com/instaloader/instaloader.git
cd instaloader
pip3 install -e .
```

Expected output:

```
Cloning into 'instaloader'...
remote: Enumerating objects: 5234, done.
remote: Counting objects: 100% (1234/1234), done.
remote: Compressing objects: 100% (456/456), done.
remote: Total 5234 (delta 876), reused 1100 (delta 750), pack-reused 4000
Receiving objects: 100% (5234/5234), 1.23 MiB | 5.67 MiB/s, done.
Resolving deltas: 100% (3456/3456), done.
Installing collected packages: instaloader
  Running setup.py develop for instaloader
Successfully installed instaloader-4.10.1
```

### Verify Installation

```bash
instaloader --version
```

Expected output:

```
4.10.1
```

```bash
instaloader --help | head -20
```

Expected output:

```
usage: instaloader [-h] [--version] [-V] [--login login] [--password password]
                   [-d directory] [--filename-pattern filename-pattern]
                   [--dirname-pattern dirname-pattern] [--id] [--json]
                   [--no-captions] [--no-compress-json] [--no-metadata-json]
                   [--no-pictures] [--no-videos] [--no-video-thumbnails]
                   [--count COUNT] [--sleep SLEEP]
                   ...
```

### Authentication Setup

For accessing private profiles, stories, and highlights, you need to authenticate with Instagram.

#### Interactive Login

```bash
instaloader --login YOUR_USERNAME
```

Expected output:

```
Enter Instagram password for YOUR_USERNAME:
```

After entering your password:

```
Login to YOUR_USERNAME successful.
Session file stored in /home/user/.config/instaloader/YOUR_USERNAME.session
```

The session file is cached at `~/.config/instaloader/YOUR_USERNAME.session` and reused for subsequent requests.

#### Non-Interactive Login (Scripts)

```bash
instaloader --login YOUR_USERNAME --password YOUR_PASSWORD target_profile
```

Expected output:

```
Login to YOUR_USERNAME successful.
Downloading target_profile
target_profile/2024-01-15_14-30-22_UTC_1.jpg
target_profile/2024-01-15_14-30-22_UTC_1.mp4
...
```

> **Warning:** Storing passwords in scripts or command history is a security risk. Use environment variables or session files instead.

#### Using Session Files

```bash
# Export session to environment variable
export INSTALOADER_SESSION=/home/user/.config/instaloader/YOUR_USERNAME.session

# Or specify session file directly
instaloader --session-file /path/to/session.file target_profile
```

### Configuration Options

#### Set Download Directory Pattern

```bash
instaloader --dirname-pattern './downloads/{profile}' target_username
```

#### Set Filename Pattern

```bash
instaloader --filename-pattern '{date_utc}_UTC_{shortcode}' target_username
```

#### Configure Connection Settings

```bash
# Maximum connection attempts before failing
instaloader --max-connection-attempts 3 target_username

# Sleep between requests (seconds)
instaloader --sleep 60 target_username

# Request timeout (seconds)
instaloader --request-timeout 30 target_username
```

### Python API Setup

```python
import instaloader

# Basic initialization
L = instaloader.Instaloader()

# With custom settings
L = instaloader.Instaloader(
    download_videos=False,
    download_video_thumbnails=False,
    download_geotags=False,
    download_comments=False,
    save_metadata=True,
    compress_json=False,
    dirname_pattern='{profile}',
    filename_pattern='{date_utc}_UTC_{shortcode}'
)
```

---

## Chapter 3: Basic Usage

### First Download — Quick Start

The simplest way to use Instaloader is to pass a username directly:

```bash
instaloader target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg
target_username/2024-01-15_14-30-22_UTC_1_description.txt
target_username/2024-01-14_09-15-44_UTC_2.jpg
target_username/2024-01-14_09-15-44_UTC_2_description.txt
target_username/2024-01-13_18-45-11_UTC_3.mp4
target_username/2024-01-13_18-45-11_UTC_3_description.txt
[45 posts downloaded, 0 skipped, 2 errors]
```

### Profile Download

#### Download All Posts from a Profile

```bash
instaloader target_username
```

This downloads every post from the profile, organized in a directory named after the username.

#### Download Only Recent Posts

```bash
instaloader --count 30 target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg
target_username/2024-01-15_14-30-22_UTC_1_description.txt
...
[30 posts downloaded, 0 skipped, 0 errors]
```

#### Download with Metadata

```bash
instaloader --metadata-json target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg
target_username/2024-01-15_14-30-22_UTC_1_metadata.json
...
[45 posts downloaded, metadata saved to JSON]
```

The metadata JSON contains structured data including:

```json
{
  "shortcode": "CwBQAEoF",
  "date_utc": "2024-01-15T14:30:22",
  "caption": "Beautiful sunset #photography #nature",
  "likes": 1523,
  "comments": 47,
  "is_video": false,
  "location": {"name": "New York, NY"},
  "hashtags": ["photography", "nature"],
  "mentions": ["@friend1", "@friend2"]
}
```

### Specific Content Download

#### Download a Specific Post by Shortcode

```bash
instaloader -- -CwBQAEoF
```

Expected output:

```
Downloading post CwBQAEoF
-CwBQAEoF_2024-01-10_12-00-00_UTC.jpg
-CwBQAEoF_2024-01-10_12-00-00_UTC_description.txt
```

#### Download a Post by Full URL

```bash
instaloader https://www.instagram.com/p/CwBQAEoF/
```

Expected output:

```
Downloading post CwBQAEoF
p/CwBQAEoF_2024-01-10_12-00-00_UTC.jpg
p/CwBQAEoF_2024-01-10_12-00-00_UTC_description.txt
```

#### Download a Profile Picture

```bash
instaloader --no-videos --count 0 target_username
```

Expected output:

```
Downloading target_username profile picture
target_username/profile.jpg
```

### Content Filtering

#### Download Only Videos

```bash
instaloader --no-pictures target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-13_18-45-11_UTC_3.mp4
target_username/2024-01-10_08-22-33_UTC_5.mp4
[2 posts downloaded, 43 skipped (pictures not wanted)]
```

#### Download Only Pictures

```bash
instaloader --no-videos target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg
target_username/2024-01-14_09-15-44_UTC_2.jpg
[43 posts downloaded, 2 skipped (videos not wanted)]
```

#### Skip Video Thumbnails

```bash
instaloader --no-video-thumbnails target_username
```

#### Download without Captions

```bash
instaloader --no-captions target_username
```

### Output Control

#### Save to Specific Directory

```bash
instaloader -d ./output target_username
```

Expected output:

```
Downloading target_username
./output/target_username/2024-01-15_14-30-22_UTC_1.jpg
...
```

#### Compress Metadata JSON

```bash
instaloader --compress-json target_username
```

This produces `.json.xz` files instead of plain `.json`, reducing disk usage.

### Complete Flag Reference

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `-d`, `--dirname-pattern` | Output directory pattern | `{profile}` | `-d ./downloads/{profile}` |
| `--filename-pattern` | Filename pattern | `{date_utc}_UTC_{shortcode}` | `--filename-pattern {date}` |
| `--count` | Number of posts to download | All | `--count 50` |
| `--no-pictures` | Skip image posts | false | `--no-pictures` |
| `--no-videos` | Skip video posts | false | `--no-videos` |
| `--no-video-thumbnails` | Skip video thumbnails | false | `--no-video-thumbnails` |
| `--no-captions` | Skip downloading captions | false | `--no-captions` |
| `--no-metadata-json` | Don't save metadata as JSON | false | `--no-metadata-json` |
| `--metadata-json` | Save metadata as JSON | false | `--metadata-json` |
| `--compress-json` | Compress JSON with xz | false | `--compress-json` |
| `--login` | Instagram username for auth | None | `--login user` |
| `--password` | Instagram password (insecure) | None | `--password pass` |
| `--session-file` | Path to session file | Auto | `--session-file /path` |
| `--sleep` | Seconds to sleep between requests | 0 | `--sleep 60` |
| `--max-connection-attempts` | Max retries on failure | 3 | `--max-connection-attempts 5` |
| `--request-timeout` | HTTP request timeout in seconds | 300 | `--request-timeout 30` |
| `-V`, `--verbose` | Enable verbose output | false | `-V` |
| `--id` | Download by numeric user ID | false | `--id 12345678` |
| `--json` | Save profile info as JSON | false | `--json` |
| `--geotags` | Download geotag data | false | `--geotags` |
| `--comments` | Download post comments | false | `--comments` |
| `--no-video-downloads` | Don't download video files | false | `--no-video-downloads` |
| `--slide` | Download slideshows as individual | false | `--slide` |
| `-h`, `--help` | Show help message | - | `-h` |

#### Filename Pattern Variables

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `{profile}` | Profile username | `target_username` |
| `{date_utc}` | Post date in UTC | `2024-01-15_14-30-22` |
| `{date}` | Post date in local time | `2024-01-15_22-30-22` |
| `{shortcode}` | Post shortcode | `CwBQAEoF` |
| `{mediaid}` | Numeric media ID | `3001234567890123456` |
| `{caption}` | Post caption text | `Beautiful sunset` |
| `{likes}` | Number of likes | `1523` |
| `{comments}` | Number of comments | `47` |
| `{typename}` | Media type | `GraphImage` |

---

## Chapter 4: Intermediate Usage

### Python API — Profile Information

The Python API provides full programmatic access to Instaloader's functionality.

#### Fetching Profile Details

```python
#!/usr/bin/env python3
import instaloader

L = instaloader.Instaloader()

# Get profile by username
profile = instaloader.Profile.from_username(L.context, "target_username")

print(f"Username:    {profile.username}")
print(f"Full Name:   {profile.full_name}")
print(f"Bio:         {profile.biography}")
print(f"Posts:       {profile.mediacount}")
print(f"Followers:   {profile.followers}")
print(f"Following:   {profile.followees}")
print(f"Is Private:  {profile.is_private}")
print(f"Is Verified: {profile.is_verified}")
print(f"Profile Pic: {profile.profile_pic_url}")
```

Expected output:

```
Username:    target_username
Full Name:   John Doe
Bio:         Photographer | Traveler | NYC based
Posts:       847
Followers:   15234
Following:   432
Is Private:  False
Is Verified: False
Profile Pic: https://scontent-lax3-1.cdninstagram.com/...
```

#### Downloading Profile Posts via API

```python
#!/usr/bin/env python3
import instaloader

L = instaloader.Instaloader()
profile = instaloader.Profile.from_username(L.context, "target_username")

count = 0
for post in profile.get_posts():
    print(f"[{count+1}] {post.shortcode} - {post.date_utc} - {post.likes} likes")
    if post.caption:
        print(f"    Caption: {post.caption[:80]}...")
    L.download_post(post, target=profile.username)
    count += 1
    if count >= 10:
        break

print(f"\nDone: {count} posts downloaded")
```

Expected output:

```
[1] CwBQAEoF - 2024-01-15 14:30:22+00:00 - 1523 likes
    Caption: Beautiful sunset over Manhattan #photography #nature...
[2] BxKz9pQr - 2024-01-14 09:15:44+00:00 - 2105 likes
    Caption: Street food tour in Bangkok #travel #food...
...
Done: 10 posts downloaded
```

#### Fetching Post Metadata

```python
#!/usr/bin/env python3
import instaloader

L = instaloader.Instaloader()

# Download specific post by shortcode
post = instaloader.Post.from_shortcode(L.context, "CwBQAEoF")

print(f"Shortcode:  {post.shortcode}")
print(f"Date:       {post.date_utc}")
print(f"Caption:    {post.caption}")
print(f"Likes:      {post.likes}")
print(f"Comments:   {post.comments}")
print(f"Is Video:   {post.is_video}")
print(f"Video URL:  {post.video_url}")
print(f"Location:   {post.location}")
print(f"Hashtags:   {post.caption_hashtags}")
print(f"Mentions:   {post.caption_mentions}")
```

Expected output:

```
Shortcode:  CwBQAEoF
Date:       2024-01-15 14:30:22+00:00
Caption:    Beautiful sunset over Manhattan #photography #nature
Likes:      1523
Comments:   47
Is Video:   False
Video URL:  None
Location:   New York, New York
Hashtags:   ['photography', 'nature']
Mentions:   ['@friend1']
```

### Batch Download Script

Download multiple profiles from a list file:

```python
#!/usr/bin/env python3
"""Batch download multiple Instagram profiles."""

import instaloader
import sys
import time
import random

def batch_download(usernames_file, max_posts=50, delay_range=(60, 120)):
    """Download profiles listed in a file, one per line."""
    L = instaloader.Instaloader(
        download_videos=True,
        download_video_thumbnails=False,
        save_metadata=True,
        compress_json=False
    )

    with open(usernames_file) as f:
        usernames = [line.strip() for line in f if line.strip()]

    results = {"success": [], "failed": [], "private": []}

    for i, username in enumerate(usernames, 1):
        print(f"\n[{i}/{len(usernames)}] Downloading: {username}")

        try:
            profile = instaloader.Profile.from_username(L.context, username)

            if profile.is_private:
                print(f"  [!] Private profile — skipping (login required)")
                results["private"].append(username)
                continue

            print(f"  Posts: {profile.mediacount} | Followers: {profile.followers}")

            count = 0
            for post in profile.get_posts():
                if count >= max_posts:
                    break
                L.download_post(post, target=username)
                count += 1

            print(f"  [+] Downloaded {count} posts")
            results["success"].append(username)

        except instaloader.exceptions.ProfileNotExistsException:
            print(f"  [-] Profile not found")
            results["failed"].append(username)
        except instaloader.exceptions.LoginRequiredException:
            print(f"  [-] Login required")
            results["private"].append(username)
        except Exception as e:
            print(f"  [-] Error: {e}")
            results["failed"].append(username)

        # Random delay between profiles
        if i < len(usernames):
            delay = random.randint(*delay_range)
            print(f"  Waiting {delay}s before next profile...")
            time.sleep(delay)

    # Summary
    print("\n" + "=" * 50)
    print("BATCH DOWNLOAD SUMMARY")
    print("=" * 50)
    print(f"Successful: {len(results['success'])}")
    print(f"Failed:     {len(results['failed'])}")
    print(f"Private:    {len(results['private'])}")

    if results["failed"]:
        print(f"\nFailed profiles: {', '.join(results['failed'])}")
    if results["private"]:
        print(f"Private profiles: {', '.join(results['private'])}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <usernames_file> [max_posts]")
        sys.exit(1)

    max_posts = int(sys.argv[2]) if len(sys.argv) > 2 else 50
    batch_download(sys.argv[1], max_posts)
```

Usage:

```bash
# Create a list of target usernames
echo -e "user1\nuser2\nuser3" > targets.txt

# Run batch download (50 posts per profile, random 60-120s delay)
python3 batch_download.py targets.txt 50
```

Expected output:

```
[1/3] Downloading: user1
  Posts: 342 | Followers: 8921
  [+] Downloaded 50 posts
  Waiting 87s before next profile...

[2/3] Downloading: user2
  Posts: 156 | Followers: 3456
  [+] Downloaded 50 posts
  Waiting 103s before next profile...

[3/3] Downloading: user3
  Posts: 89 | Followers: 1234
  [+] Downloaded 50 posts

==================================================
BATCH DOWNLOAD SUMMARY
==================================================
Successful: 3
Failed:     0
Private:    0
```

### Metadata Analysis Script

```python
#!/usr/bin/env python3
"""Analyze downloaded Instagram metadata for patterns."""

import json
from pathlib import Path
from collections import Counter
from datetime import datetime

def analyze_profile(profile_dir):
    """Analyze all post metadata in a profile directory."""
    profile_dir = Path(profile_dir)

    # Load all metadata JSON files
    metadata_files = list(profile_dir.glob("*_metadata.json"))
    if not metadata_files:
        # Try alternate naming
        metadata_files = list(profile_dir.glob("*.json"))

    posts = []
    for f in metadata_files:
        try:
            with open(f) as fp:
                data = json.load(fp)
                if isinstance(data, dict):
                    posts.append(data)
        except json.JSONDecodeError:
            continue

    if not posts:
        print("No metadata found. Ensure you downloaded with --metadata-json")
        return

    # Posting patterns
    dates = []
    for p in posts:
        if "date_utc" in p:
            try:
                dt = datetime.fromisoformat(p["date_utc"].replace("Z", "+00:00"))
                dates.append(dt)
            except (ValueError, TypeError):
                pass

    # Hashtag analysis
    hashtags = []
    for p in posts:
        caption = p.get("caption", "")
        if caption:
            tags = [word[1:] for word in caption.split() if word.startswith("#")]
            hashtags.extend(tags)

    # Engagement metrics
    likes = [p.get("likes", 0) for p in posts if "likes" in p]
    comments = [p.get("comments", 0) for p in posts if "comments" in p]

    # Media types
    media_types = Counter(p.get("typename", "unknown") for p in posts)

    # Report
    print("=" * 60)
    print("INSTAGRAM PROFILE ANALYSIS REPORT")
    print("=" * 60)
    print(f"Posts analyzed: {len(posts)}")

    if dates:
        print(f"Date range:     {min(dates).strftime('%Y-%m-%d')} to {max(dates).strftime('%Y-%m-%d')}")
        # Posting frequency
        if len(dates) > 1:
            date_range = (max(dates) - min(dates)).days
            if date_range > 0:
                freq = len(dates) / date_range
                print(f"Posting freq:   {freq:.2f} posts/day")

    if likes:
        print(f"\nEngagement:")
        print(f"  Avg likes:    {sum(likes)/len(likes):.0f}")
        print(f"  Max likes:    {max(likes)}")
        print(f"  Min likes:    {min(likes)}")

    if comments:
        print(f"  Avg comments: {sum(comments)/len(comments):.0f}")

    print(f"\nMedia Types:")
    for mtype, count in media_types.most_common():
        print(f"  {mtype}: {count}")

    if hashtags:
        print(f"\nTop Hashtags:")
        for tag, count in Counter(hashtags).most_common(15):
            print(f"  #{tag}: {count}")

    print("=" * 60)

if __name__ == "__main__":
    import sys
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <profile_directory>")
        sys.exit(1)
    analyze_profile(sys.argv[1])
```

Usage and output:

```bash
python3 analyze_metadata.py target_username/
```

```
============================================================
INSTAGRAM PROFILE ANALYSIS REPORT
============================================================
Posts analyzed: 847
Date range:     2019-03-15 to 2024-01-15
Posting freq:   0.48 posts/day

Engagement:
  Avg likes:    1234
  Max likes:    15230
  Min likes:    23
  Avg comments: 89

Media Types:
  GraphImage: 512
  GraphVideo: 201
  GraphSidecar: 134

Top Hashtags:
  #photography: 312
  #travel: 187
  #nyc: 156
  #nature: 134
  #food: 98
============================================================
```

### Stories and Highlights (Authenticated)

```python
#!/usr/bin/env python3
"""Download stories and highlights from authenticated session."""

import instaloader

L = instaloader.Instaloader()
L.login("your_username", "your_password")  # Or use --login flag

profile = instaloader.Profile.from_username(L.context, "target_username")

# Download current stories
try:
    stories = L.get_stories(userids=[profile.userid])
    story_count = 0
    for story in stories:
        for item in story.get_items():
            L.download_storyitem(item, target=f"{profile.username}/stories")
            story_count += 1
    print(f"Downloaded {story_count} stories")
except Exception as e:
    print(f"Stories error: {e}")

# Download highlights
try:
    highlights = L.get_highlights(profile)
    for highlight in highlights:
        print(f"Highlight: {highlight.title}")
        for item in highlight.get_items():
            L.download_storyitem(item, target=f"{profile.username}/highlights/{highlight.title}")
except Exception as e:
    print(f"Highlights error: {e}")
```

Expected output:

```
Downloaded 12 stories
Highlight: Travel 2024
Highlight: Food Adventures
Highlight: Behind the Scenes
```

---

## Chapter 5: Advanced Usage

### Evasion Techniques

#### Request Throttling and Delays

```bash
# Add a fixed delay between requests (in seconds)
instaloader --sleep 60 target_username
```

Expected output:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg
(target_username has 45 posts, 60s delay between each)
[Sleeping 60 seconds...]
target_username/2024-01-14_09-15-44_UTC_2.jpg
[Sleeping 60 seconds...]
...
```

#### Randomized Delays in Scripts

```bash
#!/bin/bash
# stealth_download.sh — randomized delays between requests

TARGETS_FILE="$1"
MAX_POSTS=${2:-50}

while read -r username; do
    DELAY=$(( RANDOM % 121 + 60 ))  # Random 60-180 seconds
    echo "[*] Downloading: $username (delay: ${DELAY}s)"
    instaloader --count "$MAX_POSTS" --sleep "$DELAY" "$username"
    echo "[+] Completed: $username"
done < "$TARGETS_FILE"
```

Usage:

```bash
chmod +x stealth_download.sh
./stealth_download.sh targets.txt 30
```

Expected output:

```
[*] Downloading: target1 (delay: 94s)
[+] Completed: target1
[*] Downloading: target2 (delay: 157s)
[+] Completed: target2
[*] Downloading: target3 (delay: 73s)
[+ ] Completed: target3
```

#### Proxy Support

```bash
# Use HTTP proxy
instaloader --proxy http://proxy.example.com:8080 target_username

# Use SOCKS5 proxy
instaloader --proxy socks5://proxy.example.com:1080 target_username

# Use proxy with authentication
instaloader --proxy http://user:pass@proxy.example.com:8080 target_username
```

Expected output:

```
Downloading target_username via proxy http://proxy.example.com:8080
target_username/2024-01-15_14-30-22_UTC_1.jpg
...
```

#### Connection Retry Configuration

```bash
# Increase retry attempts and reduce timeout
instaloader --max-connection-attempts 5 --request-timeout 15 target_username
```

### Scaling — Large-Volume Downloads

#### Parallel Downloads with GNU Parallel

```bash
#!/bin/bash
# parallel_download.sh — use GNU Parallel for concurrent downloads

USERS_FILE="$1"
MAX_POSTS=${2:-50}
MAX_PARALLEL=${3:-3}
DELAY_MIN=60
DELAY_MAX=180

download_user() {
    local user="$1"
    local delay=$(( RANDOM % ($DELAY_MAX - $DELAY_MIN + 1) + $DELAY_MIN ))
    sleep "$delay"  # Stagger start times
    instaloader --count "$MAX_POSTS" --sleep 30 "$user" 2>&1 | \
        while read -r line; do echo "[$user] $line"; done
}

export -f download_user

cat "$USERS_FILE" | parallel -j "$MAX_PARALLEL" download_user {}
```

#### Resumable Downloads

Instaloader automatically skips already-downloaded content, making it safe to re-run:

```bash
# Run 1 — interrupted after 30 posts
instaloader --count 100 target_username
# [Ctrl+C after 30 posts]

# Run 2 — resumes, skips already downloaded
instaloader --count 100 target_username
# Downloads only the remaining 70 posts
```

Expected output on resume:

```
Downloading target_username
target_username/2024-01-15_14-30-22_UTC_1.jpg exists
target_username/2024-01-14_09-15-44_UTC_2.jpg exists
...
[target_username] 30 of 100 posts already exist, skipping
target_username/2023-12-20_11-22-33_UTC_31.jpg
...
[70 new posts downloaded]
```

### Integration with Other OSINT Tools

#### Pipeline: Sherlock → Instaloader

```bash
# Step 1: Discover Instagram accounts using Sherlock
sherlock target_person --site instagram

# Step 2: Extract found Instagram usernames
grep -oE 'instagram\.com/[a-zA-Z0-9._]+' sherlock_results/target_person/*.txt | \
    cut -d'/' -f3 | sort | uniq > found_profiles.txt

# Step 3: Download each found profile
while read -r username; do
    instaloader --count 100 --metadata-json "$username"
done < found_profiles.txt
```

#### Pipeline: Photon → Instaloader

```bash
# Step 1: Crawl a website for Instagram references
photon -u https://target-website.com -o photon_output -l 3

# Step 2: Extract Instagram usernames from crawled pages
grep -rhoE 'instagram\.com/[a-zA-Z0-9._]+' photon_output/ | \
    cut -d'/' -f2 | sort -u > instagram_users.txt

# Step 3: Download all discovered profiles
instaloader -d ./ig_downloads --count 200 --metadata-json $(cat instagram_users.txt)
```

#### Integration with Metasploit for Recon

```bash
# After identifying target Instagram accounts
# Use Instaloader to collect intelligence
instaloader --metadata-json --count 200 suspect_username

# Parse metadata for investigation
python3 -c "
import json, glob
for f in glob.glob('suspect_username/*.json'):
    with open(f) as fp:
        data = json.load(fp)
        if 'caption' in data:
            print(f\"{data.get('shortcode', 'N/A')}: {data['caption'][:100]}\")
"
```

### Custom Download Patterns

#### Download Only Posts from a Specific Year

```python
#!/usr/bin/env python3
"""Download posts from a specific year only."""

import instaloader
from datetime import datetime

L = instaloader.Instaloader()
profile = instaloader.Profile.from_username(L.context, "target_username")

target_year = 2024
count = 0

for post in profile.get_posts():
    if post.date_utc.year < target_year:
        break  # Posts are in reverse chronological order
    if post.date_utc.year == target_year:
        L.download_post(post, target=f"{profile.username}/{target_year}")
        count += 1

print(f"Downloaded {count} posts from {target_year}")
```

#### Download Posts Containing Specific Keywords

```python
#!/usr/bin/env python3
"""Download posts matching keyword filters."""

import instaloader

L = instaloader.Instaloader()
profile = instaloader.Profile.from_username(L.context, "target_username")

keywords = ["confidential", "password", "secret", "internal"]
count = 0

for post in profile.get_posts():
    if post.caption:
        caption_lower = post.caption.lower()
        if any(kw in caption_lower for kw in keywords):
            print(f"[MATCH] {post.shortcode}: {post.caption[:80]}")
            L.download_post(post, target=f"{profile.username}/flagged")
            count += 1

print(f"\nFound {count} posts matching keywords")
```

#### Download Posts with Geotags

```python
#!/usr/bin/env python3
"""Download posts that have location data."""

import instaloader
import json

L = instaloader.Instaloader()
profile = instaloader.Profile.from_username(L.context, "target_username")

location_posts = []

for post in profile.get_posts():
    if post.location:
        loc = post.location
        print(f"[{post.shortcode}] Location: {loc}")
        L.download_post(post, target=f"{profile.username}/geotagged")
        location_posts.append({
            "shortcode": post.shortcode,
            "location": str(loc),
            "date": str(post.date_utc),
            "caption": post.caption[:100] if post.caption else ""
        })

# Save location index
with open(f"{profile.username}/location_index.json", "w") as f:
    json.dump(location_posts, f, indent=2)

print(f"\nFound {len(location_posts)} posts with location data")
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Influencer Investigation

**Objective:** Gather intelligence on a suspected influencer account for a brand investigation.

```bash
# Step 1: Download full profile with metadata
instaloader --metadata-json --count 200 influencer_username

# Step 2: Analyze the downloaded data
python3 << 'ANALYSIS'
import json, glob
from collections import Counter

# Load all metadata
posts = []
for f in glob.glob("influencer_username/*_metadata.json"):
    with open(f) as fp:
        posts.append(json.load(fp))

print(f"Posts analyzed: {len(posts)}")

# Hashtag analysis
hashtags = []
for p in posts:
    cap = p.get("caption", "")
    hashtags.extend([w[1:] for w in cap.split() if w.startswith("#")])

print("\nTop hashtags:")
for tag, count in Counter(hashtags).most_common(10):
    print(f"  #{tag}: {count}")

# Posting times
from datetime import datetime
hours = []
for p in posts:
    if "date_utc" in p:
        dt = datetime.fromisoformat(p["date_utc"].replace("Z", "+00:00"))
        hours.append(dt.hour)

print("\nMost active hours (UTC):")
for h, c in Counter(hours).most_common(5):
    print(f"  {h:02d}:00 - {c} posts")
ANALYSIS
```

Expected output:

```
Posts analyzed: 187

Top hashtags:
  #ad: 45
  #sponsored: 32
  #fitness: 28
  #health: 21
  #workout: 18

Most active hours (UTC):
  14:00 - 34 posts
  18:00 - 28 posts
  12:00 - 22 posts
  20:00 - 19 posts
  16:00 - 17 posts
```

### Scenario 2: Digital Forensics — Evidence Preservation

**Objective:** Archive a public Instagram profile as potential evidence with chain-of-custody metadata.

```bash
#!/bin/bash
# forensics_archive.sh — preserve Instagram evidence with timestamps

TARGET="$1"
OUTPUT_DIR="evidence_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

echo "=== Instagram Evidence Archive ===" | tee "$OUTPUT_DIR/chain_of_custody.txt"
echo "Target:     $TARGET" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
echo "Operator:   $(whoami)" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
echo "Timestamp:  $(date -u +%Y-%m-%dT%H:%M:%SZ)" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
echo "Tool:       instaloader $(instaloader --version)" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
echo "Machine:    $(hostname)" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"

instaloader -d "$OUTPUT_DIR" \
    --metadata-json \
    --compress-json \
    --count 1000 \
    "$TARGET" 2>&1 | tee -a "$OUTPUT_DIR/chain_of_custody.txt"

# Generate file hashes for integrity
echo "" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
echo "=== File Integrity Hashes ===" | tee -a "$OUTPUT_DIR/chain_of_custody.txt"
find "$OUTPUT_DIR" -type f -exec sha256sum {} \; > "$OUTPUT_DIR/checksums.sha256"
echo "Checksums saved to $OUTPUT_DIR/checksums.sha256"

echo "Archive complete: $OUTPUT_DIR"
```

### Scenario 3: Brand Monitoring — Hashtag Tracking

**Objective:** Monitor Instagram for brand mentions via hashtags.

```python
#!/usr/bin/env python3
"""Monitor Instagram hashtags for brand mentions."""

import instaloader
import json
from datetime import datetime, timedelta

L = instaloader.Instaloader()

hashtags_to_monitor = ["#brandname", "#brandproduct", "#brandtag"]
monitoring_period = timedelta(days=7)
cutoff = datetime.now(monitoring_period.tzinfo) - monitoring_period

mentions = []

for tag in hashtags_to_monitor:
    try:
        # Instaloader can search hashtags
        posts = L.get_hashtag_posts(tag.lstrip("#"))
        for post in posts:
            if post.date_utc > cutoff:
                mentions.append({
                    "hashtag": tag,
                    "shortcode": post.shortcode,
                    "author": post.owner_username,
                    "date": str(post.date_utc),
                    "caption": post.caption[:200] if post.caption else "",
                    "likes": post.likes,
                    "comments": post.comments
                })
    except Exception as e:
        print(f"Error fetching {tag}: {e}")

# Save report
with open("brand_mentions_report.json", "w") as f:
    json.dump(mentions, f, indent=2)

print(f"Found {len(mentions)} mentions in the last 7 days")
```

Expected output:

```
Found 156 mentions in the last 7 days
```

### Scenario 4: Threat Intelligence — Monitoring Known Actors

**Objective:** Monitor threat actor Instagram accounts for new posts or communications.

```bash
#!/bin/bash
# monitor_actors.sh — watch known threat actor accounts

KNOWN_ACTORS="actors_list.txt"
ALERT_KEYWORDS="weapons|explosives|attack|target|plan"

while read -r actor; do
    instaloader --count 5 --metadata-json --sleep 120 "$actor" 2>/dev/null

    # Check for alert keywords in recent captions
    grep -rliE "$ALERT_KEYWORDS" "$actor/"*.json 2>/dev/null | while read -r f; do
        echo "[ALERT] Keyword match in $actor: $f"
        # Send alert (customize as needed)
        # mail -s "Instagram Alert: $actor" admin@company.com < "$f"
    done
done < "$KNOWN_ACTORS"
```

### Scenario 5: Background Check Supplementation

```bash
# Download all public posts for background check
instaloader --metadata-json --count 500 --comments target_subject

# Generate quick summary
python3 << 'EOF'
import json, glob
from collections import Counter

files = glob.glob("target_subject/*_metadata.json")
posts = []
for f in files:
    with open(f) as fp:
        posts.append(json.load(fp))

print(f"Profile: target_subject")
print(f"Total posts analyzed: {len(posts)}")
print(f"Average engagement: {sum(p.get('likes',0) for p in posts)/len(posts):.0f} likes")

# Extract location mentions
locations = [p.get('location', {}).get('name') for p in posts if p.get('location')]
if locations:
    print(f"Common locations: {Counter(locations).most_common(5)}")

# Extract mentions
mentions = []
for p in posts:
    cap = p.get('caption', '')
    mentions.extend([w[1:] for w in cap.split() if w.startswith('@')])
if mentions:
    print(f"Frequent mentions: {Counter(mentions).most_common(5)}")
EOF
```

---

## Chapter 7: Detection & Defense

### Detection Signatures

Instaloader generates identifiable network traffic and behavioral patterns that defenders can monitor.

#### Network-Level Indicators

```
# Instaloader API request pattern
GET /api/v1/users/web_profile_info/?username=target HTTP/1.1
Host: www.instagram.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...
X-IG-App-ID: 936619743392459
X-Requested-With: XMLHttpRequest

# GraphQL query pattern
POST /graphql/query/ HTTP/1.1
Host: www.instagram.com
Content-Type: application/x-www-form-urlencoded
```

#### Behavioral Indicators

| Indicator | Description |
|-----------|-------------|
| Rapid profile access | Multiple profile page requests in quick succession |
| API endpoint pattern | Repeated calls to `web_profile_info` or `graphql/query` |
| Sequential user IDs | Querying incrementing user IDs |
| Regular request intervals | Mechanical timing patterns (unlike human browsing) |
| Media download patterns | Bulk image/video downloads following profile access |
| Missing browser artifacts | No JavaScript execution, no cookie accumulation patterns |
| User-Agent anomalies | Outdated or generic User-Agent strings |

#### SIEM Detection Rules (Pseudocode)

```
# Rule 1: Excessive Instagram API requests
ALERT WHEN:
    source_ip makes > 50 requests to instagram.com/api/* within 5 minutes
    AND requests target different profile endpoints
THEN:
    FLAG as potential scraping activity

# Rule 2: Sequential profile enumeration
ALERT WHEN:
    source_ip queries instagram.com/api/v1/users/web_profile_info/
    AND username parameter changes every request
    AND > 20 unique usernames queried in 10 minutes
THEN:
    FLAG as profile enumeration

# Rule 3: Bulk media download
ALERT WHEN:
    source_ip downloads > 100 files from scontent-*.cdninstagram.com
    within 30 minutes
THEN:
    FLAG as bulk content scraping
```

#### Log Analysis Commands

```bash
# Search web server logs for Instagram API requests
grep -E 'instagram\.com/api' /var/log/nginx/access.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Check for bulk CDN downloads
grep 'cdninstagram.com' /var/log/nginx/access.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Find rapid profile enumeration
grep 'web_profile_info' /var/log/nginx/access.log | \
    awk '{print $1, $7}' | sort | uniq -c | sort -rn | head -20
```

### Defensive Measures

#### Instagram-Side Defenses

Instagram employs several defenses against automated scraping:

- **Rate limiting**: Limits the number of API requests per IP/session
- **Challenge-response**: Forces CAPTCHA or email verification for suspicious activity
- **Session validation**: Requires valid session cookies for authenticated endpoints
- **Bot detection**: Machine learning models detect automated behavior patterns
- **IP blocking**: Temporarily or permanently blocks IPs exhibiting scraping behavior
- **GraphQL complexity limits**: Limits the depth and breadth of GraphQL queries

#### Network-Level Defenses

```bash
# Implement rate limiting for Instagram-bound traffic
iptables -A OUTPUT -d instagram.com -m limit --limit 30/min -j ACCEPT
iptables -A OUTPUT -d instagram.com -j DROP

# Block known scraping tool User-Agents
iptables -A INPUT -m string --string "instaloader" --algo bm -j DROP
iptables -A INPUT -m string --string "python-requests" --algo bm -j DROP
```

#### Application-Level Defenses

```python
# Example: Flask middleware to detect scraping patterns
from flask import Flask, request
from collections import defaultdict
import time

app = Flask(__name__)
request_counts = defaultdict(list)

@app.before_request
def check_scraping():
    ip = request.remote_addr
    now = time.time()

    # Clean old entries
    request_counts[ip] = [t for t in request_counts[ip] if now - t < 300]

    # Check rate
    if len(request_counts[ip]) > 100:
        return "Rate limit exceeded", 429

    request_counts[ip].append(now)
```

#### Egress Filtering

Monitor outbound connections to Instagram infrastructure:

```bash
# Detect instaloader processes
ps aux | grep -i instaloader

# Monitor network connections to Instagram
ss -tnp | grep -E '157\.240\.[0-9]+\.[0-9]+|instagram'

# Use tcpdump to capture Instagram API traffic
sudo tcpdump -i any -A 'host www.instagram.com and tcp port 443' -w instagram_traffic.pcap
```

---

## Chapter 8: Troubleshooting

### Common Errors and Solutions

#### Error: "Login required"

**Cause:** Trying to access a private profile or stories without authentication.

**Solution:**

```bash
# Login first, then access the profile
instaloader --login your_username private_profile
```

Expected output:

```
Enter Instagram password for your_username:
Login to your_username successful.
Downloading private_profile
private_profile/2024-01-15_14-30-22_UTC_1.jpg
...
```

#### Error: "Rate limit exceeded"

**Cause:** Too many requests in a short period; Instagram has throttled your access.

**Solution:**

```bash
# Add significant delays between requests
instaloader --sleep 120 --max-connection-attempts 5 target_username
```

If rate-limited, wait 15-30 minutes before retrying. For persistent issues:

```bash
# Use a different IP via proxy
instaloader --proxy http://proxy.example.com:8080 --sleep 60 target_username
```

#### Error: "Profile not found"

**Cause:** The username doesn't exist, has been changed, or the account has been deleted.

**Solution:**

```bash
# Verify the profile exists using curl
curl -s -o /dev/null -w "%{http_code}" "https://www.instagram.com/username/"
# Returns 200 if profile exists, 404 if not

# Or check via the API directly
curl -s "https://www.instagram.com/username/?__a=1&__d=dis" | python3 -m json.tool 2>/dev/null | head -5
```

#### Error: "Challenge required"

**Cause:** Instagram has flagged the session as suspicious and requires verification.

**Solution:**

```bash
# Delete the cached session and re-login interactively
rm -f ~/.config/instaloader/your_username.session
instaloader --login your_username target_profile
# Complete the challenge in your browser if prompted
```

#### Error: "401 Unauthorized"

**Cause:** Session has expired or credentials are incorrect.

**Solution:**

```bash
# Remove expired session
rm -f ~/.config/instaloader/your_username.session

# Re-authenticate
instaloader --login your_username target_profile
```

#### Error: "Connection timeout"

**Cause:** Network issues or Instagram is blocking your connection.

**Solution:**

```bash
# Increase timeout and retry attempts
instaloader --request-timeout 60 --max-connection-attempts 5 target_username

# Test basic connectivity first
curl -I https://www.instagram.com/
```

#### Error: No files downloaded despite posts existing

**Cause:** Content filtering flags may be excluding all posts.

**Solution:**

```bash
# Ensure you haven't excluded both media types
instaloader --no-pictures --no-videos target_username  # This downloads NOTHING

# Correct usage — download everything
instaloader target_username
```

### Performance Tips

1. **Use `--count` for initial recon**: Download only recent posts first to assess the target before full download
2. **Batch with delays**: Always add `--sleep` between profiles to avoid rate limits
3. **Cache sessions**: A cached `.session` file avoids repeated logins
4. **Use `--compress-json`**: Reduces disk usage for metadata-heavy downloads
5. **Parallel cautiously**: Running multiple instaloader instances multiplies your request rate — increase delays accordingly

### Verifying Download Integrity

```bash
# Check that all expected files exist
ls -la target_username/ | wc -l

# Verify metadata JSON is valid
for f in target_username/*.json; do
    python3 -c "import json; json.load(open('$f'))" 2>/dev/null && echo "OK: $f" || echo "BROKEN: $f"
done

# Check for zero-byte files (incomplete downloads)
find target_username/ -size 0 -type f
```

---

## Resources

### Official Documentation

- [Instaloader GitHub Repository](https://github.com/instaloader/instaloader) — Source code, issues, and releases
- [Instaloader Documentation](https://instaloader.github.io/) — Official usage docs and API reference
- [Instagram API (Meta for Developers)](https://developers.facebook.com/docs/instagram-api/) — Official API documentation
- [Instaloader PyPI Page](https://pypi.org/project/instaloader/) — Package information and changelog

### Related Tools

| Tool | Purpose | Relation to Instaloader |
|------|---------|------------------------|
| [Sherlock](https://github.com/sherlock-project/sherlock) | Username enumeration across platforms | Discovers Instagram accounts; Instaloader downloads them |
| [Photon](https://github.com/s0md3v/Photon) | Web crawler for OSINT | Crawls websites for Instagram links |
| [Maigret](https://github.com/soxoj/maigret) | Username OSINT across 3000+ sites | Extended username discovery |
| [Holmes](https://github.com/google/inaturalist) | Social media analysis | Complementary analysis |

### Kali Linux Resources

- [Kali Tools — Instaloader](https://www.kali.org/tools/instaloader/) — Kali package page
- [Kali OSINT Guide](https://www.kali.org/docs/introduction-to-osint/) — Kali's OSINT methodology guide

### Learning Resources

- [OSINT Framework](https://osintframework.com/) — Collection of OSINT tools and techniques
- [r/OSINT](https://reddit.com/r/OSINT) — Reddit OSINT community
- [Instaloader GitHub Issues](https://github.com/instaloader/instaloader/issues) — Community troubleshooting and feature requests

---

## CHEATSHEET

### Quick Commands

| Task | Command |
|------|---------|
| Download a profile | `instaloader target_username` |
| Download last 50 posts | `instaloader --count 50 target_username` |
| Download with metadata | `instaloader --metadata-json target_username` |
| Download specific post | `instaloader -- SHORTCODE` |
| Download by URL | `instaloader https://www.instagram.com/p/SHORTCODE/` |
| Download only videos | `instaloader --no-pictures target_username` |
| Download only images | `instaloader --no-videos target_username` |
| Download to directory | `instaloader -d ./output target_username` |
| Login to Instagram | `instaloader --login username` |
| Download with delay | `instaloader --sleep 60 target_username` |
| Download via proxy | `instaloader --proxy http://proxy:8080 target_username` |
| Download profile picture | `instaloader --count 0 target_username` |
| Compress metadata | `instaloader --compress-json target_username` |
| Resume download | `instaloader target_username` (auto-skips existing) |
| Verbose output | `instaloader -V target_username` |

### Python API Quick Reference

```python
import instaloader
L = instaloader.Instaloader()

# Profile info
profile = instaloader.Profile.from_username(L.context, "user")
print(profile.username, profile.followers, profile.mediacount)

# Download profile
for post in profile.get_posts():
    L.download_post(post, target="user")

# Specific post
post = instaloader.Post.from_shortcode(L.context, "SHORTCODE")
L.download_post(post)

# Stories (requires login)
L.login("you", "pass")
stories = L.get_stories(userids=[profile.userid])
```
