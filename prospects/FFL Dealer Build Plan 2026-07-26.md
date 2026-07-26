---
title: FFL Dealer Site — Build Plan and Reuse Strategy
tags: [prospects, ffl, build-plan, reuse, jigsy, prompt, pdf]
date: 2026-07-26
status: active
---

# FFL Dealer Site — Build Plan and Reuse Strategy

> Plan to build the Thompsontown FFL dealer website by reusing code from the Jigsy's site to save AI quota. Copy-paste approach, not generate-from-scratch.

Related: [[prospects/FFL Dealer Website Research 2026-07-26]], [[prospects/Thompsontown FFL Dealer — Warm Lead Strategy]], [[business/Reusable Platform Components 2026-07-26]], [[business/Pricing Models Ownership and Exit 2026-07-26]], [[NOW]]

---

## What to Reuse from the Jigsy's Repo

The Jigsy's site (github.com/nicholaswittle/jigsysite or jigsysiteworking) has code we can copy-paste directly. No need to generate from scratch.

### Pull directly (copy-paste, swap content):

| Jigsy's File | What to Reuse | What to Change |
|---|---|---|
| `index.html` | Page structure, layout, sections, responsive design | Swap restaurant content for FFL dealer content |
| `demo.css` | Theme, responsive rules, mobile-first CSS, section styling | Swap colors: warm/amber → tactical dark (charcoal, steel, brass) |
| Catering form (in index.html) | Form structure, validation, submit handler | Swap fields: catering → FFL transfer request |
| `images/` folder structure | Icon, favicon, hero image slots | Swap food photos for tactical/gun store images |
| `vercel.json` | Deploy config | Same — just deploy |
| `.gitignore` | Same | No change |
| `vite.config.ts` | Build config | No change |
| `package.json` | Dependencies (if any) | No change |
| `staff.webmanifest` | PWA manifest | Swap name/icons |

### Do NOT pull (not needed for this site):

| File | Why Skip |
|---|---|
| `order-demo.html` | No online ordering for FFL v1 |
| `order-demo.js` | No cart/menu logic needed |
| `staff-demo.html` | No staff console needed |
| `staff-demo.js` | No staff console needed |
| `api-client.js` | No API calls needed (form submits via mailto or simple endpoint) |
| `worker/api.ts` | No backend API needed for v1 |
| `worker/index.ts` | No Cloudflare Worker needed for v1 |
| `db/schema.ts` | No database needed for v1 |
| `config/menu-catalog.json` | No menu — swap for services list |
| Square integration files | No payment processing |

### The form approach for v1:

The Jigsy's catering form used JavaScript to handle submission. For the FFL transfer form, simplest approach:

Option A (zero backend): Form uses `mailto:` or a free form service (Formspree, Netlify Forms, Getform) — no backend, no database, form submissions go straight to the owner's email.

