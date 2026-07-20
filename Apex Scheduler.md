---
title: Apex Scheduler
tags: [app, launch-priority, apex]
aliases: [jigsy, jigsy-schedule]
---

# Apex Scheduler

Mobile staff scheduling for Jigsy's Brewpub — shifts, swaps, time clock, sidework, labor cost.

| | |
|---|---|
| **Repo** | `C:\development\projects\apex\apex` — **own git repo** (`main` tracking `origin/main`) |
| **Bundle ID** | `com.wisense.apex` |
| **Stack** | Flutter · Supabase · Firebase Cloud Messaging · Sentry |
| **Platforms** | iOS · Android · Web |
| **Phase** | Stabilize Jigsy's pilot before new features |

## Features

Shift calendar, swap board with owner approval, sidework checklists, time-off workflow, time clock + CSV export, org invite codes, push + in-app notifications. Billing (Stripe) deferred for pilot — all features unlocked.

## Launch blockers (priority order)

1. [ ] **Security / tenancy** — see [[Apex Security Audit 2026-07-19]]
   - Commit missing RLS migrations; verify live on Supabase
   - Org-scope every stream/query (defense in depth)
   - Atomic open-shift claim; clock-in duplicate guard; UTC timestamps
2. [ ] Resolve any remaining merge/analyzer issues if still present after current branch work
3. [ ] Run `gate0_verify.sh` + `flutter analyze` + `flutter test`
4. [ ] Fill `JIGSYS_BASELINE.md` with owner metrics by Day 3
5. [ ] 7-day clean run per `GATE0.md`

## Audit artifact

- Full write-up: `apex/audit/AUDIT_2026-07-19.md`
- Vault summary: [[Apex Security Audit 2026-07-19]]

Related: [[Apex Scheduler — Code Reference]], [[Parent Repo Cleanup]], [[Audit Findings Loop]], [[Supabase]]
