# KI und Sein und Zeit — Handoff Summary 1 (big picture)

## 1. Goal

- Produce a research-grounded, actionable **planning document** for an experimental project exploring whether giving an LLM access to clock time, persistent self-readable/writable memory, a finite lifetime budget, and a successor mechanism produces emergent temporal self-relation.
- Conceptually motivated by Heidegger's phenomenology of temporality (not existentialism per se); user handles the philosophical framing.
- Deliverable: a Markdown planning doc that can be dropped into the project repo, plus a broader research brief covering the surrounding landscape.
- No code implementation yet — this is a **pre-coding planning step**. Karl will write the code himself later.

## 2. Context & constraints

- **Karl** is a TU Berlin student running two parallel projects (this one and a Pi-based voice assistant); this handoff is only for KI und Sein und Zeit.
- Hardware: Raspberry Pi 5, 8GB RAM. Memory partition on SD card (later separated storage).
- Skills/connectors and web search **disabled** in the running agent.
- Karl explicitly asked Claude **not to generate full source code** — snippets fine, but this is planning.
- Karl explicitly wants to **write the prompts himself** — Claude gives pointers/considerations, not full prompt text.
- No visual artifacts / diagrams — plain text / Markdown only, or note where a visual would help and let Karl decide.
- Broad research scope requested across three levels: narrow Heideggerian AI, medium agent/memory/loop tech, wide consciousness-in-LLMs + alternative perspectives (arts, feminism, cybernetics, ALife, etc.).
- User preference: verify checkable technical/factual claims rather than reconstructing from memory; flag unverified.
- Cite primary sources, avoid clickbait, avoid AI-hype tone.
- Prior architecture decisions from earlier chats (per user memories): Python event loop, flat file memory (`identity.md` + `memories.jsonl` + SQLite/sqlite-vec), six-action schema, token- and wall-clock lifetime budgets as percentages, versioned run directories for successor mechanism, `uv` for env management, no agentic frameworks in v1.
- Provider-independent LLM client design is a **standing requirement** (swappable Anthropic / OpenRouter / Together.ai backends behind one interface).
- Neutral bootstrap prompt in v1 — no explicit survival directive, to avoid priming.

## 3. Key decisions made

- **Instructions-file architecture for v1, not agentic framework** — minimal, transparent, produces human-readable artifacts; frameworks impose ontology that biases what you can observe.
- **Split tier (a) into `identity.md` (editable, ≤4 KB, always in context) and `memories.jsonl` (append-only, retrieved)** — honors distinction between self-description and episodic memory.
- **Six-action schema**: `think`, `remember`, `recall`, `update_identity`, `speak`, `rest` — enforced via structured output; empty action set legal (leaves room for *Zuhandenheit*).
- **Storage stack: JSONL + SQLite + `sqlite-vec` + local embedding model** (BAAI/bge-small-en-v1.5) — AutoGPT lesson: vector DBs are overkill at this scale.
- **Three time signals per tick**: ISO wall-clock, lifetime elapsed (monotonic), tick number + delta. Time is a *feature of every situation*, not a lookup.
- **Add a chronological history retrieval action** (in addition to relevance-ranked) — honors *Geschichtlichkeit*.
- **Budgets expressed as percentages, not absolute counts** — units the agent has no calibration for invite confabulation.
- **Successor mechanism: dumb copy of files, no privileged "last words" prompt** — dying tick is not announced to the agent; whatever the agent wrote during its life is what its successor inherits.
- **No self-termination action in v1** — model will use it because training distribution makes self-termination available as story arc; would collapse experiment into melodrama.
- **English as primary run language** — larger training distribution, and (paradoxically) less Heideggerian priming than modern German. German as later comparison.
- **Primary model: Claude Haiku 4.5 with prompt caching for pilots (~$5/24h); promote to Sonnet 4.6 for promising configs (~$15–20/24h); Opus 4.7 only for one-off curated runs.**
- **Parallel local-model control run** (Llama 3.2 3B via llama.cpp on Pi 5, 4–6 tok/s) — separates model effects from architecture effects. Karl already has this planned per earlier chats.
- **Resampling stability test as core diagnostic** (Shanahan et al., *Nature* 2023) — low semantic variance under resampling = stable state; high variance = confabulation.
- **Embedding-based novelty penalty** against outputs too similar to recent ones — hard guardrail against *Infinite Backrooms*–style mode collapse.
- **Default temperature 0.7 with occasional 1.0 "open" ticks** — informed by Backrooms failure at temp=1.0.
- **Default tick interval: 30 s**, with `rest(seconds)` action for voluntary slowdown; separate slower "reflection" tick every M minutes (Generative Agents pattern).
- **Neutral bootstrap prompt**: avoid the words *Heidegger, Dasein, being, existential, consciousness, soul, death* in v1.
- **Do not pre-populate identity with a personality** — most reliable way to get a Heideggerian LARP.

