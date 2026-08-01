---
title: Supabase
tags: [vendor, api, auth, database]
---

# Supabase

Auth + database backend used by [[Apex Scheduler]], [[New Horizon|Horizon V2]], and [[New Horizon]].

## Per-app usage

- **Apex** — auth, migrations, functions. Pilot launch.
- **Horizon V2** — auth + travel booking data
- **New Horizon** — auth + governance log store + travel offers

## Config pattern

Each app uses `supabase_flutter` with `SUPABASE_URL` and `SUPABASE_ANON_KEY` passed via `--dart-define` or `.env.local`. See each app's `.env.local.example`.