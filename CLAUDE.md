# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

**Pre-implementation.** No source code yet — the repo currently holds only the README and handoff notes from prior claude.ai chats. There is nothing to build, lint, or test. The immediate work is the last few planning artifacts before the first `uv init`.

## Scope

Bachelor's-thesis proof of concept, not a production system. Keep the implementation Arduino-tutorial-simple: flat layout, small files, hide complexity (sockets, threading, etc.) behind small helpers like `send_update()`. Stubs and dummies are acceptable placeholders for things like tests, robust error handling, encryption, or elaborate logging — mention them as future work rather than building them out.

## Working mode

Karl writes the code himself and uses Claude as an adviser. Do **not** produce full implementations unprompted — offer pointers, sketches, and trade-offs; ask clarifying questions when he seems undecided; challenge weak choices rather than agreeing. When he starts down a rabbit hole (framework selection, over-engineering, premature optimization), remind him of thesis scope. See `/mnt/c/Users/smitz/.claude/rules/custom-instructions.instructions.md` for the full instructions.

## Prior context

The four files in `notes/` (`claudeAI-handoff-projectmemory.md` and `claudeAI-handoff-summary-{1,2,3}.md`) are structured handoffs from three prior claude.ai chats. They are the source of truth for decisions already made — read the relevant one before re-litigating an architectural choice. Summary:

- **Handoff 1** — big-picture research + planning for KI und Sein und Zeit; reading list; open questions.
- **Handoff 2** — minimal-implementation decisions (API-first, `uv`, file layout, provider-independent LLM client).
- **Handoff 3** — desktop voice assistant hardware bring-up (Xiaozhi, INMP441 + PCM5102 I²S wiring, config.txt fix).
- **projectmemory** — cross-project state and principles.

## Two projects in this repo

### 1. KI und Sein und Zeit (thesis focus)

Experimental proof of concept exploring whether giving an LLM (a) clock-time access, (b) persistent self-readable/writable memory, (c) a finite lifetime budget expressed as percentages, and (d) a successor-passing mechanism produces emergent temporal self-relation. Heidegger-motivated; Karl handles the philosophy.

Intended v0.1 architecture (settled in prior chats):

- **Flat layout**, ~5 Python files at repo root: `main.py` (loop), `llm.py` (provider-independent client), `memory.py`, plus `data/{identity.md, memories.jsonl, trace.jsonl}`. No `src/` nesting.
- **`uv`** for env management (single `pyproject.toml`, no Docker in v0.1).
- **Provider-independent LLM client from day one** — swappable Anthropic / OpenRouter / Together / placeholder backends behind one interface. Easy to forget mid-implementation.
- **Placeholder backend first** — debug loop logic before spending on real API calls.
- **API-first**, Claude Haiku 4.5 as primary model with prompt caching. Do **not** pass `tools=[...]` to the Anthropic API (avoids the auto-injected tool-use system prompt).
- **Time is injected by the script**, not fetched by the model. The agent has no "check the clock" tool; per-tick prompt carries ISO wall-clock, lifetime elapsed, tick number/delta.
- **Six-action schema**: `think`, `remember`, `recall`, `update_identity`, `speak`, `rest`. Empty action set is legal.
- **Memory split**: `identity.md` (≤4 KB, always in context, editable) vs. `memories.jsonl` (append-only, retrieved via `sqlite-vec` + local BGE embeddings). Add successor mechanism as a dumb file copy into `runs/run_NNNN/`.
- **Budgets as percentages**, never absolute counts. Default tick 30 s.
- **Neutral bootstrap prompt** — avoid the words *Heidegger, Dasein, being, existential, consciousness, soul, death* in v1. Don't pre-populate identity with a personality. English is the primary run language.

Three artifacts owed before writing Python (in this order):
1. One-paragraph prose description of the loop in README.
2. **LLM output-format decision** (free text vs. text + fenced JSON action block vs. full structured JSON). This is the blocker — parsing, prompt design, action schema, and logging all depend on it.
3. Rough draft of the bootstrap system prompt (Karl writes himself).

### 2. Desktop voice assistant (secondary / warm-up)

Xiaozhi-based (`xinnan-tech/xiaozhi-esp32-server` on Pi 5, ESP32-S3 thin client). Not yet in this repo as code. Immediate step is physical: INMP441 mic bring-up on the Pi 5's I²S bus alongside the already-working PCM5102 DAC. Wiring table, `config.txt` block, and verification commands are in `notes/claudeAI-handoff-summary-3.md`.

Treat this as secondary — don't let it eat time meant for the thesis project.
