---
title: Environment Map & Port Registry
tags: [environment, ports, keys, config, setup]
aliases: [Port Registry, Environment Map]
---

# ENVIRONMENT MAP & PORT REGISTRY

Canonical reference for network ports, local services, environment variables, and authentication tokens across active WiSense projects.

---

## Local Port Registry

| Service / App | Host | Port | Endpoint / Health Probe | Purpose |
|---|---|---|---|---|
| **Ollama** | `127.0.0.1` | `11434` | `GET /api/tags` | Local / Ollama-cloud LLM inference ([[Working Stack — Claude CLI and Ollama]]) |
| **Supabase Local CLI** | `127.0.0.1` | `54321` | `http://127.0.0.1:54321/health` | Local Supabase DB & Auth (when used) |
| **Flutter Web Dev Server** | `127.0.0.1` | `8080` | `http://127.0.0.1:8080` | Local web testing for Apex / Horizon |
| **Dart VM Debugger** | dynamic | — | DevTools URI from `flutter run` | Debugger & profiler |

### Obsolete (do not assume running)

| Service | Port | Notes |
|---|---|---|
| WiSense OS Engine | `5050` | Project abandoned 2026-07-19 — see [[Abandoned Projects — Lessons]] |

---

## Environment Variables & Security Tokens

### 1. Apex Scheduler (`C:\development\projects\apex\apex`)

- `SUPABASE_URL` / `SUPABASE_ANON_KEY` — via `--dart-define` / `.env` patterns used by the app
- `SENTRY_DSN` — crash reporting
- FCM / Firebase — push (see project docs)

### 2. New Horizon (`C:\development\projects\wisense_new_horizon`)

- `SUPABASE_URL` / `SUPABASE_ANON_KEY`
- `DUFFEL_API_KEY` — flights
- Affiliate keys (Viator / Stay22) as configured in that app

### 3. COMMS LINK (`C:\development\projects\wisense_decompression`)

- On-device zero-cloud. `flutter_secure_storage` & `local_auth`. No external API keys required.

---

## Pre-Flight Diagnostic

```powershell
# Ollama up?
Get-NetTCPConnection -LocalPort 11434 -ErrorAction SilentlyContinue | Select-Object LocalPort, State
# Or:
Invoke-WebRequest -Uri http://127.0.0.1:11434/api/tags -UseBasicParsing -TimeoutSec 2
```

Related: [[00_AI_AGENT_MANIFEST]], [[Working Stack — Claude CLI and Ollama]], [[WiSense Governance — Rules and Protocols]]
