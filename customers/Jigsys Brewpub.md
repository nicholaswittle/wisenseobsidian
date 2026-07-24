---
title: Jigsy's Brewpub
tags: [customers, apex, pilot]
aliases: [Jigsy, Jigsys]
date: 2026-07-20
updated: 2026-07-24
app: Apex Scheduler + website ordering concept
stage: interested-demo-built
---

# Jigsy's Brewpub

## Context

- **Contact / role**: Pub owner (pilot) — name/details as Nicholas adds them
- **App**: [[Apex Scheduler]]
- **Why they care**: Staff scheduling, swaps, time clock, sidework — reduce owner hours on ops

## Interviews / feedback

### 2026-07-24 — owner presentation and hardware confirmation

- Nicholas presented the owner deck and live customer/staff ordering demo in
  person to two of the three ownership stakeholders.
- Both owners appeared positive and are waiting for their son, the remaining
  stakeholder, to review it.
- The owners described online ordering as a local competitive gap and were
  especially receptive to the ability to pause ordering during a rush.
- They were comfortable using the existing iPad for a staff Home Screen web
  app and with accepted orders printing as ordinary kitchen tickets.
- The proposed 99-cent accepted-order fee was explained. They asked whether
  online-order fees are normal but did not reject the proposal; exact approval
  and wording remain pending.
- On-site photos confirm an iPad running Square POS in a Square Stand. Square
  showed both the Stand and Printer connected.
- The printer is a Star Micronics **TSP100 futurePRNT-series** thermal printer.
  The exact TSP143 variant and connection type still require a label/photo and
  Square printer-settings check.
- Overall feasibility remains positive. Push notifications are compatible
  with an installed iPad web app, but automatic printing of an unpaid pickup
  order through Square must be proven with the actual merchant setup.
- No new printer purchase is recommended until that live validation is
  complete.

### 2026-07-23 — website and online-ordering interest

- Emily showed the owner the website concept; the owner liked it.
- Nicholas later discussed the site directly while helping at Jigsy's. The owner
  again said she liked it.
- The design looked as though it might support ordering. Nicholas clarified
  that ordering was not live and said he could build it if she wanted, planting
  the idea without making a hard sale.
- A separate customer/staff ordering demo is now live:
  https://jigsys-ordering-demo.nicholaswittle.chatgpt.site
- The demo now covers the full 52-item priced ordering menu, categorized
  availability controls, accept/reject, printable daily archives, staff-set
  pickup estimates, and customer accepted/rejected status.
- The owner-facing position is now **one website, optional ordering**: staff
  can pause during a rush and all public ordering buttons disappear while the
  menu, hours, phone, directions, and restaurant site remain. Staff can reopen
  ordering during a slower period.
- A revised five-page owner leave-behind and private planning pack are stored
  in `output/` and match the pay-at-pickup / 99-cent accepted-order model.
- Superseded by the 2026-07-24 meeting: two ownership stakeholders now show a
  positive signal, but no formal pilot commitment or production connection
  exists yet.

### 2026-07-20 — placeholder

- No structured interview filed yet. Capture first owner conversation here (Mom Test: past behavior, not hypothetical praise).
- Baseline metrics target: see Apex `JIGSYS_BASELINE.md` / Gate0 once pilot starts — owner time saved, swap resolution time.

## Outcome

- Keep / kill / pivot: _pending first pilot week_
- Metric touched: _Shift Swap Resolution Time; Owner Time Saved (target >5 hrs/week)_ — from [[business/WiSense Service as a Software Execution Strategy]]

## Related work

- [[Jigsys Website Concept]] — independent redesign of Jigsy's public site (live at https://jigsyssite.vercel.app · GitHub `nicholaswittle/jigsysite`). Passes A–D through 2026-07-22: Board menu, Google trust, What’s good here / FAQ, downstairs = skill games + TV lounge (no live music). Concept/portfolio piece; not the official site. Confirm peanut butter pie still served before client pitch.
- [[business/Jigsys Ordering Demo — Build Record 2026-07-23]] — isolated
  customer ordering and staff Accept & Print demo, Vercel deployment, final
  99-cent model, printer plan, and production gaps.

Related: [[customers/_Index]], [[NOW]], [[output/Launch Readiness — COMMS LINK and Apex 2026-07-20]], [[Apex Security Audit 2026-07-19]]
