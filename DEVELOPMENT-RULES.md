# Development Rules & Quick Checklist

## Git Workflow Rules
- ✅ ALWAYS `git pull origin main` before starting work
- ✅ ALWAYS `git pull origin main` after using GitHub Desktop
- ✅ Use `git status` before commits to verify staged files
- ❌ NEVER use multi-line `cat` commands in Git Bash
- ❌ NEVER initialize repos in multiple places

## File Creation Rules
- ✅ Use simple `echo` commands for short files
- ✅ Use text editors for complex files
- ✅ Include README.md in every directory
- ❌ NEVER use here-documents (<<'EOF') in Git Bash

## Pre-Work Checklist
1. [ ] `git pull origin main`
2. [ ] `git status` (should be clean)
3. [ ] Verify current branch: `git branch`

## Post-Work Checklist
1. [ ] `git status` (verify changes)
2. [ ] `git add .` or specific files
3. [ ] `git commit -m "descriptive message"`
4. [ ] `git push`
