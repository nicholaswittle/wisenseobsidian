---
type: strategy
title: "WiSense Restaurant OS — Flagship Website Agency Upsell Strategy"
tags: [strategy, pricing, business-model, agency-upsell, apex-v2, high-ticket]
date: 2026-07-28
status: active
---

# 💰 WiSense Restaurant OS — Flagship Website Agency Upsell Strategy

> **Core Concept:** Offering a high-ticket **"Flagship Custom Site + OS Integration Package"** ($1,499 upfront + $99/mo OS) alongside the $0 setup self-serve OS.

---

## 1. Why This Strategy Works (The Cash Flow Engine)

For a solo founder, relying exclusively on $99/mo self-serve SaaS means waiting 12 months to collect $1,188 from a single customer. 

Adding an optional **Custom Website Build Upsell** creates immediate upfront cash flow:

```
[ Self-Serve Customer ]  ──>  $0 Setup + $99/mo OS           = $1,188 ARR (Hands-Off)
[ Flagship Customer ]    ──>  $1,499 Setup + $99/mo OS        = $2,687 Year 1 (High Cash Flow)
```

**Financial Impact:**  
Closing just **2 custom website setups per month** generates **$3,000 upfront cash flow**, funding SaaS infrastructure while building high-proof regional case studies (like Jigsy's).

---

## 2. Two-Tier Product Menu

| Package | Upfront Fee | Monthly Fee | What the Customer Gets | Founder Ops Load |
|:---|:---:|:---:|:---|:---:|
| **Self-Serve OS** | **$0** | **$99 / mo** | Venue uses their existing site / Instagram / Google Business profile and points an "Order Online" button to their Apex hosted guest link (`/?token=XYZ`). | **0 Hours (Hands-Off)** |
| **Flagship Custom Package** | **$1,499** | **$99 / mo** | WiSense builds a custom, high-converting, mobile-first website (like Jigsy's) with photography, SEO Schema.org, local story band, and embedded ordering. | **4 Hours (Template Sprint)** |

---

## 3. The 90-Minute Template Deployment Playbook

Because `jigsys_site` was built as a clean, tokenized single-page HTML/CSS architecture, deploying a custom website for a new restaurant client is literally a **90-minute copy-paste workflow**:

1. **Copy Template Repository:** `cp -r jigsys_site new_venue_site`
2. **Swap Design Tokens (5 mins):** Update `:root` CSS variables in `index.html`:
   - `--sauce`: Primary accent / brand color
   - `--crust`: Secondary accent
   - `--ground`: Background tone (dark/light)
3. **Insert Content & Photos (45 mins):** Replace logo, hero banner, venue story, hours, and phone number.
4. **Wire Supabase `public_token` (10 mins):** Set `public_token = 'new-venue-token'` in the embedded guest order button script.
5. **1-Command Deploy (5 mins):**  
   `vercel --cwd . deploy --prod --yes --scope wi-sense-llc`

### Financial Return on Time:
* **Fulfillment Time:** 90 Minutes (1.5 Hours)
* **Client Fee:** $1,499 Upfront + $99/mo OS
* **Effective Hourly Rate:** **~$1,000 / Hour**

---

## 4. Operational Rules: How to Avoid "Agency Trap"

To prevent custom website builds from consuming founder time:

1. **Standardize on the Jigsy Single-Page Template Core:**  
   Do not build custom WordPress or complex CMS backends. Use the single-page, ultra-fast HTML/CSS template (`jigsys_site` architecture) hosted on Vercel under `wi-sense-llc`.
2. **4-Hour Fulfillment Cap per Site:**  
   - Hour 1: Receive brand assets, menu, hours, and photos.
   - Hour 2: Transcribe menu tokens & color variables into template.
   - Hour 3: Test responsive preview & wire Supabase `public_token`.
   - Hour 4: Deploy to Vercel & point venue domain.
3. **Capacity Hard Cap:**  
   Cap custom website builds at **maximum 3 per month**. If demand exceeds 3, raise the setup price to $2,499.

---

## 📄 File Location & Sync
Saved to Obsidian Vault static reference:  
`C:\Users\nikwi\Notes\wisense\projects\APEX_V2_AGENCY_UPSELL_STRATEGY_2026-07-28.md`  
Committed and pushed to GitHub `https://github.com/nicholaswittle/wisenseobsidian.git`.