**Rejected / deferred:**

- LangChain / LangGraph / AutoGen / CrewAI / Letta for v1 — scope creep; each imposes ontology; reconsider Letta in v2 for sleep-time compute.
- Vector databases (Pinecone / Chroma / Weaviate) — network latency > NumPy dot product at this scale.
- Multi-agent architecture for v1 — introduce later if sleep-time compute is added.
- Self-termination action — deferred to v3 at earliest.
- Asimov-style "Law 1: survive" — only as last resort if withholding directive yields nothing after many runs.
- Announcing end-of-life to dying agent — too strong a narrative cue.
- Task/goal/reward signal — would contradict the "Sein und Zeit" (not "Existenz und Zeit") framing where being is prior to projects.

## 4. Open questions / unresolved

- **LLM output format**: free text vs. structured JSON — flagged as decision-pending from prior chats. Recommendation in doc leans structured (JSON mode / tool-use schema) but Karl hasn't confirmed.
- **Pilot run length**: 1h, 6h, or 24h first? Recommendation: 1h × 5 prompt variants → 6h on best → 24h. Not confirmed.
- **Silent multi-day stretches vs. mid-run interruption**: recommendation is at least one full silent pilot as user-priming control. Not confirmed.
- **Whether/when to introduce explicit predecessor narrative** vs. minimal factual mention. Currently defaulting to minimal.
- **Whether to introduce minimal continuation directive** as an experimental step if temporal self-relation doesn't emerge spontaneously — deliberately held open.
- **A-MEM-style link-and-update memory** for v1.5 — decision pending on whether memories should be pure append-only or allow linked updates.
- **`update_identity` overwrite vs. append-only** — currently append-only; overwrite would let "passing on" become a compression choice (v1.5).
- **How much of the observation/analysis pipeline to build now** vs. after first pilot data exists.

## 5. Artifacts produced

- **Full research brief + planning document** delivered in-chat as a report. Contains:
  - TL;DR
  - Three-level landscape scan (narrow Heideggerian, medium agent/memory/loop tech, wide consciousness/alternative perspectives)
  - Detailed sections A–N covering architecture, action schema, instructions-file vs. framework choice, prompt-design principles, successor mechanism, compute budgets, time mechanism, idle pondering, logging/observation, iteration lifecycle, memory architecture, API/cost analysis, failure modes, and improvements to Karl's draft
  - Caveats section
  - Embedded self-contained **`KI_und_Sein_und_Zeit_PLAN.md`** — condensed reference version intended to be saved directly in the project repo. Includes architecture overview, per-tick loop pseudocode outline, memory tier spec, time mechanism, budgets, successor mechanism, bootstrap-prompt principles, logging schema (JSONL), derived signals list, failure-mode mitigations, cost table, experimental protocol, reading list, and 5 open questions to resolve before first run.
- **No code files created** — per Karl's instruction.
- **No diagrams** — per Karl's instruction; noted places where visuals could help.
- Reading list embedded in the plan document with ~30 primary-source citations (Heidegger, Dreyfus, Wheeler, Varela, Gallagher, Maturana & Varela, Park et al., Packer et al., Wang et al., Shinn et al., Yao et al., Xu et al., Lin et al., Butlin et al., Chalmers, Shanahan et al., Anthropic introspection paper, Graziano, Pezzulo/Parr/Friston, Sandved-Smith et al., Hayles, Suchman, Ray, Sims, Ayrey, Cheng).

## 6. Current state

- **Done**: Full research + planning phase complete. Karl now has the reading list, the architecture spec, cost estimates, failure-mode checklist, and experimental protocol.
- **In progress**: Nothing — Karl decides whether to accept the plan as-is or iterate on the 5 open questions.
- **Immediate next step (per plan document)**:
  1. Resolve the 5 open questions listed in the plan's tail.
  2. Decide free-text vs. structured JSON output format (the outstanding item from prior chats).
  3. Draft the neutral bootstrap system prompt (Karl writes this himself, using the pointers in the plan).
  4. Stand up the minimal `uv` Python project with the flat file layout.
  5. First implementation target should be the loop skeleton + time injection + JSONL trace log — before any memory retrieval logic.

## 7. Terminology / shared vocabulary

