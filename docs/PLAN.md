# jev-triage — Implementation Plan

Status: SKELETON phase (2026-09-29). This plan defines the milestones Gui
approved. Size estimate: 2–3 milestones (small feature, built modularly).

## Milestone JT-1 — Core engine (in-repo, Python, stdlib-first)
- `questions.json` — versioned question set (type choice, severity score,
  subsystem choice, per-question confidence) as DATA with a schema
- `judge.py` — judgment plumbing: submit questions to a Jev-compatible
  endpoint (abstract provider: local Tierllama proxy, cloud Jev, or a stub
  for tests), return calibrated answers
- `route.py` — per-action threshold policy + handler registry
  (`register_route`) + clarify-or-ask below threshold
- `triage.py` — the public one-call API: `triage(submission) -> verdict`
- audit trail: every verdict logs questions asked + answers + confidence
- tests: golden submission set (v1, ~20 cases with truth labels), full
  sandbox discipline (no live network — stub provider in tests)

## Milestone JT-2 — First consumer integration (comfyui-video-ui support tab)
- Host adapter: FastAPI endpoint in the video-UI backend calling triage()
- Support intake form (React) → triage → route to: auto-ack bug / feature
  log / docs question reply / human
- Dashboard view for the team: triaged queue with confidence + audit
- Feedback hook: user corrections of mis-triage = labeled data (feeds JT-3)

## Milestone JT-3 — Reusable packaging + export
- Cut the module free of any host assumptions; publish to its own repo
  (this repo) as a pip-installable package
- tierllama adoption: proxy error reports + support intake route through
  the same engine (export path proven)
- Optional: question-set variants per product (config, not code)

## Design guardrails (inherited from Tierllama doctrine)
- Clarify-or-ask is absolute — never silently guess a category
- Weights and thresholds live in CODE (pattern 3), questions in DATA
- Beta features OFF by default; consent for anything that sends user
  content to a cloud judgment endpoint
- One module, own tests, own commits; never silent behavior changes