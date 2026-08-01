---
title: Stripe
tags: [vendor, payments, billing, apex]
aliases: [Stripe API]
date: 2026-07-20
---

# Stripe

Payment processing vendor. Deferred for Apex pilot launch — not currently integrated.

## Planned Use

- **Apex Scheduler**: Subscription billing for brewpub accounts (owner/manager/staff tiers). Deferred until pilot proves product fit.
- **New Horizon**: Not used — Duffel Payments handles checkout. See [[Duffel]].

## Status

Deferred. No API keys configured. No SDK installed in any project. When ready:
1. Add `stripe_flutter` or server-side `stripe` SDK
2. Route through Supabase Edge Functions for PCI compliance
3. Implement subscription tiers in Apex `organizations` table

Related: [[Apex Scheduler]], [[Supabase]], [[Duffel]]