Option B (reuse Jigsy's backend): If we want the form to store submissions and show them in a staff console later, pull the Cloudflare Worker + D1 setup. But that's overkill for v1.

**Recommendation: Option A.** Use Formspree (free tier, 50 submissions/month) or Netlify Forms (free, 100 submissions/month). The form posts to their endpoint, they email the shop owner. Zero backend code. When the client is paying $79/mo and wants a staff dashboard, upgrade to the Cloudflare Worker + D1 setup from Jigsy's repo.

---

## Features for v1 (Minimum Viable Site)

### Must-have:
1. **Hero section** — Store name, FFL trust badge, phone number (click-to-call), tagline
2. **FFL Transfer Request Form** — The money-maker. Fields: name, phone, email, firearm make/model/caliber/type/serial, selling dealer name + contact, order number, notes, preferred pickup time
3. **Services section** — FFL transfers, firearms manufacturing/gunsmithing (Type 07), NFA/Class 3 services, ammo/accessories special orders
4. **Hours & Location** — Store hours, address, Google Maps embed, phone, directions link
5. **Contact** — Phone, email, Facebook (if they have one)
6. **Footer** — "Built by WiSense LLC" credit, copyright

### Nice-to-have (add if time allows):
7. **FAQ section** — Transfer process, what to bring, NICS info, wait times
8. **Inventory inquiry form** — "Looking for something specific?" (reuse same form structure)
9. **NFA/Class 3 info section** — Suppressor process, Form 4 help

### Skip for v1:
- E-commerce / online sales
- Live inventory
- Staff console
- Square integration
- Blog
- Newsletter
- Reviews (no reviews to show yet)

---

## Build Steps (Off-Weekend, 2-3 Hours Total)

### Step 1: Clone Jigsy's repo (5 min)
```
git clone https://github.com/nicholaswittle/jigsysite.git ffl-dealer-demo
cd ffl-dealer-demo
git checkout main
```

### Step 2: Strip what we don't need (10 min)
Delete: order-demo.html, order-demo.js, staff-demo.html, staff-demo.js, api-client.js, worker/, db/, config/, drizzle/, demo-data.js
Keep: index.html, demo.css, images/ (structure, swap content), vercel.json, vite.config.ts, package.json, .gitignore

### Step 3: Swap the content in index.html (60-90 min)
Copy the section structure from Jigsy's index.html:
- Hero section → FFL dealer hero (name, badge, phone, tagline)
- Menu section → Services section (transfers, manufacturing, NFA, special orders)
- Catering form → FFL Transfer Request Form (swap fields)
- Hours/address section → FFL dealer hours + address + Google Maps
- Footer → WiSense credit

Swap all text content. Keep the HTML structure, CSS classes, responsive layout.

### Step 4: Swap the CSS theme (20 min)
In demo.css, find/replace:
- Warm amber/gold colors → charcoal (#1a1a2e, #16213e), steel grey (#0f3460), brass (#c9a227)
- Pizza/restaurant fonts → tactical/clean fonts (same font family is fine, just change colors)
- Keep all responsive rules, mobile-first breakpoints, section spacing

### Step 5: Swap images (10 min)
Replace Jigsy's food photos with:
- Placeholder tactical images (free stock photos of firearms, gun store interior, tactical gear)
- Or ask the owner for real photos later
- Swap favicon to a tactical icon

### Step 6: Set up the form (15 min)
Create a free Formspree account (formspree.io) or use Netlify Forms:
- Create a form endpoint
- Point the FFL transfer form's action to the endpoint
- Set the owner's email as the recipient
- Test with a dummy submission

### Step 7: Deploy to Vercel (5 min)
```
git init
git add -A
git commit -m "FFL dealer demo site"
git remote add origin https://github.com/nicholaswittle/ffl-dealer-demo.git
git push -u origin main
```
Then in Vercel: import the repo, deploy. Live URL in 2 minutes.

### Step 8: Text the owner
"Hey, I put together that demo site for you. Take a look: [live URL]. Let me know what you think."

---

## Color Palette (Tactical Dark)

| Element | Color | Hex |
|---|---|---|
| Background | Charcoal | #1a1a2e |
| Secondary bg | Dark steel | #16213e |
| Accent | Brass/bronze | #c9a227 |
| Text | Light grey | #e0e0e0 |
| Headings | White | #ffffff |
| Links/buttons | Brass | #c9a227 |
| Form borders | Steel | #0f3460 |

---

## FFL Transfer Form Fields

```
Section 1: Customer Info
- Full Name (required)
- Phone Number (required)
- Email Address (required)

Section 2: Firearm Info
- Make/Manufacturer (required)
- Model (required)
- Caliber (required)
- Type (dropdown: Handgun, Rifle, Shotgun, Other)
- Serial Number (if known)

Section 3: Transfer Details
- Selling Dealer Name (required)
- Selling Dealer Phone or Email (required)
- Order/Reference Number
- Has the seller been contacted about shipping? (Yes/No/Not sure)

Section 4: Additional
- Preferred Pickup Time (text)
- Notes/Special Instructions (textarea)
- Submit button: "Submit Transfer Request"
```

On submit: email to shop owner with all fields. Auto-reply to customer: "We received your transfer request. We'll contact you when the firearm arrives at our shop."

---

## Selling Point PDF Prompt

Paste this into Claude or ChatGPT to generate the PDF:

```
Create a professional 2-page PDF sales sheet for a local FFL dealer (Type 07 Firearms Manufacturer, NFA/Class III eligible) in Thompsontown, PA who currently has NO website.

The business: A licensed FFL dealer and firearms manufacturer in Thompsontown, PA (17094). They handle FFL transfers, custom gunsmithing/manufacturing, NFA/Class 3 services (suppressors, SBRs), and ammo/accessories special orders. They are listed on GunMade.com but have no website. Their phone is 717-953-6359.

The offer: I will build them a custom, modern, mobile-friendly website with an FFL transfer request form, services section, hours/location, and contact info. $0 upfront, $79/month, I handle all hosting, maintenance, and updates. 3-month minimum, cancel anytime. If they ever want to own the code, $500 buyout.

The PDF should include:

PAGE 1:
- Title: "Your Shop Needs a Website — Here's Why"
- The problem: Right now, buyers searching for "FFL transfer near Thompsontown" or "gun store Thompsontown PA" cannot find this shop online. They find competitors instead. When someone buys a gun on GunBroker or Palmetto State Armory and needs a local FFL transfer, they Google for a dealer — and this shop is invisible.
- The solution: A professional, mobile-friendly website with an online FFL transfer request form. Customers find the shop, request transfers from their phone, and the shop owner gets an email with all the details. No more phone tag, no more missed transfers.
- The FFL Transfer Form: Explain how it works — customer fills out the form online (name, phone, firearm details, selling dealer info), shop owner gets an email instantly, customer comes in for pickup and pays the transfer fee ($25-35). One transfer per week pays the entire $79/month cost.
- Key features list: FFL transfer request form, services breakdown (transfers, manufacturing, NFA/Class 3, special orders), store hours and location with Google Maps, click-to-call phone number, mobile-friendly design, tactical dark professional aesthetic.
- The hosting advantage: "Shopify, Squarespace, and Wix restrict or ban firearms-related businesses. My hosting (Cloudflare) has no content restrictions on legal FFL dealers. Your site stays up."

PAGE 2:
- Pricing: $0 setup fee, $79/month, 3-month minimum, cancel anytime. No contracts. Buyout option: $500 one-time to own the code.
- ROI math: "One FFL transfer per week = $100-140/month in transfer fees. Your website costs $79/month. The site pays for itself with ONE transfer." Show a simple table: 1 transfer/week = $100-140/mo revenue, minus $79/mo website = $21-61 profit. 2 transfers/week = $200-280/mo revenue, minus $79 = $121-201 profit. 4 transfers/week = $400-560/mo revenue, minus $79 = $321-481 profit.
- What I handle: Website design, hosting, SSL security, domain setup, form setup, maintenance, content updates (hours, services, etc.). The owner does nothing technical.
- What the owner handles: Telling me their store name, hours, services, transfer fee, and any photos they want. That's it. 15 minutes of their time.
- About me: "Nicholas Wittle, WiSense LLC. Local to the area. I build websites for local businesses using modern AI tools. I also build mobile apps (I have apps on the App Store). I'm not a big agency — I'm a local guy who happens to be good at this."
- Call to action: "Want to see what your site could look like? I'll build you a free demo. Text me at [phone] or email nicholaswittle@wisensellc.com. No pressure, no obligation."
- Footer: WiSense LLC · nicholaswittle@wisensellc.com · wisensellc.com

Design: Clean, professional, tactical aesthetic. Dark charcoal background, white text, brass/bronze accents. Use a sans-serif font. Include a mockup screenshot of a tactical-themed FFL website on page 1 if possible. Make it look like something a professional agency would hand a client.

Output as a formatted document ready to save as PDF.
```

---

## Timeline

| Step | Time | When |
|---|---|---|
| Clone + strip Jigsy's repo | 15 min | Off-weekend morning |
| Swap content in index.html | 90 min | Same |
| Swap CSS colors | 20 min | Same |
| Swap images | 10 min | Same |
| Set up Formspree form | 15 min | Same |
| Deploy to Vercel | 5 min | Same |
| Generate PDF with Claude | 15 min | Same |
| Text owner the link + PDF | 2 min | Same evening |
| **Total** | **~3 hours** | **One off-weekend** |

After he says yes:
- Buy his domain (~$10/yr, he pays)
- Point domain to Vercel
- Go live
- First client acquired

---

## What Happens After the Demo

If he likes it:
1. Ask for his store name, hours, services list, transfer fee, and any photos
2. Swap the demo content for his real content (30 min)
3. He buys a domain on Namecheap (~$10/yr) or I help him buy it
4. Point the domain at Vercel (5 min — I do this)
5. Set up the form to email his real address
6. Go live
7. He pays $79/mo via PayPal, Venmo, or check
8. First paying client acquired

If he wants changes:
- Adjust colors, layout, content (30 min)
- Re-deploy
- Same process

If he says no:
- Still have an FFL dealer template to sell to the next gun store
- Zero waste