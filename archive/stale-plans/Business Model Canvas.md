---
title: Business Model Canvas
tags: [business, model, canvas, strategy]
date: 2026-07-20
---

# Business Model Canvas — WiSense LLC

> Living document. Update as the model evolves.

## Value Proposition

**For** small teams and individuals who need specialized mobile tools
**who** are underserved by generic SaaS,
**our** Flutter apps deliver focused, on-device, privacy-first experiences
**that** eliminate cloud dependency, reduce cost, and work offline.
**Unlike** competitors who require accounts, subscriptions, and always-on internet,
**we** ship apps that respect privacy and work anywhere.

## Customer Segments

| Segment | App | Status | Pain Point |
|---|---|---|---|
| Veterans/first responders needing decompression | COMMS LINK | Near launch | Mental health apps are cloud-based, require accounts, retain data |
| Small businesses (brewpubs) needing staff scheduling | Apex Scheduler | Security fixes needed | Existing schedulers are enterprise-priced, complex, no mobile-first |
| Couples planning travel together | New Horizon | In development | Travel apps are solo-first, no consensus/alignment feature |
| Small teams needing shared UI/core packages | (Internal) | Active | Code reuse across apps saves AI quota and dev time |

## Revenue Streams

- **COMMS LINK**: One-time purchase ($4.99–$9.99) — no subscription, privacy is the selling point
- **Apex Scheduler**: Tiered subscription ($9/mo solo, $29/mo team, $79/mo multi-org) — deferred Stripe until pilot proves fit
- **New Horizon**: Affiliate commission (Viator 3–8%, Expedia 2–5%) + potential Duffel booking markup
- **Shared packages**: No direct revenue — internal cost saving (AI quota, dev time)

## Key Resources

- Flutter/Dart codebase (3 apps + 2 shared packages)
- Multi-AI agent team (Claude, Gemini, Codex, Cursor, Groq, Hermes)
- Supabase backend (auth, database, edge functions)
- Duffel API partnership (flight booking)
- Nicholas's domain expertise (Marine veteran, PA State Trooper, father — real user perspective)

## Key Activities

- Ship apps to app stores (Android-first, no Mac)
- Maintain shared package quality (MCA/MDT audits)
- Security hardening (RLS, session binding, Human Veto gates)
- Launch marketing (store optimization, privacy-first messaging)

## Channels

- Google Play Store (primary)
- Apple App Store (via cloud CI when Mac available)
- Direct web (Vercel for New Horizon)
- Content marketing (privacy-first, veteran-built narrative)

## Cost Structure

- Supabase free tier -> paid tier at scale
- Duffel API (pay per booking)
- AI agent quota (Claude, Gemini, OpenRouter)
- Apple Developer $99/yr (deferred)
- Google Play $25 one-time
- Vercel Hobby -> Pro at scale

## Open Questions

- [ ] Should COMMS LINK be freemium (free basic, paid pro) or one-time purchase?
- [ ] Apex: per-seat or flat org pricing for brewpubs?
- [ ] Apex: move messaging toward outcome pricing (conflict-free published weeks) — see [[business/Info-Tech Tech Trends 2026 — WiSense Playbook]] #8?
- [ ] New Horizon: pure affiliate or add subscription for premium features?
- [ ] Cross-app bundle discount strategy?
- [ ] Document vendor escape hatches (Duffel / Supabase) for supply-chain resilience (#1)?

Related: [[business/Startup Playbook]], [[business/Pricing Strategy]], [[business/Info-Tech Tech Trends 2026 — WiSense Playbook]], [[DECISIONS]]