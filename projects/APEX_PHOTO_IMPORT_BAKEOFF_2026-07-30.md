---
type: record
title: "Apex photo import — model bakeoff and the two prompt bugs"
tags: [apex-v2, ai, parse-schedule, gemini, claude, measurement]
date: 2026-07-30
status: shipped
---

# Photo import — what actually fixed it

**Shipped:** `claude-opus-5` at `effort: low`, a rewritten prompt, and a
server-side contradiction check. `parse-schedule` at `07fa32a`.

**Cost: ~10¢ per schedule import, one-time per venue** (measured twice against
the credit balance, not estimated). Every other AI function in Apex runs on
Haiku at ~a tenth of a cent. A venue that photographs one calendar during
onboarding and then schedules in the app costs about a dime, ever.

> [!warning] A failed import bills exactly like a successful one
> The client giving up at 120s does not stop the model. A timed-out attempt
> completes server-side and charges in full. Three retries cost 30¢, and the
> logs show only a rate-limiter charge with no status — which is why the cause
> of a hang has to be inferred unless it is logged. It now is.

## The model was never the problem

Seven commits flipped this function between Gemini and Claude. Two prompt bugs
were doing the damage:

1. **A month calendar was being read with a week anchor.** The prompt said
   "this week starts Monday X; dates belong to it or the week after" while the
   real input is a 31-cell wall calendar. Anchoring each entry to the date
   printed in its own cell took the read from **10 shifts to 28**.
2. **Cancellation was an afterthought.** Struck writing stays perfectly legible
   under the line, so a model that reads first and judges second resurrects
   cancelled shifts. Making cancellation a decision stated per cell *before any
   hours are read* closed most of the rest.

Neither had anything to do with which model was running.

## 🔴 ListModels advertises models that generateContent 404s

**This is the trap worth remembering.** Every Gemini model ID in the deployed
function returned `404 "no longer available"` — `gemini-2.0-flash`,
`gemini-1.5-pro`, `gemini-2.0-pro-exp-02-05`. The "Gemini primary" branch had
**never once executed in production**: it 404'd three times per request and
fell through to Claude silently.

They survived review because `GET /v1beta/models` still lists them. Probing all
12 listed Gemini models with a one-token request: **4 dead, 8 live** — every
2.5-series model is retired but still advertised.

**A model listing is not proof a model runs.** Probe with a real one-token
generate call before trusting an ID.

Also disproved along the way: the $10 AI Studio credit was fine the whole time
(`serviceTier: "standard"` on a successful call proves paid tier), and the key
was valid. Three separate misdiagnoses — "OAuth token", "billing", "intermittent
auth" — before the real cause, which was a **stale `GEMINI_API_KEY` in the
Windows environment shadowing the correct one in the key file**. Both 53 chars,
both `AQ.Ab8…`, differing at character 10.

## Scored against ground truth, 10 runs each

Answer key confirmed by the owner: Morgan is crossed out on the 13th and 14th,
and everything from the 1st to the 9th is crossed out.

| | claude-opus-5 (low) | gemini-3.5-flash |
|---|---|---|
| real shifts found in **every** run | **23/24** | 0/24 |
| distinct invented entries | **3** | 11 |
| worst invented rate | 60% | 90% |
| 10 runs | 68s | 191s |

Claude wins on the error that matters: it puts a crossed-out person back on the
schedule 10–20% of the time where Gemini does it **90%** of the time. Gemini's
cost advantage is real but irrelevant at ~$5/venue/lifetime.

**Claude is not clean either** — roughly one wrong entry per import:

```
INVENTED   Aug 1  Marsha   60% of runs   <-- crossed out
INVENTED   Aug 13 Morgan   20% of runs   <-- crossed out
INVENTED   Aug 14 Morgan   10% of runs   <-- crossed out
FLAKY MISS Aug 14 Avi      missing in 30% of runs
```

Marsha/Aug 1 failed at 60% on **both** model families — two unrelated
architectures failing identically at the same rate is the image, not the model.

## Two methodology errors worth not repeating

- **Ranking by shift count picked the worse model.** The first bench scored
  "more shifts found" as better; the winner had silently shifted an entire row
  by one day. More rows looked like better extraction and was actually date
  drift plus junk. Precision and recall against a confirmed answer, or nothing.
- **Two clean runs got reported as "perfect".** The next three runs were not.
  On a task where the model's cancellation call is the unstable part, a single
  score says almost nothing — what matters per entry is how often it appears.

## The fix that survives model churn

`scripts/score_parse.mjs` + `scripts/schedule_truth.json` score any
model/prompt against the confirmed answer; `scripts/variance_parse.mjs` runs it
N times and reports per-entry flip rates. Adding a provider is ~15 lines.

**Today's model choice will be stale in six months. The harness will not.**

Because ~1 error per import is the normal case, the product answer is not a
better model — it is making the error visible. `struck` now carries a day and a
name instead of prose, so the server can catch the reply contradicting itself
(marks a day CANCELLED, then emits a shift on it — ~60% of runs). Those rows
render outlined with the reason instead of blending into the list.

## Also fixed today

- **Imports were hanging.** 2400px/85% produced a 1–3 MB base64 payload
  crossing the network twice. Dropped to 2000/80. The photo that reads with
  zero errors is 1974px on its long edge and the model downsamples above
  2576px, so the earlier bump to 2400 bought compression, not legibility.
- **`parse-menu` still runs `claude-sonnet-4-5`** (legacy). Not urgent — a menu
  is a much easier read — but worth revisiting.

## Update 2026-08-01 — the parse-menu TODO is closed, and the dead id had spread

Both loose ends from "Also fixed today" are now done.

**`parse-menu` moved to `claude-opus-5` at `effort: low`** (`296fdd1`), with
Gemini as fallback rather than primary — the same order this bakeoff settled
for `parse-schedule`. It had still been running `claude-sonnet-4-5` with Gemini
*first*. `max_tokens` went 4096 → 16000 for the image path, because Opus 5
thinks by default and the cap covers thinking plus output together; the old
budget would have truncated mid-JSON and read as bad OCR. A `stop_reason:
"refusal"` check was added ahead of parsing, and every Claude failure now
returns null and falls through to Gemini instead of throwing.

**The dead Gemini id had been copied into `parse-menu` too** (`a13582c`).
Re-probed 2026-08-01 with a one-token generate call:

| id | result |
|---|---|
| `gemini-2.0-flash` | **404** |
| `gemini-2.5-flash` | **404** |
| `gemini-3.5-flash` | live |

Same trap as before, one function over. Worse here, because
`tryGeminiParseMenu` returns null on any error — a dead id is
indistinguishable from "Claude handled it", and nothing surfaces until Claude
is *also* unavailable.

> [!caution] Not scored
> The schedule change in this note was measured over 10 runs against an
> owner-confirmed key. The `parse-menu` change was **not** — it rests on this
> note plus recollection. There is no menu photo and no `menu_truth.json`. By
> this note's own standard that is not evidence. Needs a Jigsy's menu photo and
> a confirmed item list before it counts as verified.

`parse-schedule` was deliberately left untouched. Its comment at line 265 —
*"effort buys thinking tokens, not accuracy"* — contradicted the change that
would otherwise have been made, and the comment matched the code.

## Related

- [[APEX_V2_LIVE_CATALOG_SWEEP_2026-07-29]] — the other "audit the running
  thing, not the description of it" finding
- [[APEX_SESSION_2026-07-29_30_FULL_RECORD]]
