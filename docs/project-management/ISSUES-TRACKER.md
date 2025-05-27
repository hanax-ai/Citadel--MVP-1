# Issues Tracker & Lessons Learned

## Active Issues
*No active issues currently*

---

## Resolved Issues

### Issue #001: Git Line Ending Conflicts (Windows/Linux)
- **Date:** 2025-05-26
- **Environment:** Windows Git Bash
- **Problem:** `fatal: LF would be replaced by CRLF` preventing commits
- **Root Cause:** Windows/Linux line ending differences
- **Solution:** `git config core.safecrlf false` + `git add --renormalize .`
- **Prevention:** Use consistent development environment (Linux/WSL preferred)
- **Status:** ✅ RESOLVED - DO NOT RETRY original approach

### Issue #002: Unrelated Git Histories
- **Date:** 2025-05-26
- **Environment:** Multiple Git environments (Ubuntu server, Windows, GitHub Desktop)
- **Problem:** `fatal: refusing to merge unrelated histories`
- **Root Cause:** Repository initialized in multiple places independently
- **Solution:** `git pull origin main --allow-unrelated-histories`
- **Prevention:** Always clone from single source, avoid multiple initializations
- **Status:** ✅ RESOLVED - DO NOT RETRY multiple repo initialization

### Issue #003: Empty Directory Git Tracking
- **Date:** 2025-05-26
- **Problem:** Git doesn't track empty directories
- **Solution:** Add README.md files to all directories for tracking
- **Prevention:** Always include at least one file per directory
- **Status:** ✅ RESOLVED - Standard practice established

### Issue #004: Multi-Interface Git Workflow Sync
- **Date:** 2025-05-26
- **Environment:** Git Bash + GitHub Desktop
- **Problem:** Local repository becomes out of sync when using multiple Git interfaces
- **Root Cause:** Changes made in GitHub Desktop not reflected in Git Bash
- **Solution:** Always `git pull origin main` before making changes in Git Bash
- **Prevention:** Establish consistent workflow - sync before switching interfaces
- **Status:** ✅ RESOLVED - Standard practice established

---

## Blocked/Avoided Approaches

### ❌ DO NOT RETRY: Multi-part `cat` commands in Git Bash
- **Issue:** Complex here-documents with multiple parts fail inconsistently
- **Alternative:** Use simple `echo` commands or text editors
- **Reason:** Unreliable EOF detection in Windows Git Bash

### ❌ DO NOT RETRY: Installing packages without sudo
- **Issue:** Cannot install `tree` or other packages without admin rights
- **Alternative:** Use `ls -R` or `find` commands for directory listing
- **Reason:** Limited permissions on development server

---

## Best Practices Established

### ✅ Git Workflow
1. Always `git pull origin main` before starting work
2. Always `git pull origin main` after using GitHub Desktop
3. Use `git status` before commits to verify staged files
4. Include README.md in every directory for Git tracking
5. Use GitHub Desktop for Windows if Git Bash causes issues

### ✅ Documentation
1. Update status documents after each major milestone
2. Track issues and solutions for future reference
3. Maintain change log for project history
4. Document "DO NOT RETRY" items to avoid repeated failures

### ✅ Development Rules
1. Created `DEVELOPMENT-RULES.md` for quick reference
2. Established `git rules` alias for instant access
3. AI assistant automatically enforces established patterns

---

## Template for New Issues

### Issue #XXX: [Brief Description]
- **Date:** YYYY-MM-DD
- **Environment:** [Development environment details]
- **Problem:** [Detailed problem description]
- **Root Cause:** [Analysis of why it occurred]
- **Solution:** [How it was resolved]
- **Prevention:** [How to avoid in future]
- **Status:** [ACTIVE/RESOLVED/BLOCKED]
