---
title: Tripartite Protocol
tags: [governance, protocol, reference]
aliases: [WiSense Tripartite Architecture, MCA, MDT]
---

# WiSense Tripartite Protocol

The operating protocol for all WiSense work. Source of truth: `C:\Users\nikwi\CLAUDE.md` and `AGENTS.md`, plus `C:\development\projects\wisense_governance_hub\meta_prompts\`.

## Three branches

| Branch | Role | Mandate |
|---|---|---|
| **Executive** (Chief of Staff) | AI governor | Decompose requests, delegate, integrate, block on unresolved findings, author Completion Report |
| **Legislative** (Architect) | Claude | Design + implement code per WiSense minimalist standards |
| **Judicial** (Auditor) | External AI — never self-audit | MCA + MDT audit before delivery |

## External audit chain

1. Gemini (`gemini-2.5-flash`) — primary
2. Groq (`llama-3.3-70b-versatile`) — if Gemini unavailable
3. OpenRouter (`openrouter/auto`) — if Groq fails

## Key rules

- No change reaches the Architect without explicit approval. Silence ≠ approval.
- Destructive actions without ratification within 60s: prohibited.
- Gemini is a mandatory partner on all builds, not optional.
- Travel/booking logic must route through ToolRegistry (`lib/core/ai/agent_bridge.dart`) with typed transforms, session binding, Human Veto gates.

## Three pillars (every change evaluated against)

1. Code Quality — correct, secure, minimal, maintainable
2. Business Viability — does this move a product toward value?
3. Scalability — will this hold as the product grows?

Related: [[Audit Findings Loop]]