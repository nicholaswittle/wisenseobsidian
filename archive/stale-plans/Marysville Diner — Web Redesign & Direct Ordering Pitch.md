---
title: Marysville Diner — Web Redesign & Direct Ordering Pitch
tags: [marysville-diner, business, proposal, cold-pitch, web-redesign, direct-ordering]
date: 2026-07-26
status: demo-deployed
---

# 🍳 Marysville All-American Diner — Pitch Audit & Demo Strategy

> **Target:** Marysville All-American Diner  
> **Location:** 510 S State Rd, Marysville, PA 17053 · (717) 957-2105  
> **Current Site:** `marysvillediner.com` (GoDaddy Builder 8.0)  
> **Live Redesign Demo:** [https://marysvilledinersite.vercel.app](https://marysvilledinersite.vercel.app)

---

## 🚨 Audited Weaknesses on Their Current Site (`marysvillediner.com`)

1. **Broken Primary Button:** The hero section button `"Explore Menu"` has a broken anchor link (`#ccebf178-b739-43b6-8041-03886eed22ff`) that does literally **nothing** when clicked by visitors on mobile or desktop.
2. **Outdated GoDaddy Builder:** Plain, low-contrast template built on GoDaddy Website Builder 8.0 with static PDF menu download links instead of an interactive, searchable menu.
3. **No Direct Online Ordering:** Takeout is 100% phone-only, creating busy signal bottlenecks during peak breakfast (7–9 AM) and dinner hours.
4. **Weak Aesthetics:** Lacks high-converting diner imagery, warm typography, or dark mode support.

---

## 🚀 The WiSense Redesign Demo Features ([marysvilledinersite.vercel.app](https://marysvilledinersite.vercel.app))

1. **Working "Explore Menu" Button:** Instantly scrolls smoothly to the interactive menu matrix.
2. **Interactive Homestyle Menu Board:** Tabbed categories (`Breakfast All Day`, `Sourdough Melts`, `Comfort Dinners`, `Pies & Desserts`) with live `+ Add to Cart` buttons.
3. **Slide-Over Web Cart Modal (`#cart-drawer`):** Allows customers to place direct takeout orders online.
4. **Real-Time Hours Status Badge:** Dynamically calculates open status (`🟢 Open Now til 9 PM` vs `🔴 Closed`).
5. **Staff Counter Rush Controls (`/staff.html`):** Turnkey tablet portal featuring a 1-tap **"Rush Mode" (+15m delay)** button and **"Pause Web Orders"** toggle.

---

## 🗣️ The Pitch Script to Send/Present to the Owner

> *"Hi [Owner Name],*
>
> *I’m Nicholas with WiSense LLC here in Perry/Cumberland County. I’m a huge fan of Marysville Diner’s homemade cooking, but I noticed a couple of technical glitches on your current site — specifically, your top 'Explore Menu' button is currently broken and doesn't open the menu when clicked.*
>
> *I went ahead and built a live, working modern prototype for Marysville Diner so you can see what an upgraded experience would look like:*
> 
> 🔗 **Live Demo:** https://marysvilledinersite.vercel.app
>
> *Here is what we added:*
> 1. **Fixed the Menu Button** so mobile visitors land directly on your food items.
> 2. **Interactive Menu Board** for Breakfast All Day, Sourdough Melts, and Mile-High Meatloaf.
> 3. **Optional Direct Web Ordering** so customers can order takeout online instead of clogging your phone lines during morning & weekend rushes.
> 4. **1-Tap 'Rush Mode' Counter Tablet** so your staff can pause web orders with a single tap if the dining room gets slammed.
>
> *We offer a $0 upfront setup model. You keep 100% ownership of your domain (marysvillediner.com) in your own GoDaddy account — we just update two quick DNS records so your domain loads our fast new site instead of the old broken builder. Would you be open to taking a look at the live demo?"*

---

## 🌐 Existing GoDaddy Domain Mapping & Cutover (3-Minute Setup)

> **Key Client Benefit:** The owner does **NOT** need to transfer `marysvillediner.com` or buy a new domain. They retain 100% ownership in their own GoDaddy account, avoiding monthly GoDaddy builder fees while keeping their existing web address.

### The 3-Minute DNS Cutover Steps:
1. Log into the client's GoDaddy account (or guide them over screen share).
2. Go to **DNS Management** for `marysvillediner.com`.
3. Update two standard records:
   - **A Record (`@`):** Point to `76.76.21.21` *(Vercel Production Server IP)*
   - **CNAME Record (`www`):** Point to `cname.vercel-dns.com`
4. **Result:** `marysvillediner.com` automatically loads the new WiSense site with free SSL (HTTPS) enabled in under 5 minutes.

---

Related: [[business/WiSense Operational Partner Plan — Jigsy's]], [[business/Jigsys Website & Direct Ordering Master Plan]], [[NOW]]