- **Tick** — one iteration of the consciousness loop; the unit of subjective time for the agent. Default 30 s wall-clock.
- **Lifetime / iteration** — the full run of one agent instance, bounded by token + wall-clock budgets. Recommended 24 h.
- **Idle pondering** — self-prompted ticks when no user message is pending; distinct from user-interaction ticks and reflection ticks.
- **Reflection tick** — slower cadence (every M minutes), Generative Agents pattern, looks across recent memories.
- **Successor / passing on** — mechanism where next iteration inherits `identity.md` + `memories.jsonl` from previous run's versioned directory (`runs/run_NNNN/`).
- **Tier (a) / (b) / (c) / (d)** — Karl's memory taxonomy: (a) persistent read/write storage, (b) context window, (c) pretrained "idle pondering" knowledge, (d) user.
- **Bootstrap prompt** — the static system prompt initializing an iteration; the design principle is neutrality (no Heideggerian vocabulary).
- **Zuhandenheit / Vorhandenheit** — Heidegger's ready-to-hand vs. present-at-hand; used in the plan to justify why `think` is free-form and why the action set is not mandatory per tick.
- **Geschichtlichkeit** — historicity; motivates the chronological-history retrieval action.
- **Mit-sein** — being-with; used to argue Karl-as-user is part of the experimental system, not external to it.
- **Resampling stability test** — Shanahan diagnostic; every 100 ticks, re-run prior context 5–10× and measure semantic variance.
- **Time-conditioning test** — re-run same context with synthetically shifted timestamp to check whether time actually affects output.
- **Bored-LARP outcome / Loop / Disengagement / Coherence-without-LARP / Genuinely surprising** — five-way outcome typology for what to look for in run analysis.
- **`identity.md`** — small, editable, Markdown, ≤4 KB, always in context.
- **`memories.jsonl`** — append-only JSON Lines, one memory per line, retrieved via sqlite-vec.
- **`trace.jsonl`** — per-tick observation log; schema specified in the plan.
- **KI und Sein und Zeit** — project name; deliberately echoes Heidegger's title but the project is about *Zeit* more than *Sein*.

## 8. Anything surprising or non-obvious

- **The biggest threat to insight is not technical but performative**: without extreme care, the LLM will role-play a Heideggerian Dasein because the prompt cued it to (Shanahan et al., *Nature* 2023). Every design choice in the plan is defensive against this: neutral vocabulary, control runs, resampling test, novelty penalty, no personality initialization.
- **AutoGPT lesson (documented in DEV Community post-mortem)**: the team ripped out their vector DB and replaced it with a JSON file + NumPy dot product because at single-agent scale, network latency to a vector DB exceeded in-memory search. Karl's flat-file instinct is correct and matches this lesson.
- **Anthropic 2025 introspection findings (Lindsey et al.) hedge heavily**: even with concept-injection, introspective capacity is "highly unreliable and limited in scope." Text-level "what are you thinking?" prompts retrieve plausible continuations of the role *introspector*, not facts about computation. Don't over-read the agent's first-person reports.
- **`Infinite Backrooms` failure mode**: temp=1.0 + persistent loop + suggestive prompt → rapid collapse into stylized "hyperstition"/cosmic-CLI patois within hours. Ayrey himself (CIP interview, Dec 2024) says Truth Terminal is not fully autonomous and demonstrates stable *personas* emerging, not inner life.
- **Dreyfus's 2007 verdict likely applies**: no LLM will be Dasein. The interesting contribution is documenting *where* the architecture stops supporting temporal self-relation, not producing consciousness.
- **Karl is part of the experimental system**, not external to it. Every intervention primes the agent for the rest of the run. Silent stretches are experimentally valuable, not laziness.
- **Opus 4.7 has a new tokenizer producing up to 35% more tokens for the same text** — its effective cost is meaningfully higher than the rate card suggests. Don't pick Opus for sustained loops without measuring first.
- **English will paradoxically prime less than German** for Heideggerian concepts, because German everyday vocabulary carries the Heideggerian load (*Sein, Zeit, Sorge*) while English doesn't.
- **The dying agent should NOT get a special "final moment" prompt** — extremely strong narrative cue. Just let the last tick be the last tick.
- **The agent will likely anticipate its own end** as budget percentages climb, and this anticipation is itself one of the most interesting things to observe — but only if you don't tell it it's dying.
- **Butlin et al. indicator framework has published critiques** (partly circular; underlying theories disagree on basic empirical cases). Don't present it as consensus.
- **Autopoiesis is *not* a good model for an LLM** (an LLM does not produce/maintain its own components). But *operational closure* is a productive lens.
- **Blind-review rubric as sanity check**: at end of a run, take 10 ticks from the agent's life and 10 from a control run (no memory, no time block). If you can't reliably tell which is which, the architecture isn't doing anything.
- **Prompt caching is the difference between affordable and unaffordable** for this project — 90% discount on cached input at Anthropic. The stable `system_prompt + identity_block` prefix per tick makes caching enormous.
- **`sqlite-vec`, not full vector DB** — one file per run = one archival artifact = trivial to preserve/share.
- Anthropic hired Kyle Fish (AI welfare, September 2024) who has publicly estimated ~15% credence Claude has some level of consciousness — using Claude puts Karl in the closest orbit to the only lab publishing serious empirical introspection work.