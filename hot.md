---
type: meta
title: "Hot Cache"
tags: [meta, hot-cache, context]
updated: 2026-07-29T18:00:00
---

# Recent Context

> ~500-word cache for fast boot. Any agent/session reads this FIRST, then [[NOW]], then [[index]]. Overwrite completely each update — cache, not journal.

## Last Updated
2026-07-29 evening. Apex live: [https://apex-v2-ten.vercel.app](https://apex-v2-ten.vercel.app) · demo `apex-v2-demo.vercel.app` · HEAD `542fbb1`.

**Today:** membership moved to `organization_members` (an account belongs to the person, not the venue — staff can hold two jobs, leaving a venue unlinks rather than deletes); multi-venue picker; pay made genuinely manager-only via column grants; capacity now suggests instead of auto-pausing; **two Antigravity audits, all 13 findings closed**. Full write-up: [[Apex v2 — Restaurant OS Build]].

**Both audits' most severe item was one the auditor missed or mis-rated** — found by checking claims against the live DB, not by reading the report. Treat audit reports as leads, not conclusions.

🔴 **2026-07-29 — the three worst holes in Apex were in none of the four audits, because they are in no migration file.** `apex_grant_membership` was `SECURITY DEFINER`, checked nothing, and was granted to `anon`: one call with the publishable key made you Owner of any venue. `org invite consume` let one statement rewrite every pending invite in the database. `apex_set_subscription_status` handed out paid tiers. All closed — see [[wisense/projects/APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]]. **Audit the catalog, not the repo.** Until the migration history is reconciled, the repo cannot answer what is running in production.

**BEFORE THE PILOT:** test camera capture on a phone (desktop upload is proven, camera never was — it would have crashed on iOS until `542fbb1`); clear the test venues/accounts; set job roles.

✅ **Email confirmation is ON** (2026-07-29). ⚠️ **Configure custom SMTP before onboarding a venue.** Supabase's built-in mailer allows only a handful of messages per hour — it is a testing service. Invite six staff in one evening and most will never receive the confirmation, and it will look like the app is broken rather than like a mail limit. Auth → Emails → SMTP Settings; any of Resend/SendGrid/Postmark on a free tier covers this volume.

**Stripe is tested, contrary to the 2026-07-29 audits.** All three say "live Stripe charges not triggered" — meaning *those audits* did not trigger them. The payment path was audited and live-tested in test mode on 2026-07-28 at `eb45717`: [[wisense/projects/APEX_V2_STRIPE_CONNECT_AUDIT_2026-07-28]]. Open item is only a re-run on current HEAD, since `stripe-connect-onboard` changed on 2026-07-29.

⚠️ **Delete the five `@roster.local` placeholder profiles before launch** (Kim, Marsha, Dana, Courtney, Morgan). They are name tags with fake emails and no login, added so the photo reader would recognise those names during testing. Decision 2026-07-29: keep for testing, delete before going live — **so no automatic merge was built**. If any of them signs up while a placeholder still exists, the roster gets two entries with the same name, which is exactly the ambiguity that produced "lim" for Kim. Pay is unaffected (shifts store a name, not an account).

⚠️ `apex/apex/supabase/config.toml` still points at **Horizon's** project — a `supabase db reset --linked` from that folder hits the wrong database.
⚠️ After any deploy, **hard-refresh** — the Flutter service worker serves the old bundle and it looks exactly like a bug.

## Key Recent Facts
- **Vault Boot Chain:** [[hot]] → [[NOW]] → [[index]].
- **Session Handover:** [[wisense/projects/APEX_V2_COMPLETE_SESSION_HANDOVER_2026-07-28]] (Full summary of all 2026-07-28 audits, fixes, and deploys).
- **Flagship Strategy:** [[wisense/projects/WISENSE_LLC_APEX_FLAGSHIP_PIVOT_PLAN_2026-07-28]] (Full strategic pivot plan for wisensellc.com to feature Apex Restaurant OS as flagship).
- **Recent Shipped Audits & Fixes:**
  - [[wisense/projects/APEX_V2_STRIPE_PAYMENT_CONFIRMATION_MODAL_2026-07-28]] (Guest Stripe Payment Confirmed modal & URL fragment router fix - `fa16abd`).
  - [[wisense/projects/APEX_V2_INTERACTIVE_STRIPE_DISCONNECT_BUTTON_2026-07-28]] (Interactive Stripe connected button + disconnect engine - `6f4c00a`).
  - [[wisense/projects/APEX_V2_STRIPE_DISCONNECT_AND_PAY_NOW_FIX_2026-07-28]] (Pay Now badge & backend disconnect handler - `415436a`).
  - [[wisense/projects/APEX_V2_STAFF_CONSOLE_PAID_ONLINE_FIX_2026-07-28]] (Staff Console PAID ONLINE modal & DO NOT COLLECT thermal ticket fix - `6f80167`).
  - [[wisense/projects/APEX_V2_KITCHEN_ALERTS_PRINT_AUDIT_2026-07-28]] (Kitchen alerts, Twilio SMS & thermal print audit - `6d8b0ef`).
  - [[wisense/projects/APEX_V2_SMALL_MODEL_AI_AUDIT_2026-07-28]] (Small-model AI opportunity audit).
  - [[wisense/projects/APEX_V2_FULL_SYSTEM_SECURITY_AUDIT_2026-07-28]] (Full system security audit).

## Active Project Status
- **Apex v2:** Live at `https://apex-v2-ten.vercel.app` · Supabase `pqkremkwfkudrhtxasdj`.
- **Primary Sales Path:** Apex Restaurant OS ($99/mo) + Flagship Managed Web ($299 + $99/mo).
- **COMMS LINK:** ⏸️ PARKED.

## Active Threads
- Next: Execute wisensellc.com homepage redesign based on [[wisense/projects/WISENSE_LLC_APEX_FLAGSHIP_PIVOT_PLAN_2026-07-28]].
- [[NOW]] · [[index]] · [[Apex v2 — Restaurant OS Build]]
