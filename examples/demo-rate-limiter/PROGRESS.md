# Distributed Rate Limiter (demo) — Progress

## Goal
Demo walkthrough of the Build to Learn skill — backend/systems domain, to show the mechanics work outside AI/ML projects too.

## Why this idea
Chosen to prove the skill's build loop and both companion docs work for a different kind of project than a RAG/ML pipeline — same mechanics, different domain.

## Phase roadmap
- [x] Phase 1 — Environment & tooling setup
- [x] Phase 2 — Core architecture decisions (algorithm + distributed state)
- [ ] Phase 3 — Component build: the rate-limit middleware
- [ ] Phase 4 — Evaluation: load testing the limiter under burst traffic
- [ ] Phase 5 — Hardening: Redis failover behavior
- [ ] Phase 6 — Polish & packaging
- [ ] Phase 7 — Interview-prep consolidation

## Friction log
- Phase 2 — First pass used separate GET+SET Redis calls for the token check, which created a race condition under concurrent requests; fixed by moving the check-and-decrement into a single atomic Lua script.

## Companion docs
- `concepts.html` — 2 entries
- `interview_qa.html` — 3 questions
