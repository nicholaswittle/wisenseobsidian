---
title: Apex Scheduler
tags: [app, launch-priority, apex]
aliases: [jigsy, jigsy-schedule]
updated: 2026-07-28
---

# Apex Scheduler

Mobile staff scheduling for Jigsy's Brewpub — shifts, swaps, time clock, sidework, labor cost.

| | |
|---|---|
| **Repo** | `C:\development\projects\apex\apex` — **own git repo** |
| **GitHub** | `github.com/nicholaswittle/apex` |
| **Active branch** | `feat/apex-plan-2026-07-21` (Assign Days @ `03a62e6`) |
| **Web** | https://apex-scheduler-theta.vercel.app (static `build/web` deploy) |
| **Bundle ID** | `com.wisense.apex` |
| **Stack** | Flutter · Supabase · Firebase Cloud Messaging · Sentry |
| **Platforms** | iOS · Android · Web |
| **Phase** | Assign Days live on web → apply RLS → Android packaging → Jigsy pilot |

## Features

Shift calendar, **Assign Days** (person → hours → month tap days; primary owner publish UX), swap board with owner approval, sidework checklists, time-off workflow, time clock + CSV export, org invite codes, push + in-app notifications. Billing (Stripe) deferred for pilot — all features unlocked.

### Assign Days (2026-07-27)

Ported from Apex v2 pattern into v1:
- `lib/features/schedule/assign_days_panel.dart`
- `lib/widgets/admin_publish_panel.dart` rewritten to use Assign Days as primary publish UX
- Wired from `lib/calendar_page.dart`
- Commit `03a62e6` pushed on `feat/apex-plan-2026-07-21`

## Launch status (2026-07-28)

**In-repo: done & pushed** — merge `13971b0`, races+timezone `9f723a5`, RLS migration `a66b039` (`20260720000000_launch_blockers_rls.sql`), Assign Days `03a62e6`. Analyze clean; tests green. Vercel web live.

### Remaining (priority order)

1. [ ] **Human:** Apply RLS migration on Supabase **staging → smoke-test → prod** (default-deny; needs Nicholas DB access)
2. [ ] Fill `JIGSYS_BASELINE.md` with owner metrics by Day 3 of pilot — log truth in [[customers/Jigsys Brewpub]]
3. [ ] Gate C Android packaging — [[output/Gate C — Android Packaging & Store Listings 2026-07-20]]
4. [ ] 7-day clean run per `GATE0.md` once pilot live
5. [ ] Merge `feat/apex-plan-2026-07-21` → `main` after iPhone QA

Weekly board: [[NOW]].

## Audit artifact

- Full write-up: `apex/audit/AUDIT_2026-07-19.md`
- Vault summary: [[output/Apex Security Audit 2026-07-19]]
- Launch plan: [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]]

Related: [[Apex Scheduler — Code Reference]], [[Parent Repo Cleanup]], [[Audit Findings Loop]], [[Supabase]], [[customers/Jigsys Brewpub]]
