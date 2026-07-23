---
title: Website Business Setup Guide 2026-07-21
tags: [business, websites, setup, guide, email, intake, workflow]
aliases: [Website Business Setup, Client Intake Setup]
date: 2026-07-21
status: active
---

# Website Business — Setup Guide

> How to look professional and handle inquiries when Jigsy's (or anyone) advertises your website services. Set this up BEFORE anyone posts your contact info.

Related: [[business/Small Business Websites Niche 2026-07-21]], [[business/AI Side Hustles Remote Only 2026-07-21]], [[business/Easy AI Money Plays 2026-07-21]], [[NOW]]

---

## Step 1 — Create a Dedicated Email

Don't use personal email. Create a separate business email:

- Go to Gmail and create: nicholaswittle.websites@gmail.com or wittleweb@gmail.com or nicholas.builds@gmail.com
- Keep it simple and professional
- This is where all business inquiries go

## Step 2 — Set Up an Auto-Reply

In Gmail settings, turn on "Vacation responder" but use it as auto-reply:

```
Thanks for reaching out about websites. I build fast, modern websites for small businesses — $800 flat for a full site, 1-week turnaround. Tell me about your business and I'll get back to you within 24 hours. — Nicholas Wittle
```

Even if you're at work or putting kids to bed, every inquiry gets an instant professional response.

## Step 3 — Create an Intake Form

Create a Google Form (free). Title: "Website Project Inquiry"

Questions:
1. Business name
2. What does your business do?
3. Do you have a current website? (If yes, paste the link)
4. Do you have a domain name already?
5. What pages do you need? (Home, Menu/Services, About, Contact, Gallery, Other)
6. Any websites you like the look of? (Paste links)
7. Your colors/branding?
8. Anything else I should know?
9. Best phone number to reach you

When someone emails, reply with: "Great — fill out this quick form and I'll get started: [link]"

## Step 4 — Email Response Template

Save this in Notes app. Copy-paste when inquiries come in:

```
Hi [name],

Thanks for reaching out. I build modern websites for small businesses — clean, fast, mobile-friendly, no monthly fees unless you want updates.

Here's how it works:
1. You fill out this form: [Google Form link]
2. I build a demo of your site (2-3 days)
3. You see it live before you pay anything
4. If you like it, it's $800. I'll point your domain to it and it goes live.
5. If you want changes, I adjust for free before we finalize.

Turnaround is about 1 week from start to live site.

Looking forward to hearing about your business.

— Nicholas
```

## Step 5 — Create a Simple Link Page

Go to linktr.ee (free) and create a page with:
- Your business email
- Your intake form link
- Screenshots or link to the Jigsy's site (once live)

One link to share anywhere. Put it in email signature, website footer, anywhere.

## Step 6 — What to Tell Jigsy's to Post

Give them this when they're ready to post on social media:

```
We're so happy with our new website. Our friend Nicholas Wittle built it from scratch — if your business needs a website, reach out to him at [your email]. He does great work and it's affordable.
```

Short, real, gives people exactly what they need to contact you.

## Step 7 — Vercel + Domain Setup (When You Have a Client)

### Deploy to Vercel
1. Push the React app to a GitHub repo
2. Go to vercel.com, sign in with GitHub
3. Click "Add New Project" → select the repo → click "Deploy"
4. You now have a live URL like `jigsys.vercel.app`

### Point Their Domain
1. Ask: "Where did you buy your domain?" (GoDaddy, Namecheap, Google, etc.)
2. In Vercel project: Settings → Domains → type their domain → click "Add"
3. Vercel gives you DNS records:
   - Type: A, Name: @, Value: 76.76.21.21
   - Type: CNAME, Name: www, Value: cname.vercel-dns.com
4. Log in to their domain registrar (or they do it) → DNS settings → add the records
5. Wait 5-30 minutes for DNS to propagate
6. Their domain now loads your Vercel site
7. HTTPS is automatic (Vercel handles SSL for free)

### If They Don't Have a Domain
- They buy one on Namecheap (~$10/yr)
- You set everything up
- Total cost to them: $10/year for domain, hosting is free

### If They Have an Existing Site
- Build your version, deploy it, get the demo link
- Show them, get the okay
- THEN point the domain
- Don't take down their existing site without them seeing the new one first

## Jigsy's Specific — Domain + Vercel

- Jigsy's site is already built (React app at `C:\development\projects\jigsy`)
- Deploy to Vercel on your account
- They keep owning their domain — you just point it at Vercel
- You host it on your Vercel (free). They pay $0 hosting.
- If they ever want to take it elsewhere, hand over the code
- Add footer credit: "Built by Nicholas Wittle — Websites for small businesses"

## Checklist — Do All of This Before Jigsy's Posts

- [ ] Create business email (5 min)
- [ ] Set up auto-reply (5 min)
- [ ] Make the Google Form (15 min)
- [ ] Write the response template in Notes app (5 min)
- [ ] Create the Linktree (10 min)
- [ ] Deploy Jigsy's site to Vercel
- [ ] Point their domain
- [ ] Get their social media post live
- [ ] Screenshot their post for testimonial

Total setup time: ~40 minutes (minus Vercel deploy + domain)

## Hosting = Recurring Revenue (For Future Clients)

- Vercel hosting: free for small sites
- Charge clients $25-50/mo for "hosting and maintenance"
- Pure profit — hosting costs nothing
- 10 clients = $250-500/mo recurring
- Updates (menu changes, photos, prices): $25-50 per update or 2/month in maintenance fee