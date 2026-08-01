---
title: Easy AI Money Plays 2026-07-21
tags: [business, revenue, ideas, ai, services, fast-cash, strategy]
aliases: [Easy AI Money, Fast Cash Plays, Quick Revenue]
date: 2026-07-21
status: active
---

# Easy AI Money Plays — Fast Cash with AI

> Built 2026-07-21. These are NOT long-build SaaS products. These are fast-cash plays using AI skills and existing stack. Jigsy's Brewpub is a TEST BED only (wife works there) — NOT a paying client. Every plan below assumes cold outreach to real paying customers.

Related: [[business/Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]], [[business/2026 Commercial B2B Software Ideas]], [[business/Ideas Log]], [[Apex Scheduler]], [[NOW]]

---

## TIER 1 — CASH THIS WEEK (no product, just AI skills)

### Play 1: AI Content Repurposing Service

**What:** Take a business's existing content (YouTube video, blog post, podcast) and turn it into 5 social posts, an email newsletter, and a short-form video script. All AI-generated, lightly edited by you.

**Who buys:** Any local business creating content but not repurposing it — realtors, restaurants, gyms, coaches, consultants.

**Pricing:** $200-500 per content batch. Or $500-1,500/mo retainer for weekly repurposing.

**Tools you already have:** Claude (text), Gemini (multimodal), your agent orchestration stack. Zero new build.

**Step-by-step plan:**

1. Create a portfolio — take 3 public YouTube videos from local businesses, repurpose each into social posts + newsletter + script. This is your free sample.
2. Build a 1-page landing page (Carrd or Notion site, free) showing before/after examples.
3. Find 20 local businesses on YouTube/Instagram with decent content but weak social presence.
4. DM or email each: "I took your video about X and turned it into 5 posts + a newsletter. Here's the doc. If you want this weekly, it's $500/mo."
5. First 3 paying clients at $500/mo = $1,500/mo recurring for ~3 hours of work per week.

**Time to first $:** 3-5 days
**Effort:** Low (prompt engineering + editing)
**Risk:** Low
**Scaling:** Hire a VA to do the editing pass once you have 10+ clients. You just manage prompts and quality.

---

### Play 2: Google Business Profile Optimization

**What:** Most local businesses have a half-filled Google Business Profile — no posts, bad photos, no Q&A, no review responses. You use AI to write optimized descriptions, weekly update posts, and respond to every review. Charge a monthly retainer.

**Who buys:** Restaurants, dentists, auto shops, plumbers, electricians, salons, barbers — any business with a Google listing that gets foot traffic.

**Pricing:** $150-300/mo per business. 10 businesses = $1,500-3,000/mo.

**Tools you already have:** Claude for writing, your phone for photos/screenshots, Google Business API is free.

**Step-by-step plan:**

1. Pick a 5-mile radius around your home. Open Google Maps, search "restaurant", "dentist", "plumber", "auto repair", "salon".
2. For each result, check: do they have recent posts? Do they respond to reviews? Are their hours/photos accurate? Score them 1-5.
3. Walk into the 1-star and 2-star businesses (worst profiles). Say: "Your Google profile is costing you customers. I can fix it and manage it for $200/mo. First month free."
4. For the first month free: use AI to write 4 weekly posts, rewrite their description, respond to all existing reviews, upload better photos (they provide or you take).
5. After the free month, if they don't pay, stop. Most will because they see the engagement.
6. Each client takes 30 min/week. 10 clients = 5 hours/week for $2,000/mo.

**Time to first $:** 1 week
**Effort:** Low
**Risk:** Low
**Scaling:** 20-30 clients is manageable solo. Beyond that, hire a VA.

---

### Play 3: AI Resume + Cover Letter Service (Veteran-Focused)

**What:** AI-powered resume and cover letter writing service. Target transitioning military and veterans specifically — you know that audience, they trust a Marine.

**Who buys:** Transitioning service members, veterans changing careers, military spouses, first responders changing jobs.

**Pricing:** $75-150 per resume + cover letter package. Volume play — 5-10 per day = $375-1,500/day.

**Tools you already have:** Claude for writing, your military network, veteran Facebook groups, LinkedIn.

**Step-by-step plan:**

1. Create 3 sample resumes: one junior enlisted transitioning, one NCO transitioning, one officer transitioning. Show the "before" (military jargon) and "after" (civilian-readable, ATS-optimized).
2. Post in 5 veteran Facebook groups: "I'm a Marine turned AI tech. I use AI to translate military experience into resumes that civilian hiring managers actually understand. $75 for resume + cover letter. DM me."
3. Cross-post to r/veterans, r/MilitaryTransition, LinkedIn veteran groups.
4. Set up a simple intake form (Google Forms or Typeform free tier): current resume/DD214 summary, target job title, target industry, 3 job postings they want to apply to.
5. Turnaround in 24 hours: feed their info + target job posting into Claude, generate tailored resume + cover letter, review and edit, deliver PDF.
6. Offer a $50 upsell: 5 LinkedIn message templates for networking into their target companies.

