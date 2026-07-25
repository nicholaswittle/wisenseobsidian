---
title: Jigsy's Ordering Platform — Claude Handoff
tags: [jigsys, handoff, cloudflare, square, orders-api, online-ordering]
date: 2026-07-25
status: active-development
supersedes: Jigsys Ordering Platform — Claude Handoff 2026-07-24
---

# Jigsy's Ordering Platform — Claude Handoff (2026-07-25)

This supersedes the 2026-07-24 handoff. Major change: the app moved off the
private OpenAI Sites host onto **public Cloudflare Workers**, and the Square
integration changed from **online card checkout** to **pushing orders into the
restaurant's Square** with payment collected at the counter.

## Read this first

The pilot app is live, public (no login wall), and working end to end in Square
**Sandbox**. Customers place pickup orders and pay at the restaurant; staff accept
in one click, which prints a ticket, pushes the order into Jigsy's Square, and
marks it done. Nothing here has touched a real card or real money.

## Live URLs

- App (canonical): **https://jigsys-ordering-demo.wisense.workers.dev**
  - Customer: `/order-demo` · Staff: `/staff-demo`
- Vercel link (redirects to the app): **https://jigsys-ordering-demo.vercel.app**
  - Production deploy `dpl_3f12j3ZHCkVRX25CfG3FtFqbvdWV`, 307-redirects all paths.

## Why we moved off Sites

The OpenAI Sites deployment (`*.chatgpt.site`) forces **every visitor to sign in
with a ChatGPT account**, and its auth was erroring. That's unusable for customers,
staff, or a resale tenant. The app was already Cloudflare-native, so we redeployed
it to **public Cloudflare Workers + D1** — same code, public URL, no gate.

## Hosting & deploy (Cloudflare)

- Worker `jigsys-ordering-demo`; D1 database `jigsys-ordering`
  (id `84ecf5aa-ce54-497f-9e7b-4e13197fe04a`, binding `DB`), region ENAM.
- Cloudflare account subdomain: `wisense.workers.dev`.
- Node + wrangler are installed on Nick's Mac. Deploy from the repo:
  `CF_D1_DATABASE_ID=<id> CF_D1_DATABASE_NAME=jigsys-ordering pnpm build`
  then `pnpm exec wrangler deploy --config dist/server/wrangler.json`.
- Full runbook committed at `docs/CLOUDFLARE-DEPLOY.md` in the ordering repo.
- Secrets are Cloudflare Worker secrets (`STAFF_PIN`, `STAFF_SESSION_SECRET`,
  `SQUARE_APPLICATION_ID/SECRET`, `SQUARE_TOKEN_ENCRYPTION_KEY`, `SQUARE_ENV`,
  `SQUARE_REDIRECT_URI`) — never in the repo.

## How Square works now (the model)

- Business model: **order intake + kitchen ticket; customer pays at the counter**
  on Jigsy's own Square POS. The app does **not** process cards online (no PCI).
- On staff **Accept** (one click): order → **Completed**, prints a backup ticket,
  and is pushed to Square via `POST /v2/orders` (line items + $0.99 fee line +
  PICKUP fulfillment with the customer's name/phone/pickup time). Stored as
  `orders.square_order_id`. Verified in Sandbox (e.g. order `J047569`).
- **No-show handling:** staff can mark a Completed order **"Didn't pay"** at end of
  shift → it becomes **Unpaid**, dropping its $0.99 (and value) from the WiSense
  fee tracker and daily report. Reversible via "Mark as paid".
- The old card-checkout / capture / refund code still exists but is **unused** in
  this model. The app's backup ticket stays until Square auto-print is confirmed on
  Jigsy's real printer (owner reports Square auto-prints on Save + name).

## Repos & state

- Ordering app: `nicholaswittle/jigsysiteworking`, branch
  `agent/reusable-ordering-core`, HEAD `dbcf0c2` (pushed).
- This vault: `nicholaswittle/wisenseobsidian`, branch
  `agent/update-jigsys-owner-feedback`.
- Owner presentation/questionnaire files under `output/` are untracked user files —
  preserved, not committed here.

## Sandbox connection note

Square Sandbox OAuth kept failing in the console ("launch the seller test account"),
so the WiSense Test Restaurant authorization was **seeded directly into D1** for
validation. For Jigsy's real account, use the proper OAuth flow — an authorized
owner clicks Connect Square and approves on Square's page (production has no
test-seller gate). `SQUARE_TOKEN_ENCRYPTION_KEY` must match the key that encrypted
the stored tokens (a mismatch caused a "Decryption failed" bug we fixed).

## Security / cleanup before anything real

- Several **sandbox** secrets/tokens were pasted into the working chat (Square app
  secret, token encryption key, WiSense Test Restaurant access/refresh tokens).
  Fine for practice — **regenerate all before any non-sandbox use.**
- Never commit Square secrets, tokens, encryption keys, staff PINs, or session
  secrets.

## Remaining work (priority order)

1. Owner sign-off on real **menu, prices, tax treatment, and $0.99 fee wording**
   (checklist committed at `docs/OWNER-APPROVAL-CHECKLIST.md`).
2. On Jigsy's **real Square**: connect via OAuth (owner approves) and confirm their
   Square **auto-prints** the pushed orders on the kitchen printer; then the app's
   backup ticket can be removed.
3. Test the installable iPad **PWA** on the shop's device/Wi-Fi.
4. Regenerate the exposed sandbox secrets.
5. Confirm tax handling on the pushed Square order (currently food + fee, no tax
   line — relies on Square location settings at the counter).

## Related vault notes

- [[Jigsys Ordering Platform — Claude Handoff 2026-07-24]]
- [[Jigsys Square Sandbox Checkout — Completion Note 2026-07-24]]
- [[Jigsys Square Connection Requirements]]
- [[Jigsys Website & Direct Ordering Master Plan]]
