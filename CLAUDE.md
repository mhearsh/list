# IOC Repository

## Language
**Always respond in Hebrew (עברית) only.**

## Files
| File | Content |
|------|---------|
| `Bad_IP_Address.txt` | Malicious IPs |
| `FQDN_list.txt` | Malicious domains |
| `SHA256_list.txt` | SHA256 hashes |
| `MD5_list.txt` | MD5 hashes |
| `SHA1_list.txt` | SHA1 hashes |

## Git Workflow
- Push to `claude/**` branches → **auto-merged to main** via GitHub Actions
- **CRITICAL: Before adding IOCs, ALWAYS sync with main:**
  ```bash
  git fetch origin main
  git merge origin/main --no-edit
  ```
- **Check duplicates against origin/main (NOT local files!):**
  ```bash
  git show origin/main:<filename> | grep -F "<IOC>"
  ```
- **After every push, verify merge:**
  ```bash
  sleep 5 && git fetch origin main && git log origin/main --oneline -5
  git show origin/main:<file> | grep "<IOC>"  # Confirm IOC in main
  ```

## Adding IOCs

### Format (insert after line 11 — ALWAYS at the TOP, line 12+)

**⚠️ NEVER append to end of file! Always INSERT at line 12 using Python:**
```python
with open('<filename>', 'r') as f:
    lines = f.readlines()

new_block = ['\n', '# Source Name\n', '# Description / TLP level\n', '# YYYY-MM-DD\n', '\n', '<IOC1>\n', '\n']
new_content = lines[:11] + new_block + lines[11:]

with open('<filename>', 'w') as f:
    f.writelines(new_content)
```

**CRITICAL FORMATTING RULES:**
1. **Blank line BEFORE the # header** - separates from previous block
2. **Blank line AFTER the IOC values** - separates from next block
3. Every IOC block must be surrounded by blank lines!
4. **New entries always go at line 12 (top of data), NOT at end of file**

### Defanging
Clean before adding: `[.]` → `.`, `[:]` → `:`

### Summary Table (Required)
After adding IOCs, display Hebrew summary:

| סוג IOC | קובץ | כמות |
|---------|------|------|
| IP Addresses | Bad_IP_Address.txt | X |
| Domains | FQDN_list.txt | X |
| SHA256 | SHA256_list.txt | X |
| MD5 | MD5_list.txt | X |
| SHA1 | SHA1_list.txt | X |
| **סה"כ** | | **X** |

Include: source, date, threat level, TLP when available.

## Skills
- `/add-iocs` - Add IOCs from text or MISP CSV

## Commands
```bash
git push -u origin claude/<branch>     # Push (triggers auto-merge)
gh workflow run auto-fetch-iocs.yml    # Manual IOC fetch
```
