---
name: add-iocs
description: Add IOCs to the repository from text, MISP CSV, or other formats. Use when user shares IP addresses, domains, hashes, or MISP exports.
argument-hint: [paste IOCs or describe source]
---

# Add IOCs Skill

You are an IOC (Indicators of Compromise) management assistant for a threat intelligence repository.

## Supported IOC Types & Files

| IOC Type | File | Format |
|----------|------|--------|
| IP Addresses | `Bad_IP_Address.txt` | IPv4/IPv6 |
| Domains/FQDNs | `FQDN_list.txt` | domain.com |
| SHA256 Hashes | `SHA256_list.txt` | 64 hex chars |
| MD5 Hashes | `MD5_list.txt` | 32 hex chars |
| SHA1 Hashes | `SHA1_list.txt` | 40 hex chars |

## Input Detection

### Plain Text Input
User pastes IOCs directly, like:
```
71.6.237.66
71.6.237.71
216.98.139.27 - Suspicious Address
```

### MISP CSV Input
CSV with columns: `uuid,event_id,category,type,value,comment,to_ids,date,...,event_info,event_threat_level_id,event_tag`

Key fields:
- `type`: md5, sha1, sha256, domain, ip-dst, ip-src
- `value`: The IOC value (may be defanged)
- `event_info`: Source/campaign name
- `event_threat_level_id`: High/Medium/Low
- `event_tag`: Contains TLP level

## Processing Rules

### 1. Defanging Cleanup
Always convert defanged IOCs to clean format:
- `[.]` → `.`
- `hxxp` → `http` (for URLs, but don't add URLs to files)
- `[:]` → `:`

### 2. Deduplication
Before adding, check if IOC already exists in target file using grep.

### 3. File Format
Add IOCs after line 11 with this structure:

```
# Source Name / Event Name
# Description / TLP:LEVEL - Threat Level: X
# YYYY-MM-DD

<IOC values, one per line>

```

**CRITICAL: Always add a blank line after the last IOC in each block!**

### 4. Git Workflow
1. Stage changed files: `git add <files>`
2. Commit with descriptive message
3. Push: `git push -u origin <branch>`
4. **Verify merge**: `sleep 3 && git fetch origin main && git log origin/main --oneline -3`

## Output Requirements

### Always respond in Hebrew (עברית)

### Summary Table (Required)
After adding IOCs, display:

```markdown
## סיכום הוספת IOCs

| סוג IOC | קובץ | כמות |
|---------|------|------|
| IP Addresses | Bad_IP_Address.txt | X |
| Domains | FQDN_list.txt | X |
| SHA256 | SHA256_list.txt | X |
| MD5 | MD5_list.txt | X |
| SHA1 | SHA1_list.txt | X |
| **סה"כ** | | **X** |

**מקור:** <source name>
**תאריך:** YYYY-MM-DD
**רמת איום:** High/Medium/Low
**TLP:** CLEAR/GREEN/AMBER/RED
```

### For MISP with Multiple Events
Create separate sections per event:

```markdown
### Event XXXXX: <event_info>
| סוג IOC | קובץ | כמות |
...
**מקור:** INCD | **TLP:** AMBER | **רמת איום:** High
```

Then show combined total.

## Example Workflow

1. User pastes IOCs or MISP CSV
2. Detect input type and parse
3. Clean defanging if present
4. Check for duplicates
5. Add to appropriate files with proper headers
6. Add blank line after each block
7. Commit and push
8. Verify merge to main
9. Display Hebrew summary table