**Time to first $:** 2-3 days
**Effort:** Low
**Risk:** Low
**Scaling:** At 10+ per day, build a simple web app with self-service intake + Stripe checkout. Still AI-generated, you just review.

---

## TIER 2 — CASH IN 2-4 WEEKS (tiny product, reuse your stack)

### Play 4: ReviewGuard AI — Automated Review Response

**What:** An automated agent that monitors Google and Yelp reviews for local businesses, drafts empathetic brand-compliant responses, and lets the owner approve and post in 1 tap from their phone.

**Who buys:** Multi-location restaurant groups, dental chains, auto repair shops, home services, any local business with 20+ reviews.

**Pricing:** $149/location/mo. 5 locations = $745/mo. 20 locations = $2,980/mo.

**Tools you need:** Flutter mobile (you have this), Supabase (you have this), Google Business Profile API (free), Yelp API (free for basic access), Claude/Gemini API for response generation.

**Step-by-step plan:**

Week 1 — Build the MVP:
1. Flutter app: login, see list of reviews (pulled from Google Business API), tap to see AI-drafted response, tap to approve/edit/post.
2. Supabase backend: store business profiles, review cache, response drafts, posting log.
3. Python worker (or Supabase Edge Function): poll Google/Yelp API every 2 hours for new reviews, send each to Claude API with brand voice instructions, store draft.
4. Push notification when new review + draft is ready (FCM — you have this).
5. Brand voice setup: on signup, ask the owner for 3 sample responses they've written. Use those as the few-shot prompt for their AI responses.

Week 2 — Get your first 3 paying customers:
6. Walk into 5 local businesses with 30+ reviews and poor response rates. Show them their own reviews with AI-drafted responses already done.
7. Free 2-week trial. If they use it, $149/mo.
8. Cold email 50 multi-location restaurant groups (find them on Google Maps — any place with 3+ locations). Pitch: "We respond to every review within 24 hours. You approve in 1 tap. $149/location/mo."

Week 3-4 — Scale to 10 locations:
9. Each happy customer gives you a referral. Ask for 1 intro to another business owner.
10. Post in local business Facebook groups and chambers of commerce.
11. At 10 locations ($1,490/mo), the Claude API cost is ~$30-50/mo. Margin is ~96%.

**Time to first $:** 2-3 weeks
**Effort:** Medium (one weekend of coding, then sales)
**Risk:** Low-Medium (API integration is the only new piece)
**Scaling:** 50+ locations is a one-person operation. Beyond that, hire a VA for onboarding.

---

### Play 5: AI Phone Call Summary Service for Contractors

**What:** Plumbers, electricians, HVAC techs take calls all day and forget details. They forward calls through your number, AI transcribes and summarizes each call into a job ticket with customer name, address, issue, requested time, and urgency.

**Who buys:** Solo contractors and small trade businesses (1-5 techs) who answer their own phones.

**Pricing:** $99/mo per business. 10 contractors = $990/mo. 20 = $1,980/mo.

**Tools you need:** Twilio phone number ($1-5/mo per number), Whisper STT or Deepgram (you have both in your stack), Claude for summarization, Flutter web dashboard or simple email/SMS delivery.

**Step-by-step plan:**

Week 1 — Build the pipeline:
1. Buy a Twilio number. Set up call forwarding: contractor's customers call your Twilio number, it rings through to the contractor's phone, call is recorded.
2. After each call ends, Twilio sends the recording to your server.
3. Server runs Whisper/Deepgram to transcribe, then Claude to extract: customer name, phone, address, issue described, urgency, preferred appointment time.
4. Send the contractor an SMS + email with the structured job ticket within 2 minutes of the call ending.
5. Optional: Flutter web dashboard showing all job tickets for the week.

Week 2 — Get your first 3 paying customers:
6. Walk into 5 local plumbing/HVAC/electrical shops. Say: "You're taking calls while working and forgetting details. I'll give you a dedicated number that forwards to your phone, and after every call you get a text summary with the customer's info and what they need. $99/mo."
7. Free 2-week trial. The call summaries sell themselves — they'll see the value on day 1.
8. Post in contractor Facebook groups and trade subreddits.

Week 3-4 — Scale:
9. Each contractor refers you to 2-3 others in adjacent trades.
10. At 20 contractors ($1,980/mo), Twilio costs ~$100/mo, transcription API ~$50/mo. Margin is ~88%.

**Time to first $:** 2-3 weeks
**Effort:** Medium
**Risk:** Medium (Twilio setup + call quality)
**Scaling:** 50+ contractors is manageable. The infrastructure scales linearly.

---

### Play 6: Menu / Price-Sheet Digitization

**What:** Take a photo of a paper menu, handwritten price sheet, or old PDF menu. AI extracts it into a clean digital format — updated PDF, web page, or Google Sheet. One-off service, fast turnaround.

**Who buys:** Restaurants, food trucks, bars, salons, any business with a physical or outdated price list.

**Pricing:** $75-150 per menu. Volume play — 5 per week = $375-750/week.

