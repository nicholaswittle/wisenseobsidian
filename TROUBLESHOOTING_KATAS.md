---
title: Troubleshooting Katas & Error Recovery
tags: [troubleshooting, errors, fixes, recovery, recipes]
aliases: [Troubleshooting Index, Error Katas]
---

# 🛠 TROUBLESHOOTING KATAS & ERROR RECOVERY RECIPES

A catalog of known error signatures across Windows, PowerShell, Python, and Flutter, with exact 1-line resolution commands for AI coding agents.

---

## 1. PowerShell Script Execution & Parsing Errors

### Error: `The string is missing the terminator: "` or `The ampersand (&) character is not allowed.`
- **Cause**: PowerShell script contains non-ASCII quotes (`“`, `”`) or unquoted ampersands.
- **Fix**: Re-save `.ps1` file as standard ASCII with plain quotes (`"`) and wrapped ampersands (`"&"`).

### Error: `Execution of scripts is disabled on this system.`
- **Fix**: Run script with bypass policy:
  ```powershell
  powershell -ExecutionPolicy Bypass -File .\script.ps1
  ```

---

## 2. Python & Pytest Lock Errors

### Error: `PermissionDenied: Access to .pytest_cache is denied` or `database is locked`
- **Cause**: Background pytest or python engine process is holding an active file handle.
- **Fix**: Kill background processes on port 5050 and python:
  ```powershell
  Get-NetTCPConnection -LocalPort 5050 -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force -ErrorAction SilentlyContinue }
  ```

---

## 3. Flutter & Dart Build Errors

### Error: `Undefined name 'defaultOrganizationId'` (Apex)
- **Cause**: Stale parameter name in `staff_repository.dart`.
- **Fix**: Replace `defaultOrganizationId` with `organizationId` or `orgId`.

### Error: `Flutter pubget dependency conflict`
- **Fix**: Clear package cache and re-fetch:
  ```powershell
  flutter clean
  flutter pub get
  ```

---

## 4. Git Merge Conflict Recovery

### Error: `Unmerged paths (both modified / both added)`
- **Fix**: Inspect conflict files, accept incoming changes, and commit:
  ```powershell
  git status
  git add <resolved-file>
  git commit -m "fix: resolve merge conflicts"
  ```

Related: [[00_AI_AGENT_MANIFEST]], [[ENVIRONMENT_MAP]]
