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
- **After every push, verify merge:**
  ```bash
  sleep 3 && git fetch origin main && git log origin/main --oneline -3
  ```

## Adding IOCs

### Format (insert after line 11)
```
# Source Name
# Description / TLP level
# YYYY-MM-DD

<IOC values>

```
**IMPORTANT: Always add blank line after each IOC block!**

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
