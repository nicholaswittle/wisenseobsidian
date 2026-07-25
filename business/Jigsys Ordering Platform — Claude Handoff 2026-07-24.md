---
title: Jigsy's Ordering Platform — Claude Handoff
tags: [jigsys, handoff, square, sandbox, vercel, github, online-ordering]
date: 2026-07-24
status: active-development
---

# Jigsy's Ordering Platform — Claude Handoff

## Read this first

The reusable Jigsy's ordering demo is working end to end on the private Sites
deployment with Square Sandbox. The Vercel production alias now redirects its
homepage, customer page, and staff page to that verified transactional build.
GitHub App connection is not required for the current pilot deployment; Vercel
was deployed through the already-authenticated local CLI.

## Repositories and exact state

### Ordering demo

- Local: `/Users/nickwittle/Documents/Codex/2026-07-12/what/jigsysite-ordering-demo`
- GitHub: https://github.com/nicholaswittle/jigsysiteworking
- Branch: `agent/reusable-ordering-core`
- Current commit: `3ef78e2` (transactional build plus Vercel redirects)
- Transactional build commit: `616a3faacdda187f5e4128ac34058ab8620b3132`
- Working tree was clean after the final code push.
- The original `nicholaswittle/jigsysite` concept repository remains unchanged.

### WiSense vault

- Local: `/Users/nickwittle/Documents/Codex/2026-07-12/what/wisenseobsidian`
- GitHub: https://github.com/nicholaswittle/wisenseobsidian
- Branch: `agent/update-jigsys-owner-feedback`
- Previous documentation commit: `2ab784e`
- Several owner presentation/questionnaire files under `output/` are untracked
  user files. Preserve them and do not delete, overwrite, or include them in an
  unrelated commit.

## Deployments

### Verified transactional deployment

- URL: https://jigsys-ordering-demo.nicholaswittle.chatgpt.site
- Customer: `/order-demo`
- Staff: `/staff-demo`
- Sites project: `appgprj_6a62c69129608191830ef5698f7a16f9`
- Saved/deployed version: 18
- Source commit: `616a3faacdda187f5e4128ac34058ab8620b3132`
- Environment revision: 2
- Access: private/owner-only practice site

### Vercel redirect deployment

- Production alias: https://jigsys-ordering-demo.vercel.app
- Deployment ID: `dpl_46bq4QDrMdSQJKY4eNHW9YKLbG34`
- Immutable URL:
  https://jigsys-ordering-demo-ef77f7sa9-wi-sense-llc.vercel.app
- Vercel project: `prj_QSnq0n2m1ycQDWI4lplT51FJ5bma`
- Team: `team_AetWe6FmD1tHGwvqCOqTgqAE`
- Status: READY
- `/`, `/order-demo`, and `/staff-demo` were each verified to return a `307`
  redirect to the matching Sites page.
- Vercel remains a lightweight entry URL; the API, D1 database, staff sessions,
  OAuth, and Square payments continue to run on Sites.

## What is implemented

- Shared hosted orders and restaurant settings
- Protected staff login and time-limited session
- Full menu and staff item-availability controls
- Pause/reopen online ordering and shared pickup estimate
- Customer private order-status token
- Waiting, accepted, rejected, completed, and cancelled states
- Receipt-width kitchen ticket and daily reports
- $0.99 fee included in order total
- Fee reporting counts only completed paid orders
- Square Sandbox OAuth connection to **WiSense Test Restaurant**
- Encrypted server-side Square access and refresh tokens
- Automatic Square OAuth token refresh
- Separate staff switch for Sandbox card checkout
- Square Web Payments card entry on customer checkout
- Server-side total verification
- Delayed capture:
  - customer submit = authorize
  - staff accept = capture
  - staff reject = void
- Payment-pending rows hidden from the kitchen queue
- Captured Square orders blocked from ordinary cancellation until a refund flow
  exists
- Production Square mode deliberately disabled

Important database addition:

- `orders.square_payment_id`
- Migration: `drizzle/0002_talented_mordo.sql`

Important payment routes:

- `GET /api/public/square-config`
- `GET /api/staff/square/status`
- `POST /api/staff/square/payment-mode`
- `POST /api/orders`
- `PATCH /api/staff/orders/:id`

## Live verification already completed

| Test | Order | Result |
|---|---|---|
| Accept/capture | `J272265` | $8.93 authorized, captured on acceptance, marked completed, fee counted |
| Reject/void | `J758481` | $8.93 authorized, voided on rejection, no fee counted |

Test card:

- Visa `4111 1111 1111 1111`
- Expiration `12/30`
- CVV `111`
- ZIP `17025`

These were Square Sandbox transactions only. No real money moved.

Final checks:

- `pnpm run stage` passed
- `pnpm lint` passed
- `pnpm build` passed
- Sites worker error scan was clean after final verification
- Vercel deployment completed successfully

## Bugs found and fixed during verification

1. Square's card stylesheet was blocked by the page CSP. The Sandbox Square CDN
   was added to `style-src`.
2. The new order insert had 18 placeholders for 17 columns. It was corrected
   before the successful authorization tests.

## Current switches and test data

- Square Sandbox checkout was left **enabled** on the private Sites deployment.
- The two fake test orders remain in the daily staff report as evidence.
- The Vercel alias redirects visitors to the private transactional build.

## Security rules

- Never commit Square secrets, OAuth tokens, encryption keys, staff PINs, or
  session secrets.
- Protected values currently live in the Sites runtime environment.
- The Square Sandbox application secret was pasted into a prior conversation.
  Rotate that Sandbox secret before relying on this beyond practice, then
  update only the protected runtime environment.
- Never request Jigsy's Square password, bank information, deposits, payouts,
  payroll access, or customer card numbers.
- In a real connection, an authorized owner approves OAuth on Square's page.

## Hosting decision completed

Sites is the canonical transactional pilot host. Vercel is a friendly entry
URL that redirects to Sites. Do not port the backend to Vercel unless Nicholas
later asks for a deliberate hosting migration.

## Remaining work in priority order

1. Add installable iPad Home Screen PWA support and real push notifications.
2. Test printing with Jigsy's actual Star Micronics TSP100 and Square printer
   profile; do not promise silent printing before this test.
3. Get the remaining owner's approval and complete the owner-information
   checklist.
4. Confirm menu, modifiers, prices, tax treatment, and customer-facing wording
   for the $0.99 fee.
5. Build and test refunds before any real-card pilot.
6. Add customer accepted/rejected notifications, outage handling, privacy
   language, support procedure, and audit reporting.
7. Only after written approval: create/approve production Square credentials,
   have an authorized Jigsy's owner connect the correct location, and run a
   tightly controlled live pilot.

## Related vault notes

- [[Jigsys Square Sandbox Checkout — Completion Note 2026-07-24]]
- [[Jigsys Square Connection Requirements]]
- [[Jigsys Ordering Demo — Build Record 2026-07-23]]
- [[Jigsys Website & Direct Ordering Master Plan]]