**Tools you need:** Claude or Gemini vision (you have both), a simple web page or just email-based workflow.

**Step-by-step plan:**

1. Walk into 10 local restaurants/food trucks with handwritten or clearly outdated menus. Say: "I'll digitize your menu into a clean PDF and a web page for $100. 24-hour turnaround."
2. Take a photo of their menu on the spot. Or they text/email it to you.
3. Feed the photo into Gemini vision (or Claude if it gets image support). Prompt: "Extract every item, price, and description from this menu. Output as clean structured data."
4. Review the output, fix any errors (the verify step from your Apex Feature A plan — same concept).
5. Generate a clean PDF (Canva template or simple HTML-to-PDF) and a one-page web site (Carrd or Google Sites).
6. Deliver within 24 hours. Upsell: "I can also put this on your Google Business Profile and update it whenever you change prices — $25/mo."

**Time to first $:** 1 week
**Effort:** Low
**Risk:** Low
**Scaling:** At 20+ menus/week, build a self-service web app where they upload a photo and get the digital version back. Still AI-powered, you just review edge cases.

---

## TIER 3 — CASH IN 4-8 WEEKS (real recurring, focused build)

### Play 7: AI-Generated Local SEO Content Service

**What:** Write weekly blog posts for local businesses optimized for local search. AI drafts based on their business + local keywords, you edit, they publish on their website or Google Business Profile.

**Who buys:** Local businesses with websites but no blog/content strategy — dentists, plumbers, HVAC, lawyers, real estate agents, restaurants.

**Pricing:** $300-500/mo per client for 4 posts/month. 5 clients = $1,500-2,500/mo. 10 clients = $3,000-5,000/mo.

**Tools you already have:** Claude for writing, your research skills, basic SEO knowledge.

**Step-by-step plan:**

Week 1 — Build the system:
1. Create a prompt template: business type, location, services, target keywords, tone. Feed into Claude to generate a 500-800 word blog post.
2. Create 3 sample posts for different business types (e.g., "Best Craft Beer Selection in [Town]" for a bar, "When to Call a Plumber vs DIY" for a plumber, "5 Signs You Need a Root Canal" for a dentist).
3. Set up a 1-page landing page showing samples.

Week 2 — Get your first 3 clients:
4. Find 20 local businesses with websites but no blog. Check if they rank for their own service + city (most won't).
5. Email/DM each: "I noticed you're not showing up when people search '[service] in [town]'. I write weekly blog posts that get you on Google's first page. $300/mo for 4 posts. First post free — here's one I wrote for your business."
6. The free sample is the hook. If it's good, they'll want more.

Week 3-4 — Deliver and scale:
7. Each client gets 4 posts/month. With Claude, writing 4 posts takes 2 hours. 5 clients = 10 hours/month for $1,500-2,500/mo.
8. Post in local business Facebook groups: "I help local businesses rank on Google with weekly blog content. DM for a free sample post."
9. At 10 clients ($3,000-5,000/mo), hire a VA to handle editing and publishing. You just manage the AI prompts and client relationships.

**Time to first $:** 2-3 weeks
**Effort:** Low-Medium
**Risk:** Low
**Scaling:** 20+ clients is the ceiling for solo. Content quality drops if you stretch further without help.

---

## HONEST ASSESSMENT — What to do first

If you need cash THIS WEEK: do Play 1 (Content Repurposing) or Play 3 (Veteran Resumes). Zero build, sell tomorrow, get paid by Friday.

If you can wait 2 weeks: Play 2 (Google Business Profile) is the easiest sell. Walk into any business, show them their broken profile, fix it on the spot. $200/mo retainer with near-zero work.

If you want recurring revenue with a real product: Play 4 (ReviewGuard) is the fastest path. One weekend of coding, then 2 weeks of door-to-door sales. $149/location/mo adds up fast.

If you want the highest hourly rate: Play 6 (Menu Digitization). $100 for 30 minutes of work = $200/hour. But it's one-off, not recurring.

The mistake to avoid: building all of these. Pick ONE, execute it fully, get paying customers, then decide if you want to add a second.

---

## Revenue Projection (Conservative)

| Play | Time to first $ | Clients to start | Monthly revenue at 10 clients | Hours/week at 10 clients |
|---|---|---|---|---|
| 1. Content Repurposing | 3-5 days | 3 | $1,500-$5,000 | 3-5 |
| 2. Google Profile Opt | 1 week | 5 | $1,500-$3,000 | 5 |
| 3. Veteran Resumes | 2-3 days | 5/day | $375-1,500/day | 10-15 |
| 4. ReviewGuard | 2-3 weeks | 3 | $1,490 | 2-3 |
| 5. Call Summary | 2-3 weeks | 3 | $990 | 3-5 |
| 6. Menu Digitization | 1 week | 5 | $375-750/week | 3-5 |
| 7. Local SEO Content | 2-3 weeks | 3 | $1,500-2,500 | 10 |

All of these can hit $1,000-2,000/mo with 3-5 paying customers. That's real money, not vanity metrics.