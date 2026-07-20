---
title: Governance Protocol
tags: [governance, business, risk, reference]
aliases: [WiSense Governance Protocol, Business Priorities, Risk Tolerance]
---

# WiSense Governance Protocol

Source: `C:\development\projects\wisense_governance_hub\meta_prompts\wisense.governance_protocol`

## Business Priorities

All WiSense AI work is evaluated against three equally weighted pillars:

1. **Code Quality** — correct, secure, minimal, maintainable.
2. **Business Viability** — does this change move a WiSense product closer to generating value?
3. **Scalability** — will this hold as the product or user base grows, or does it create future debt?

A technically correct change that undermines business viability or creates unscalable architecture must be flagged before implementation.

## Risk Tolerance

| Risk Level | Definition | Required Action |
|---|---|---|
| Low | Isolated change, easily reversible | Builder proceeds after plan approval |
| Medium | Touches shared logic or multiple files | Gemini review recommended before ratification |
| High | Deletion, dependency change, auth/data layer | Gemini review required + explicit Architect ratification |

## Gemini Review Communication

Before proposing any change, the Builder must explicitly state:
1. The risk level (Low / Medium / High)
2. Whether a Gemini review is recommended, required, or not needed

The Architect may override this recommendation in either direction — requesting a Gemini review on a low-risk change or waiving it on a high-risk change. The Builder never skips this disclosure.

## Ratification Rules

- Minor changes (single-file edits, copy/text updates): Architect approval via "confirmed" or equivalent.
- Major changes (deletions, refactors, new dependencies): Architect must reply explicitly. Silence is not approval.
- Destructive actions without ratification within 60 seconds: prohibited.

## Minimalist Standards

- Write only what the task requires.
- No speculative features or preemptive abstractions.
- No what-comments; only non-obvious why-comments.
- No stubs, TODOs, or partial implementations delivered as output.
- Prefer editing existing files over creating new ones.

Related: [[WiSense Governance — Rules and Protocols]], [[Head of Team Directive]], [[Team Workflow]]