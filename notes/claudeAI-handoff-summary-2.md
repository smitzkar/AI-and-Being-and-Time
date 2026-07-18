# KI und Sein und Zeit — Handoff Summary 2 (minimal implementation)

## 1. Goal

Decide the technical foundation for a minimal v0.1 implementation of the "KI und Sein und Zeit" experiment: an event-loop-driven LLM agent with clock-time access, persistent memory, finite lifetime budget, and successor mechanism. Specifically resolve two blockers before writing code: (a) whether raw API calls are experimentally clean, and (b) how to handle compute given very limited local hardware.

## 2. Context & constraints

- Karl writes the code himself; Claude is guide/collaborator, not code generator.
- v0.1 must produce quick, visible results with minimal moving parts to isolate concept-level insights from infrastructure noise.
- TU Berlin HPC ruled out: policy requires chair/thesis affiliation (see https://hpc.tu-berlin.de/doku.php?id=hpc:scheduling:access). Deferred to post-v0.1 once project has formal home.
- Local hardware inventory is all inadequate for hosting a meaningfully capable model:
  - Lenovo Y520, twin GT 750M/755M (Kepler, ~2GB VRAM each)
  - Acer Spin 3, integrated graphics
  - Samsung NC10, Pi 5 8GB, Pi Zero 2, ESP32-S3, OpenMV H7
- ADHD-relevant constraints: flat file structures, minimal cognitive overhead, transparent + well-documented automation, portable across machines/OS, feedback when automation runs, visual overview without deep nesting.
- English as primary run language (German later as comparative condition — carried over from prior chat, not re-decided here).

## 3. Key decisions made

- **Stay with API for v0.1** — hidden-prompt concern resolved; Haiku 4.5 as primary model.
- **Provider-independent LLM client from day one** — swappable backends behind one interface (Anthropic, OpenRouter/Together, placeholder). Karl flagged as easy to forget mid-implementation; saved to persistent memory.
- **Cross-provider comparison as the confound mitigation** — rather than seeking a "clean" model, run same loop against Anthropic + OpenAI + open-source (via OpenRouter/Together) and see what survives the swap. Addresses RLHF-baked character confound better than local hosting could.
- **`uv` for environment management** — over Docker (overkill, high cognitive tax for tiny dep footprint) and manual venv (forget-to-activate friction). Single `pyproject.toml` + `uv.lock`, `uv run script.py` handles env automatically, portable across machines, editors auto-detect the `.venv`.
- **Docker deferred until deployment-as-service on the Pi** — that's when a container solves a real problem.
- **No `tools=[...]` parameter in API calls** — avoids Anthropic's auto-constructed tool-use system prompt (the only known injection on the raw API).
- **Flat project layout** — five Python files + one `data/` folder + `README.md`; no `src/` nesting, no premature modularization.
- **Time control lives in the script, not the model** — model does not have a "check the clock" tool; script injects perceived time into prompts. Commits the experiment to studying model's relation to a *provided* sense of time, not "real" temporality. Enables acceleration, pausing, deterministic replay.
- **Split identity (small, always-loaded) from memory log (append-only JSONL) from the start** — avoids refactor when the two start being treated differently.
- **Placeholder LLM backend as first integration target** — debug loop logic without cost/latency; same interface as real backends.
- **OpenRouter or Together.ai over raw cloud GPU rentals (RunPod, Vast, Lambda)** for open-source comparisons — cheaper and frictionless for off-the-shelf models. Raw rentals reserved for unusual/fine-tuned hosting needs.

## 4. Open questions / unresolved

- **LLM output format** — free text, text + fenced JSON action block, or full structured JSON. Leaning toward text-thought + small JSON action block, not decided. Shapes parsing, prompt design, and expression cage. Must be settled before writing the loop.
- **Exact system prompt / bootstrap framing** — the most experimentally-loaded variable in the system. Only sketched conceptually in the prior chat, not drafted.
- **Whether to write a prose paragraph describing the loop before coding** — suggested as a cheap forcing function; Karl has not committed to doing it.
- **Acer Spin 3 as free loop sanity-checker with llama.cpp + 1B GGUF** — mentioned as an option, not decided.
- **When/how to introduce German-language run** — carried over from prior chat.
- **Whether to add real-timestamp mode as a later flag** — flagged as always-addable, no decision.

## 5. Artifacts produced

No code written this chat. Deliverables were decisions and one recommended layout:

- **Recommended v0.1 file structure** (described, not created):
  ```
  ki-sein-und-zeit/
  ├── pyproject.toml
  ├── uv.lock
  ├── .env                # gitignored
  ├── README.md
  ├── main.py             # loop
  ├── llm.py              # provider-independent client
  ├── memory.py           # read/write memory files
  ├── data/
  │   ├── identity.md
  │   ├── memories.jsonl
  │   └── trace.jsonl
  └── .gitignore
  ```
- **README.md structure** (recommended): three sections — "how to run", "what's where", "what I decided and why" (last one as future-self debugger).
- **Persistent memory entry saved**: reminder about provider-independent client design.

## 6. Current state

- **Done:** API-cleanliness question resolved; hardware path decided (API-first, Acer as optional sanity-check); environment management decided (`uv`); file layout drafted; provider-independence reminder persisted.
- **In progress:** none — pre-implementation phase.
- **Immediate next step:** produce three small pre-code artifacts before writing any Python:
  1. One-paragraph prose description of the loop in the README.
  2. Decision on LLM output format, with a 2-line example in the README.
  3. Rough draft of the system prompt / bootstrap framing.
- After those three exist, coding becomes transcription. Start order likely: `pyproject.toml` via `uv init` → placeholder `llm.py` backend → skeleton `main.py` loop → `memory.py` → swap in real backend.

## 7. Terminology / shared vocabulary

- **KI und Sein und Zeit** — project name; Heidegger-inspired experiment on temporal self-relation in LLMs.
- **Tick** — one iteration of the event loop; one LLM call + action parsing + memory update + sleep.
- **Lifetime budget** — finite token count and wall-clock time per iteration, expressed to the agent as percentages, never absolute counts.
- **Successor mechanism** — at end-of-life, current run's identity + memories seed the next run's directory.
- **Identity** vs. **memories** — small always-loaded self-description file vs. append-only event log.
- **Placeholder backend** — a non-LLM function returning canned/random output, used behind the LLM-client interface for loop debugging.
- **Model time vs. wall-clock time** — the script controls what "time" the model perceives, decoupled from actual elapsed real time.
- **Trained character confound** — RLHF/Constitutional-AI-baked assistant persona that persists regardless of system prompt; the reason cross-provider comparison matters.

## 8. Anything surprising or non-obvious

- **Raw Anthropic/OpenAI APIs do not inject the chat-product system prompts.** Anthropic states this explicitly in docs. The only known narrow exception on Anthropic: passing `tools=[...]` triggers an auto-constructed tool-use system prompt. Don't pass `tools` and the model sees exactly what you sent.
- **The real confound is not hidden prompts but baked-in trained character.** Not fixable by prompt engineering; addressable only by cross-provider comparison or by running a base/pretrained checkpoint (which no API exposes — a future reason to want HPC).
- **Local hosting on Karl's hardware is a trap for this experiment specifically.** Not because it can't run — Pi 5 does 4–6 tok/s on Llama 3.2 3B Q4 — but because small models produce qualitatively thinner output, introducing the ambiguity "is nothing interesting happening because the architecture is wrong, or because the model is too weak?" — exactly what v0.1 needs to avoid.
- **OpenRouter/Together beats raw GPU rentals for the open-source comparison run.** Karl might reflexively reach for RunPod/Vast because "self-hosted = more control"; the actual right answer for off-the-shelf models is the aggregator API — capable models, no infra, unified interface.
- **HPC hesitation ("should be for good use") is largely self-imposed** — university clusters are for student exploration and learning SLURM on a "silly" first job is legitimate use. Still deferred here only because policy blocks access without chair affiliation, not because of the self-imposed concern.
- **`uv run` failing loudly when misused is a feature, not a bug** — silent wrong-env behavior (classic venv failure mode) is worse than an explicit error, especially with ADHD.
- **Time-control decision commits the experiment philosophically.** It's studying how the model relates to a provided sense of time, not "real" temporality. Worth stating in the README so future-Karl doesn't second-guess it or drift into adding real timestamps without noticing the framing shift.
- **The output-format decision is upstream of almost every other implementation choice** — parsing, prompt design, action schema, logging format all depend on it. Deferring it means rewriting things.