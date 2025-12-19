# IOC Repository - Claude Automation Guide

## Repository Overview
This repository stores Indicators of Compromise (IOCs) from various threat intelligence sources, including phishing domains, malicious IPs, and file hashes.

## File Structure

```
/
├── Bad_IP_Address.txt     # Malicious IP addresses (IPv4/IPv6)
├── FQDN_list.txt          # Malicious domains, FQDNs, and URLs
├── MD5_list.txt           # MD5 file hashes
├── SHA1_list.txt          # SHA1 file hashes
├── SHA256_list.txt        # SHA256 file hashes
└── .github/
    └── workflows/
        └── auto-merge-claude.yml  # Auto-merge workflow for claude/* branches
```

## Automated Workflow

### How Auto-Merge Works
The repository has a GitHub Actions workflow (`auto-merge-claude.yml`) that automatically merges changes from `claude/*` branches to `main`.

**Workflow Configuration:**
```yaml
name: Auto-merge Claude branches to main

on:
  push:
    branches:
      - 'claude/*'

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    steps:
      - Checkout repository
      - Configure Git
      - Merge to main with --no-ff
      - Push to main
      - Notify on success/failure
```

**Key Features:**
- ✅ Automatic merge for ALL `claude/*` branches
- ✅ No manual session ID updates needed
- ✅ Creates merge commits with branch name
- ✅ Success/failure notifications

## IOC File Format

Each IOC file follows this structure:

### Example - FQDN_list.txt:
```
###############################################################################
#                                                                             #
#              MALICIOUS DOMAINS & FQDN DATABASE                              #
#           Phishing, Malware, C2 Domains and FQDNs                           #
#                                                                             #
###############################################################################

# Source attribution and date
# YYYY-MM-DD

malicious-domain.com
phishing-site.net
https://bad-url.com/path

# Next batch
# Source - @username
# YYYY-MM-DD

another-domain.com
```

### Example - Bad_IP_Address.txt:
```
###############################################################################
#                                                                             #
#                   MALICIOUS IP ADDRESSES DATABASE                           #
#                  Command & Control / Malicious IPs                          #
#                                                                             #
###############################################################################

# Source attribution
# YYYY-MM-DD

192.168.1.1
10.0.0.1
```

### Example - SHA256_list.txt:
```
# Source attribution - @username
# YYYY-MM-DD

30496f3cae78dae238ac5e7a4512f119ea29f835531775ff92d7c8b74a26295c
fc31793647211892bcb40db21f51a07f3b552804e76d71803dbb11aca7d375b7
```

## Workflow for Adding New IOCs

### Step-by-Step Process:

1. **Work on a claude/* branch**
   - Branch naming: `claude/add-<description>-<SESSION_ID>`
   - Example: `claude/add-phishing-iocs-JY2xT`

2. **Add IOCs to appropriate files:**
   - Domains/URLs → `FQDN_list.txt`
   - IP addresses → `Bad_IP_Address.txt`
   - MD5 hashes → `MD5_list.txt`
   - SHA1 hashes → `SHA1_list.txt`
   - SHA256 hashes → `SHA256_list.txt`

3. **Format each addition with:**
   ```
   # Source attribution (e.g., @romonlyht, @malwrhunterteam)
   # Date (YYYY-MM-DD)

   <IOC entries>
   ```

4. **Append to end of file:**
   ```bash
   cat >> FQDN_list.txt << 'EOF'

   # Source - @username
   # 2025-12-19

   malicious-domain.com
   phishing-site.net
   EOF
   ```

5. **Commit with descriptive message:**
   ```bash
   git add <files>
   git commit -m "Add IOCs from @source (YYYY-MM-DD)

   Added X entries to FQDN_list.txt:
   - domain1.com (@source)
   - domain2.com (@source)

   Added Y entries to Bad_IP_Address.txt:
   - IP1 (@source)
   - IP2 (@source)"
   ```

6. **Push to trigger auto-merge:**
   ```bash
   git push -u origin claude/<branch-name>
   ```

7. **Automatic merge to main**
   - GitHub Actions workflow detects push to `claude/*`
   - Automatically merges to `main` branch
   - Creates merge commit
   - Changes are live in main within seconds

## Git Branch Requirements

### Branch Naming Convention:
- **Pattern:** `claude/*`
- **Examples:**
  - `claude/add-phishing-iocs-ABC123`
  - `claude/update-malware-hashes-XYZ789`
  - `claude/add-c2-domains-QWE456`

### Push Operations:
```bash
# Always use -u flag for first push
git push -u origin claude/<branch-name>

# Branch must start with 'claude/' and end with session ID
# Otherwise push will fail with 403 error

# Retry logic for network errors:
# - Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
```

## Common IOC Sources

- `@romonlyht` - Phishing indicators
- `@malwrhunterteam` - Malware hashes
- `@urldna_bot` - URL/domain intelligence
- `@smica83` - Various threat indicators

## Example: Adding New IOCs

### Given IOCs:
```
IOCs, [19 בדצמ׳, 2025 ב-07:53]
malicious-domain.com
@romonlyht | domain | #phishing

IOCs, [19 בדצמ׳, 2025 ב-07:53]
192.168.1.1
@romonlyht | ip | #phishing

IOCs, [19 בדצמ׳, 2025 ב-08:23]
abc123def456...
@malwrhunterteam | sha256 |
```

### Commands:
```bash
# 1. Add domains
cat >> FQDN_list.txt << 'EOF'

# Phishing IOCs - @romonlyht
# 2025-12-19

malicious-domain.com
EOF

# 2. Add IPs
cat >> Bad_IP_Address.txt << 'EOF'

# Phishing IOCs - @romonlyht
# 2025-12-19

192.168.1.1
EOF

# 3. Add hashes
cat >> SHA256_list.txt << 'EOF'

# IOCs from @malwrhunterteam
# 2025-12-19

abc123def456...
EOF

# 4. Commit
git add FQDN_list.txt Bad_IP_Address.txt SHA256_list.txt
git commit -m "Add phishing IOCs from @romonlyht (2025-12-19)"

# 5. Push (auto-merge will trigger)
git push -u origin claude/add-phishing-iocs-<SESSION_ID>
```

## Important Notes

### Auto-Merge Workflow:
- ✅ Merges ALL `claude/*` branches automatically
- ✅ No manual PR creation needed
- ✅ No workflow updates for new sessions
- ✅ Creates merge commits for traceability

### Best Practices:
1. Always include source attribution in comments
2. Always include date (YYYY-MM-DD format)
3. Group IOCs by source and date
4. Use descriptive commit messages
5. Append to end of files (don't modify existing entries)
6. Work on `claude/*` branches only

### Branch Workflow:
- **Development:** `claude/*` branches
- **Production:** `main` branch (auto-merged)
- **No direct pushes to main** (only via auto-merge workflow)

## Current Status

- ✅ Auto-merge workflow configured for `claude/*` pattern
- ✅ All future `claude/*` branches will auto-merge to main
- ✅ IOC files structured with proper headers and attribution
- ✅ Git operations configured with proper retry logic

---

**Last Updated:** 2025-12-19
**Workflow Version:** Auto-merge all claude/* branches
