# IOC Repository Management Guide

## Overview
This repository contains Indicators of Compromise (IOC) lists for threat intelligence. Claude assists with adding new IOCs from various sources (CSV, plain text, images).

## Files Structure
- `Bad_IP_Address.txt` - Malicious IPv4/IPv6 addresses
- `FQDN_list.txt` - Malicious domains
- `MD5_list.txt` - MD5 hashes (32 characters)
- `SHA1_list.txt` - SHA1 hashes (40 characters)
- `SHA256_list.txt` - SHA256 hashes (64 characters)

## Adding IOCs - Format
All INCD entries must follow this exact format:
```
# INCD - [Event Name]
# [Date in YYYY-MM-DD]

[IOC data, one per line]

```
**Important:** Always insert at line 13 (after the header, before Auto-Fetch entries).

## Workflow for Adding IOCs

### Step 1: Parse Input
- CSV (MISP format): Extract `type` and `value` columns
- Plain text: Identify IOC types by pattern
- Images: Read and extract IOCs
- Defang indicators: Replace `[.]` with `.`, `hxxp` with `http`

### Step 2: Check for Duplicates
Files are too large to read entirely. Always use grep:
```bash
grep -n "IOC_VALUE" /home/user/list/[FILE].txt
```

### Step 3: Skip These Items
**Legitimate services (never add):**
- amazonaws.com, cloudfront.net (AWS)
- github.com, githubusercontent.com
- discord.com, discordapp.com
- medium.com, dev.to
- cloudinary.com
- google.com, googleapis.com
- microsoft.com, azure.com
- claude.ai, anthropic.com
- Any other well-known legitimate service

**Skip when requested:**
- IPv6 addresses
- .onion URLs

**Always skip:**
- Duplicate IOCs (already in database)

### Step 4: Edit Files
1. Read the first 20-25 lines of the target file
2. Insert new INCD block before existing INCD entries (at line 13)
3. Use Edit tool to add the new content

### Step 5: Commit and Push
```bash
git add [modified files]
git commit -m "Add INCD IOCs: [Event Name] ([Date])

- X IP addresses
- X domains
- X MD5 hashes
- X SHA1 hashes
- X SHA256 hashes

https://claude.ai/code/session_[ID]"

git push -u origin claude/[branch-name]
```

### Step 6: Display Summary Table
Always show a summary table after adding IOCs:

```
## [Event Name] - IOC Report ([Date])

| Type | Count |
|------|-------|
| IPs | X |
| Domains | X |
| MD5 | X |
| SHA1 | X |
| SHA256 | X |
| **Total** | **X** |

### Added:
- list of added items...

### Skipped (duplicates/legitimate):
- list of skipped items with reason...
```

## Git Workflow
- Branch naming: `claude/**` (triggers auto-merge GitHub Action)
- Never push directly to `main` (protected branch)
- GitHub Action automatically merges `claude/**` branches to main

## Domain Extraction from URLs
When given a URL, extract only the domain:
- `https://malicious.com/path/file` → `malicious.com`
- `http://sub.domain.com:8080/page` → `sub.domain.com`

## IOC Type Detection
- **IPv4**: `X.X.X.X` pattern (4 octets, 0-255)
- **IPv6**: Contains `:`, longer format
- **MD5**: 32 hex characters
- **SHA1**: 40 hex characters
- **SHA256**: 64 hex characters
- **Domain**: FQDN without protocol/path

## Language
User communicates in Hebrew and English. Respond in the same language used.

## Quick Commands
- Check if IOC exists: `grep -n "VALUE" /home/user/list/[FILE].txt`
- Count total IOCs: `wc -l` on each file, sum results
- View recent additions: `head -50 /home/user/list/[FILE].txt`
