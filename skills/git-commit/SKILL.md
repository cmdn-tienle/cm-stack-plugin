---
name: git-commit
description: Create clean git commits without AI signatures - use when the user asks to commit changes
---

# Git Commit Skill

A simple workflow for creating git commits with clean, concise messages and no AI signatures.

## Overview

This skill guides you through creating git commits that:
- Have clear, descriptive commit messages
- Only include relevant changes
- Do NOT include any AI signature or "Co-Authored-By" lines

## Workflow

1. **Check Current State**
   - Run `git status` to see untracked and modified files
   - Run `git diff` to see unstaged changes
   - Run `git diff --staged` to see staged changes
   - Run `git log --oneline -5` to understand recent commit style

2. **Draft Commit Message**
   - Summarize the nature of changes (new feature, bug fix, refactor, etc.)
   - Keep it concise - one line is often enough
   - Focus on WHAT changed and WHY, not HOW

3. **Stage Files**
   - Stage specific files with `git add <file>` (not `git add .` or `-A`)
   - Avoid staging files with secrets, credentials, or unrelated changes

4. **Create Commit**
   - Commit with: `git commit -m "your message here"`
   - Verify with `git status` after committing

## Key Rules

- **NO AI signature** - Never add "Co-Authored-By: Claude" or similar lines
- **NO emoji** unless the user explicitly requests them
- **Stage specific files** - Don't use `git add .` or `git add -A`
- **Ask before committing** if there are unexpected changes or secrets
