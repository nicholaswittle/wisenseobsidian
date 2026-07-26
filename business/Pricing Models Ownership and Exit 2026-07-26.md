---
title: Pricing Models — Ownership and Exit Options
tags: [business, pricing, ownership, code, sell, exit]
aliases: [Code Ownership, Buyout Model]
date: 2026-07-26
status: active
---

# Pricing Models — Ownership and Exit Options

Related: [[business/Web Redesign — Recurring Model Proposal 2026-07-23]], [[business/Small Business Websites Niche 2026-07-21]], [[business/Restaurant Ordering Template Product Strategy 2026-07-24]], [[prospects/Cold Outreach Target List]], [[NOW]]

---

## The Ownership Question

Three ways to handle who owns the code:

### Model A: Subscription (You own the code)
- Client pays monthly, you host it, you own it
- If they stop paying, you take it down
- They never get the source code
- Pro: recurring revenue forever, low churn
- Con: they may feel locked in

### Model B: One-time Sale (They own the code)
- Client pays one fee, gets the code, hosts it themselves
- You deliver the files and walk away
- Pro: big upfront check
- Con: no recurring revenue, they might break it and blame you

### Model C: Hybrid (You own during term, they can buy out)
- Subscription with a buyout option
- If they want to leave and take the code, they pay a one-time buyout fee
- If they don't want the code, they just cancel and you take it down
- Pro: recurring revenue + exit option for the client
- Con: slightly more complex to explain

---

## Recommended: Model C (Hybrid with Buyout)

### Inbound (they come to you)
- $299 setup + $79/mo, 3-month minimum
- You own the code, you host it
- Buyout option: $500 one-time and they get the source code + you transfer the Vercel project

### Outbound (you cold call them)
- $0 setup + $99/mo, 3-month minimum
- You own the code, you host it
- Buyout option: $700 one-time and they get the source code + you transfer the Vercel project

### Why buyout pricing differs
- Outbound clients pay more monthly but less upfront, so the buyout is higher
- Inbound clients paid setup, so the buyout is lower

---

## Buyout Clause (put in writing)

"WiSense LLC owns the website code and hosting. The monthly fee licenses its use, hosting, and maintenance. At any time after the 3-month minimum, the client may request a code buyout. The buyout fee is $500 (inbound) or $700 (outbound), paid one-time. Upon buyout, WiSense transfers the source code repository and Vercel project to the client. After buyout, WiSense is no longer responsible for hosting, maintenance, or updates. The client may continue to pay $79/mo for ongoing hosting and maintenance if they prefer not to self-host."

---

## Revenue Comparison (per client)

### Inbound client stays 12 months
- Setup: $299
- Monthly: $79 × 12 = $948
- Total: $1,247

### Outbound client stays 12 months
- Setup: $0
- Monthly: $99 × 12 = $1,188
- Total: $1,188

### Inbound client buys out after 6 months
- Setup: $299
- Monthly: $79 × 6 = $474
- Buyout: $500
- Total: $1,273

### Outbound client buys out after 6 months
- Setup: $0
- Monthly: $99 × 6 = $594
- Buyout: $700
- Total: $1,294

---

## Why This Works

1. 3-month minimum protects you from doing free work
2. Buyout option makes clients feel safe (no lock-in)
3. You keep recurring revenue from clients who don't want to self-host
4. You get a big check from clients who want to own it
5. Different pricing for inbound vs outbound reflects the cost of customer acquisition

---

## Quick Reference Card (for cold calls and pitches)

**If they come to you (inbound):**
"$299 setup, $79/month, you keep your domain, I host and maintain it. 3-month minimum. If you ever want the code, $500 buyout and it's yours."

**If you reach out to them (outbound/cold call):**
"No setup fee, $99/month, I host and maintain it. 3-month minimum. If you ever want the code, $700 buyout and it's yours."

**If they ask "Do I own the website?":**
"You own your domain and your content. I own the code and hosting. You can buy the code anytime after 3 months for $500-700. Most clients just keep the monthly plan because I handle everything."

**If they ask "What if I cancel?":**
"After the 3-month minimum, you can cancel anytime. I take the site down. You keep your domain. If you want to keep the site but own it yourself, the buyout option is $500-700."

---

## Is This a Sound Business Idea?

Yes. Here's why:

1. **Low risk** — 3-month minimum means you never do free work. Worst case: someone pays 3 months and cancels. You made $237-297 for a few hours of config work.

2. **High margin** — hosting is $20/mo (Vercel Pro, flat). At $79-99/mo per client, margin is 75-80%. 10 clients = $790-990/mo on $20 cost.

3. **Recurring revenue** — MRR is what makes a business valuable, not one-time sales. $79-99/mo per client compounds. 10 clients = ~$9,500-11,900/year in predictable revenue.

4. **Exit option for clients** — the buyout removes the #1 objection ("what if I want to leave?"). Most won't use it, but having it makes the sale easier.

5. **Inbound vs outbound pricing** — standard business practice. Inbound leads are worth more (pre-sold), outbound you discount (acquisition cost). Nobody questions this.

6. **You already have the product** — the restaurant ordering platform is built. Each new client is config work, not new development. 2-4 hours per client.

7. **Infrastructure cost is $0** — Cloudflare Workers + D1 free tier. Vercel Pro is $20/mo flat. That's your only cost.

8. **Breakeven at 1 client** — one outbound client at $99/mo covers Vercel Pro ($20) and puts $79 in your pocket. Everything after that is profit.

**The only risk:** scope creep on "maintenance." Cap it: "up to 2 content updates/month, 2 business day response." Anything beyond that is $25-50 per update.

This is a sound, low-risk, high-margin recurring revenue business that you can run from your phone on your off-weekends.