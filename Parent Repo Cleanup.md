---
title: Parent Repo Cleanup
tags: [decision, hygiene, infrastructure]
status: open
---

# Parent Repo Cleanup — C:\development git mess

## The problem

`C:\development\` is itself a git repo whose history is mostly the old my_ai era. Active work lives under `projects/`. Several experiments were deleted 2026-07-19 ([[Abandoned Projects — Lessons]]).

## Current state (2026-07-19)

| Path | Status |
|---|---|
| `projects/apex/apex` | **Own git repo** on `main` / `origin/main` — extraction done |
| `projects/wisense_decompression` | Own repo (COMMS LINK) |
| `projects/wisense_new_horizon` | Own repo |
| `projects/jigsy/` | May still resolve git to parent — verify before big cleans |
| `projects/local-agent-work-center` | Abandoned; may leave locked dirs on disk |
| `projects/wisense-os` | Abandoned; may leave locked dirs on disk |
| `projects/my_ai`, `command_center`, `markdown_practice` | Deleted |

## Risk

A `git clean -fd` or `git reset --hard` at `C:\development` could still damage anything that is only tracked by the parent repo (e.g. jigsy if nested). Prefer operating inside each project's own git root.

## Remaining options

1. **Finish hygiene** — confirm jigsy has its own repo or extract it; archive/remove parent my_ai history; delete locked abandoned folders after reboot/elevated PowerShell.
2. **Formally treat `C:\development` as a workspace root only** (not a product monorepo) — `.gitignore` aggressively; never commit product app code at the parent.

## Status

Partially resolved by Apex having its own repo and agent platforms being abandoned. Open item: jigsy nesting + locked folder deletes.

Related: [[Apex Scheduler]], [[Abandoned Projects — Lessons]], [[COMMS LINK]], [[New Horizon]]
