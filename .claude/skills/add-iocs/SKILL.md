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

### 0. CRITICAL: Sync with Main First
**ALWAYS fetch and check against origin/main before adding IOCs:**
```bash
git fetch origin main
git show origin/main:<filename> | grep "<IOC>"
```
This ensures you check for duplicates against the latest main branch, not local files.

### 1. Defanging Cleanup
Always convert defanged IOCs to clean format:
- `[.]` → `.`
- `hxxp` → `http` (for URLs, but don't add URLs to files)
- `[:]` → `:`

### 2. Deduplication
Before adding, check if IOC already exists in **origin/main** files using:
```bash
git show origin/main:Bad_IP_Address.txt | grep -F "<IOC>"
```

### 3. File Format — CRITICAL: Insert at Line 12 (After Line 11), NOT Append!

**⚠️ NEVER use `cat >>` or append to the end of the file. Always INSERT at line 12.**

Use Python to insert the new block right after line 11:

```python
with open('<filename>', 'r') as f:
    lines = f.readlines()

new_block = [
    '\n',
    '# Source Name / Event Name\n',
    '# Description / TLP:LEVEL - Threat Level: X\n',
    '# YYYY-MM-DD\n',
    '\n',
    'ioc_value_1\n',
    'ioc_value_2\n',
    '\n',
]

# Insert after line 11 (index 11), before existing content
new_content = lines[:11] + new_block + lines[11:]

with open('<filename>', 'w') as f:
    f.writelines(new_content)
```

**Result: new IOCs appear at lines 12-13+, existing content follows.**

Block structure:
```
(blank line)
# Source Name / Event Name
# Description / TLP:LEVEL - Threat Level: X
# YYYY-MM-DD

<IOC values, one per line>

```

**CRITICAL formatting rules:**
1. **Blank line BEFORE the `#` header** — separates from previous block
2. **Blank line AFTER the IOC values** — separates from next block
3. Insert BEFORE existing content, not after

### 4. Git Workflow
**IMPORTANT: Always sync with main before and after changes!**

**Before adding IOCs:**
```bash
git fetch origin main
git merge origin/main --no-edit  # Sync local branch with main
```

**After adding IOCs:**
1. Stage changed files: `git add <files>`
2. Commit with descriptive message
3. Push: `git push -u origin <branch>`
4. If push fails with "fetch first", run: `git pull origin <branch> --rebase && git push`
5. **Verify merge to main:**
```bash
sleep 5 && git fetch origin main && git log origin/main --oneline -5
```
6. **Confirm IOCs in main:**
```bash
git show origin/main:<filename> | grep "<one of the IOCs>"
```

**If merge conflict occurs:**
- Resolve conflicts by keeping both versions
- Remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
- Commit the resolution and push again

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

1. **FIRST: Fetch and sync with main**
   ```bash
   git fetch origin main
   git merge origin/main --no-edit
   ```
2. User pastes IOCs or MISP CSV
3. Detect input type and parse
4. Clean defanging if present
5. **Check for duplicates against origin/main** (NOT local files!)
   ```bash
   git show origin/main:<file> | grep -F "<IOC>"
   ```
6. Add to appropriate files with proper headers
7. Add blank line after each block
8. Commit and push to claude/** branch
9. Handle any push failures (pull --rebase if needed)
10. **Verify merge to main** - wait and check:
    ```bash
    sleep 5 && git fetch origin main
    git log origin/main --oneline -5
    git show origin/main:<file> | grep "<IOC>"  # Confirm IOC exists
    ```
11. Display Hebrew summary table
