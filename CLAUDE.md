# IOC Repository - Claude Code Configuration

## Language / שפה
**Always respond in Hebrew (עברית) only.** The user prefers all communication in Hebrew.

## Project Overview
This repository contains Indicators of Compromise (IOCs) for threat intelligence:
- `Bad_IP_Address.txt` - Malicious IP addresses
- `FQDN_list.txt` - Malicious domains
- `SHA256_list.txt` - SHA256 file hashes
- `MD5_list.txt` - MD5 file hashes
- `SHA1_list.txt` - SHA1 file hashes

## Git Workflow

### Auto-Merge to Main
All changes pushed to `claude/**` branches are **automatically merged to main** via GitHub Actions.

**Workflow:** `.github/workflows/sync-to-main.yml`

**How it works:**
1. Work on branch `claude/<description>-<session-id>`
2. Push changes to the claude branch
3. GitHub Actions automatically merges to main

**No manual merge or PR required.**

### IMPORTANT: Always Verify Merge
**After every push, Claude MUST verify the merge to main:**
```bash
sleep 3 && git fetch origin main && git log origin/main --oneline -3
```
Confirm that the commit appears in main before finishing. If not, push again to trigger the workflow.

### Branch Naming Convention
```
claude/<feature>-<session-id>
```
Example: `claude/add-iocs-ElieQ`

## Automated IOC Fetching

**Workflow:** `.github/workflows/auto-fetch-iocs.yml`

Runs daily at 06:00 UTC (09:00 Israel Time) and fetches IOCs from:
- **URLhaus** - Malicious URLs/domains/IPs
- **MalwareBazaar** - SHA256, MD5, SHA1 hashes
- **ThreatFox** - Domains, IPs, hashes

New IOCs are deduplicated and added automatically.

## Adding IOCs Manually

When adding IOCs, use this format at the top of each file (after line 11):

```
# Source Name
# Description
# YYYY-MM-DD

<IOC values, one per line>
```

## Summary Table Requirement

**After adding IOCs, Claude MUST display a summary table in Hebrew:**

| סוג IOC | קובץ | כמות |
|---------|------|------|
| IP Addresses | Bad_IP_Address.txt | X |
| Domains | FQDN_list.txt | X |
| SHA256 | SHA256_list.txt | X |
| MD5 | MD5_list.txt | X |
| SHA1 | SHA1_list.txt | X |
| **סה"כ** | | **X** |

Include source name, date, and threat level when available.

## Commands

```bash
# Push changes (triggers auto-merge to main)
git push -u origin claude/<branch-name>

# Manually trigger IOC fetch
gh workflow run auto-fetch-iocs.yml
```
