---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-30T20:30:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-30 evening. Apex live: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app) · HEAD `07fa32a`.

🟢 **JIGSY IS TAKING BOTH THE APP AND THE WEBSITE** (2026-07-30). This settles the open question in [[wisense/projects/SITE_TEMPLATE_ONLINE_ORDERING_PLAN]] — the site is no longer speculative. Order of work still stands: finish Apex, then build ordering into the **site template** rather than as a one-off, so every flagship site ships taking money on day one.

💵 **Fee model settled: direct charges + guest-paid service fee.** On a $32.96 order the guest pays $35.46, the venue nets $33.61, WiSense earns 52¢ — versus **losing 67¢** before. Pitch line: an online order costs the venue card processing and nothing else, same as a card tapped at their counter. Two rules keep it a service fee not a card surcharge: never call it a card/processing fee, never vary it by payment method. See [[wisense/projects/APEX_PLATFORM_FEE_ECONOMICS_2026-07-30]].

📸 **Photo import shipped and measured** — `claude-opus-5` at low effort, **~10¢ per import, one-time per venue**. Everything else in Apex runs on Haiku (~$0.001/call). Full record: [[wisense/projects/APEX_PHOTO_IMPORT_BAKEOFF_2026-07-30]]. Three things from it worth carrying:

- 🔴 **`ListModels` advertises Gemini models that `generateContent` 404s.** Every Gemini ID in the deployed function was retired; that branch had never executed in production, it 404'd and fell through to Claude silently. Probe with a real one-token call before trusting any model ID.
- 🔴 **Ranking by "shifts found" picked the worse model** — it had silently shifted a row by one day. And two clean runs got reported as "perfect" when the true rate was ~1 wrong entry per import. Score against a confirmed answer, and run it more than twice.
- ⚠️ **A failed AI call bills like a successful one.** The client giving up does not stop the model. Three retries cost 3×.

🚦 **[[wisense/projects/APEX_GO_LIVE_SEQUENCE_2026-07-30]] — read before activating a live Stripe account.** No live account yet, so **no venue has onboarded for real and architecture decisions are still free**. That window closes when Jigsy connects. Nothing carries over from test mode: keys, webhook endpoints and signing secrets, Connect accounts, and payment links are all separate live objects.

**BEFORE THE PILOT:** custom SMTP (Supabase's built-in mailer is a testing service — invite six staff in one evening and most never get the mail, and it looks like the app is broken); clear test venues/accounts; delete the five `@roster.local` placeholders (Kim, Marsha, Dana, Courtney, Morgan); set job roles; deactivate 3 orphan Stripe payment links.

⚠️ **Tier enforcement is UI-only** — the remaining commercial gap.
⚠️ **Migration history is unreconciled** — the repo cannot answer what is running in production. Audit the catalog, not the repo ([[wisense/projects/APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]]).
⚠️ `apex/apex/supabase/config.toml` still points at **Horizon's** project — a `supabase db reset --linked` from that folder hits the wrong database.
⚠️ After any deploy, **hard-refresh** — the Flutter service worker serves the old bundle and it looks exactly like a bug.

## Key Recent Facts
- **Vault Boot Chain:** [[hot]] → [[NOW]] → [[index]].
- **Primary Sales Path:** Apex Restaurant OS ($99/mo) + Flagship Managed Web ($299 + $99/mo).
- **Session record:** [[wisense/projects/APEX_SESSION_2026-07-29_30_FULL_RECORD]]
- **Flagship strategy:** [[wisense/projects/WISENSE_LLC_APEX_FLAGSHIP_PIVOT_PLAN_2026-07-28]]
- **Stripe verified live in sandbox 2026-07-29** (`acct_1TqfSbHeXj7HLVbu`): guest Pay Now works end to end; the SaaS unlock branch of the webhook has **never fired** and must be treated as untested when unlocks are switched back on.
- **COMMS LINK:** ⏸️ PARKED.

## Active Threads
- Next: go-live sequence, then the site template for Jigsy.
- [[NOW]] · [[index]] · [[Apex v2 — Restaurant OS Build]]
