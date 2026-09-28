# jev-triage

**Jev-powered submission triage — one module, every project.**

A small, dependency-light module that classifies incoming user submissions
(bug reports, feature requests, complaints, questions) with calibrated
judgment calls and routes each to the right handler. Built once, reused
everywhere: comfyui-video-ui, tierllama, and any future project with a
support intake surface.

**Status: SKELETON / MANIFESTO (2026-09-29).** The contract below is stable;
implementation comes in 2–3 milestones (see docs/PLAN.md). First consumer:
comfyui-video-ui's support tab; exportable to tierllama afterwards.

---

## Why this exists (Gui, 2026-09-29)

Both products will have customer-support surfaces. Sorting submissions by
hand doesn't scale, and asking an LLM to read every email costs real money
at scale. A judgment model (Jev/TypeSafe-class: calibrated yes/no, choice,
score questions at ~40–1,000x lower cost) sorts them at intake:

- **bug vs feature-request vs complaint vs question** (choice)
- **severity/priority** (score, 1–10)
- **which team/handler** (intent routing → engineering / bugs / docs / human)
- low confidence → **clarify-or-ask**, never silently guess
- bugs get a second-level split: **frontend / backend / model-adjacent / docs**
  (choice + score), so the right team picks it up

Reference architecture: the Simon Scrapes Jev walkthrough (yt-2dai1jvyd5m,
transcript archived) — pattern 4 (intent routing) + pattern 2 (per-action
confidence thresholds) + pattern 3 (score parts, weight in code).

## The modular contract (the part that makes it reusable)

**One direction of data flow, three pure pieces, zero project coupling:**

```
submission (text + metadata)
      │
      ▼
[1. intake]    normalize + enrich (project-specific adapters live HERE, in the host app)
      │
      ▼
[2. judge]     the Jev layer: fixed question set, calibrated answers
      │        ask: type? severity? subsystem? confidence on each
      ▼
[3. route]     handler resolution: per-action thresholds + weights in CODE
      │        (never in the model), clarify-or-ask below threshold
      ▼
verdict { category, severity, subsystem, route_to, confidence, needs_clarify? }
```

**Host apps provide:** a submission source (web form, email, API) and handlers
(tickets, teams, auto-replies). **This module provides:** the question set,
the judgment plumbing, the threshold policy, the routing decision, and the
audit trail. The host NEVER talks to a judgment model directly — it calls
`triage(submission) -> verdict`.

**Portability rules (enforced by design):**
1. No framework imports (FastAPI/React/ComfyUI stay OUTSIDE this repo)
2. Handlers are registered by the host, not hardcoded — `register_route(category, handler)`
3. The question set is DATA (versioned file), not code — new products add
   questions via config, not forks
4. Per-action thresholds are host-configurable with safe defaults
5. Every verdict carries provenance + confidence + the questions asked —
   an audit log by construction

## Non-goals (v1)
- No UI (hosts render their own forms; this module is the engine)
- No email polling/SMTP (a host adapter concern)
- No auto-close of anything destructive without explicit threshold + policy

## License
MIT. See LICENSE.