---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-28T03:20:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-28 (late). Apex v2 PILOT READY. All features built, audit fixes applied, self-service auth + super admin console built by Cursor. Vercel deployed with real Supabase keys. Emily saw v2 layout, liked it better than v1. Tip function changed to server self-report log (what she wanted). Super admin console lets Nicholas manage all orgs from one screen — never open Supabase again. Next: rebuild Vercel with latest code (Cursor committed but didn't redeploy), then real pilot test with Emily.

## Key Recent Facts
- **Vault = curated static reference** — no auto-wiki. Boot: [[hot]] → [[NOW]] → [[index]] → note.
- **Apex v2 PILOT READY:** `C:\development\projects\apex_v2` · `github.com/nicholaswittle/apex_v2` · HEAD `49d624d` on `main` (synced). Live: https://apex-v2-ten.vercel.app (real Supabase, real keys). Full status: [[Apex v2 — Restaurant OS Build]].
- **Same Supabase as v1:** `pqkremkwfkudrhtxasdj`. ALL migrations applied (9 migrations + audit fixes + self-service auth + super admin + server_tips). Schema cache may need refresh for admin_set_tier/admin_toggle_module RPCs.
- **Self-service auth:** Sign up creates org + profile automatically. Join via invite code. Owner promotes staff in Team tab. No Supabase dashboard needed ever again.
- **Super admin console:** Nicholas can see all orgs, change tiers (Free/Pro/OS/Multi), toggle modules, create orgs, generate invites — all from Admin tab in the app. Set via `is_super_admin = true` on profiles.
- **Server tip log:** Emily wanted servers to self-report tips daily (not owner-side pool split). Built server_tip_log.dart — servers enter cash + card per shift, see week summary, owners see audit comparison (declared vs pool split). Tip pool splitter removed from nav (code kept in repo for Pro tier later).
- **Audit fixes applied:** Revenue filter (only completed/accepted orders count), server_tips RLS hole (added is_member check), daily_revenue RLS (manager-only insert), demo seed data for server_tips + daily_revenue.
- **Test user:** test-delete2@wisensellc.com / test123456 — Owner of jigsys org, super admin, free tier + tipManagement module.
- **Theme:** New Horizon dark palette (purple/teal on blue-black).
- **Apex v1:** Assign Days ported by Cursor (month-grid calendar). Still uncommitted in v1 repo.
- **Audits:** [[wisense/projects/APEX_V2_OS_FULL_AUDIT_2026-07-28]] (full system, 2 blockers + 4 high — blockers fixed) · [[wisense/projects/APEX_V2_AUDIT_2026-07-27]] (code) · [[wisense/projects/RESTOS_FULL_SYSTEM_AUDIT_2026-07-27]] (archived restOS).
- Git mirror: `github.com/nicholaswittle/wisenseobsidian` (origin/main).

## Active Project Status
- **Apex v2** — PILOT READY. Self-service auth + admin console + server tip log + audit fixes all built. **NEXT:** rebuild Vercel deploy (Cursor committed code but old build still live), then real pilot test with Emily. [[Apex v2 — Restaurant OS Build]].
- **Apex Scheduler (v1)** — Assign Days ported (uncommitted); ship-to-stores Friday (keystore + accounts). [[Apex Scheduler]].
- **Jigsy's Website Concept** — live. Ordering also in Apex v2 now. [[Jigsys Website Concept]] · [[customers/Jigsys Brewpub]].
- **COMMS LINK** — ⏸️ PARKED 2026-07-20. [[COMMS LINK]].
- **New Horizon** — 117/117; fork reconciliation COMPLETE.

## Active Threads
- **REBUILD:** Vercel deploy is stale — needs `flutter build web` + `vercel deploy --prod` with real keys. Cursor committed code but didn't rebuild.
- **PILOT:** Real test with Emily on apex-v2-ten.vercel.app. Create her real account via app signup (self-service auth now works), promote to Owner, set tier.
- **LAUNCH:** Ship Apex v1 to stores (keystore + accounts — payday Friday).
- **COMMIT:** Apex v1 Assign Days changes still uncommitted in v1 repo.
- **REMAINING AUDIT ITEMS:** Order flooding DoS (needs rate limiting in edge function), shifts.staff string name (biggest structural debt), check-capacity cron enable, mobile nav bar overflow on small screens.
- **This week board:** [[NOW]]
