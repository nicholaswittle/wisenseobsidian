---
title: restOS
tags: [archive, restaurant-os, github]
aliases: [Restaurant OS archive, restOS repo]
updated: 2026-07-28
date: 2026-07-27
---

# restOS

**Archive snapshot** of Restaurant OS related projects — not the live source of truth. Refreshed on request as a one-time dump (2026-07-27, again 2026-07-28 morning + afternoon).

| | |
|---|---|
| **GitHub** | https://github.com/nicholaswittle/restOS |
| **Local clone** | `C:\development\projects\restOS` |
| **Latest snapshot** | `a6cb554` on `main` (2026-07-28 afternoon) |
| **Contents** | `apex_v2/` · `jigsys_site/` · `jigsy/` |

## What this refresh includes

- Menu stock / sold-out toggles (Apex + `staff.html`, shared `menu_items.available`)
- Kitchen flow: **Accept & print** only (no Paid & done step)
- Online alcohol (House Brews To-Go) removed
- Staff console live on site; guest catalog realtime for stock/pause

## Rules

- **Canonical sources stay under** `C:\development\projects\` (`apex_v2`, `jigsys_site` / site repo, `jigsy`).
- Archive is a copy only — no edits to originals for the push.
- Regenerable dirs excluded (`.git`, `node_modules`, `build`, `.dart_tool`, etc.).
- Do not put secrets (`supabase/keys.txt`, `.env`) in the archive.

Related: [[Apex v2 — Restaurant OS Build]], [[Jigsy Online Ordering — Live Status 2026-07-28]], [[Jigsys Website Concept]], [[customers/Jigsys Brewpub]], [[business/Restaurant OS Unified Build Plan 2026-07-27]]
