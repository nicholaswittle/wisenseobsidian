---
title: Startup Playbook
tags: [business, startup, playbook, lessons]
date: 2026-07-20
---

# Startup Playbook — What Makes a Great Startup

Patterns, anti-patterns, and principles for building WiSense into a sustainable business.

## Core Principles

### 1. Solve a real pain you've felt yourself
- COMMS LINK exists because Nicholas (Marine veteran, first responder) knows the decompression gap firsthand
- Apex exists because Jigsy's Brewpub needs scheduling and existing tools are enterprise-priced
- If you wouldn't use it yourself, don't build it

### 2. Ship before you're ready
- Perfect is the enemy of shipped
- COMMS LINK is 10 commits from Play Store — push it, learn, iterate
- "If you're not embarrassed by your first version, you launched too late" — Reid Hoffman

### 3. Privacy as a moat
- In a world of data-harvesting apps, "no cloud, no accounts, no data retention" is a differentiator
- Lean into it in marketing — it's not a limitation, it's the feature
- COMMS LINK and Apex can both lead with this

### 4. Code reuse = cost savings
- wisense_core + wisense_ui shared across all apps
- One fix propagates everywhere (fork reconciliation proved this)
- AI quota is finite — reuse saves tokens and time

## What Makes Startups Fail (from WiSense experience)

| Anti-pattern | Example | Lesson |
|---|---|---|
| Overbuilding before validation | wisense-os (custom desktop agent OS) | Killed 2026-07-19 — overbuilt for a market that didn't exist |
| Premature abstraction | my_ai, command_center | Killed — speculative architecture with no users |
| Forking shared code | New Horizon vendored packages | Reconciled 2026-07-20 — forks create silent divergence |
| Ignoring security | Apex RLS gaps | Multi-tenant data leak risk — must fix before launch |
| Never pushing to origin | COMMS LINK 10 commits ahead | Local-only work is invisible — push early, push often |

## Growth Stages (WiSense-specific)

### Stage 1: Ship one app (NOW)
- COMMS LINK to Google Play internal testing
- Learn the store flow, signing, assets
- Zero code blockers — just packaging
- Lead with purpose-built privacy (Info-Tech 2026 #7)

### Stage 2: Prove multi-tenancy (Apex)
- Fix RLS + claim races ([[Apex Security Audit 2026-07-19]]) — resilience is the product (Info-Tech #2 / #5)
- Get Jigsy's Brewpub as pilot customer
- Validate subscription pricing; test outcome language (conflict-free week) per Info-Tech #8

### Stage 3: Expand the portfolio
- New Horizon to web (Vercel) — affiliate revenue = Service as Software already
- Cross-promote between apps
- Bundle strategy
- Vendor diversification notes for Duffel/Supabase (Info-Tech #1)

### Stage 4: Scale
- Cloud CI for iOS
- Paid Supabase tier
- Content marketing + app store optimization
- Evaluate which apps earn enough to justify continued dev
- Only then consider narrow outcome agents (Info-Tech #3) — never another agent OS

Trend source: [[business/Info-Tech Tech Trends 2026 — WiSense Playbook]]

## Metrics That Matter

- **Activation**: Did the user complete the core action? (COMMS LINK: first debrief. Apex: first shift scheduled.)
- **Retention**: Week-1, week-4 return rate
- **Revenue per app**: Which apps pay for themselves?
- **AI quota efficiency**: How much does each shipped feature cost in tokens?
- **Code reuse rate**: % of shared package usage across apps (target: >60%)

## Books/References to Add

- [ ] The Lean Startup — Eric Ries
- [ ] The Mom Test — Rob Fitzpatrick
- [ ] Make Something People Love — Alexis Ohanian
- [ ] Zero to One — Peter Thiel
- [ ] Hooked — Nir Eyal

Related: [[business/Business Model Canvas]], [[business/Ideas Log]], [[business/Revenue Ideas — 12 Buildable B2B Plays 2026-07-20]], [[business/2026 Commercial B2B Software Ideas]], [[business/Info-Tech Tech Trends 2026 — WiSense Playbook]], [[Abandoned Projects — Lessons]], [[DECISIONS]]