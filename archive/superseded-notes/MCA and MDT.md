---
title: MCA and MDT
tags: [governance, audit, MCA, MDT, reference]
aliases: [Minimalist Conformance Audit, Material Defect Triage]
---

# MCA — Minimalist Conformance Audit

Source: `C:\Users\nikwi\CLAUDE.md`, `C:\Users\nikwi\AGENTS.md`

Verify that Legislative output complies with WiSense minimalist standards:

| Check | Pass condition |
|---|---|
| Scope containment | No features or abstractions beyond the stated task |
| Comment hygiene | No what-comments; only non-obvious why-comments |
| File discipline | No unnecessary new files created |
| Completeness | No stubs, TODOs, or partial implementations |
| Backwards-compat hygiene | No shims or re-exports for removed code |

# MDT — Material Defect Triage

Screen all output for defects that would affect correctness or security:

| Check | Pass condition |
|---|---|
| Logic correctness | No off-by-one errors, wrong conditions, or missed edge cases |
| Security | No OWASP Top 10 vulnerabilities introduced (injection, XSS, etc.) |
| Boundary validation | User input and external API responses are validated; internal calls are trusted |
| Error handling | No suppressed errors at system boundaries; no unnecessary try/catch internally |
| Regressions | No observable change to behavior outside the stated task scope |

## Audit outcome

- **PASS** — output forwarded to Executive for Completion Report.
- **FAIL** — findings returned to Legislative branch with specific, actionable notes. Cycle repeats until PASS.

Related: [[WiSense Governance — Rules and Protocols]], [[Tripartite Audit Chain]], [[Travel Data Integrity]]