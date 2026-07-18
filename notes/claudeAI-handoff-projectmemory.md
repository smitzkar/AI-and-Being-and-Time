Purpose & context
Karl is a student at TU Berlin working on two distinct projects:

1. "KI und Sein und Zeit" — An experimental research project exploring whether endowing an LLM with clock-time access, persistent self-readable/writable memory, a finite lifetime budget, and a successor-passing mechanism produces emergent temporal self-relation. Conceptually motivated by Heidegger's phenomenology of temporality, though Karl handles the philosophical framing independently. Goals: minimal proof-of-concept first, API-based, no agentic frameworks in v1, English as primary run language (German as later comparative condition), silent multi-day run stretches as control on user-priming effects.
2. Desktop Voice Assistant / "Status Update Device" — A hardware/software voice assistant inspired by the Talking Treebot exhibit (github.com/technologiestiftung/talking-treebot-exhibit), to which Karl previously contributed. Goals include task reminders, status-device function calls, and an internal agency loop.

*Hardware owned:* Raspberry Pi 5 8GB (primary compute), Pi Zero 2, various ESP32 and nRF devboards, INMP441 microphone, PCM5102 DAC (confirmed working).

*IMPORTANT NOTE:*  
Karl has ADHD and is easily overwhelmed by crowded file structures — prefers flat project layouts, visual overviews, and automation that is transparent, well-documented, and portable across machines/OS.

*Current state*
KI und Sein und Zeit:

- Pre-implementation phase; no code written yet (Karl explicitly wants to write it themselves, with Claude as collaborator/guide rather than code generator)
- TU Berlin HPC cluster ruled out (requires chair affiliation); staying with API access for v1
- Architecture decisions made: Python event loop with time-perception control; flat file memory (identity.md + memories.jsonl + SQLite/sqlite-vec); six-action schema (think, remember, recall, update_identity, speak, rest); token- and wall-clock lifetime budgets as percentages; versioned run directories for successor mechanism
- Three pre-coding artifacts identified as next step: prose loop description, chosen LLM output format (free text vs. structured JSON — decision pending), and rough system prompt draft
- Bootstrap prompt to be kept neutral in v1 (no explicit survival directive) to avoid priming
- uv chosen for environment management over Docker or manual venv

Voice Assistant:

- INMP441 mic bring-up on Pi 5 via I²S is the immediate practical next step
- Cleaned-up config.txt provided (using dtoverlay=googlevoicehat-soundcard only); GPIO18 conflict with dtoverlay=waveshare35a LCD overlay flagged as a debug risk — recommendation to comment it out during audio bring-up
- Seeed reSpeaker Lite (~€34) + Xiaozhi self-hosted server (xinnan-tech/xiaozhi-esp32-server on Pi 5) identified as primary recommended path


*On the horizon*
KI und Sein und Zeit:

- LLM output format decision (free text vs. structured JSON) must be made before writing the event loop
- Cheap open-source model comparison run via OpenRouter or Together.ai planned as parallel control
- Local model comparison (Llama 3.2 3B via llama.cpp) recommended as control run
- German-language run as later experimental condition
- Potential introduction of minimal explicit continuation directive as a later experimental step if temporal self-relation doesn't emerge spontaneously

Voice Assistant:

- Once INMP441 bring-up confirmed, proceed toward Xiaozhi server deployment on Pi 5 via Docker
- Home Assistant Voice Preview Edition (~€60) and LilyGO T-Circle-S3 noted as alternative hardware paths


*Key learnings & principles*

- Provider-independent LLM client is critical for "KI und Sein und Zeit": design the client interface to support Anthropic, OpenRouter/Together.ai, etc. via swappable backends behind one interface from the start — Karl flagged this as easy to forget mid-implementation.
- Raw API calls to Anthropic/OpenAI don't inject hidden system prompts (narrow exception: tool definitions); RLHF-baked model character is a more fundamental confound, best addressed via cross-provider comparison rather than seeking a "clean" model.
- Xiaozhi is architecturally equivalent to the Treebot (thin ESP32-S3 audio client + server-side LLM brain), supports OpenAI-compatible interfaces, function calling, and MCP tool integration — directly maps to Karl's goals.
- I²S shared clock bus: BCLK and LRCLK shared between INMP441 and PCM5102; Pi as clock master, both peripherals as slaves; separate unidirectional SD lines. Common failure modes: floating L/R pin and BCLK/WS swap.
- Flat five-file project structures preferred to avoid cognitive overhead from deep nesting.


*Approach & patterns*

- Prefers to write code himself with Claude as guide; explicitly instructs Claude not to generate full implementations unprompted
- Values automation that is transparent and portable (favors uv over Docker for early-stage Python projects)
- Iterative, tiered testing approach: placeholder backends before real LLMs
- Treats own interventions as explicit experimental conditions rather than noise
- Produces Markdown deliverables suitable for dropping directly into project repos


*Tools & resources*

- KI und Sein und Zeit: Anthropic API (Claude Haiku for cost efficiency), OpenRouter or Together.ai for open-source model comparison, uv for environment management, Python, sqlite-vec, llama.cpp (planned)
- Voice Assistant: Raspberry Pi 5 8GB, Xiaozhi ESP32 server (Docker), Seeed reSpeaker Lite (primary candidate), INMP441 + PCM5102 on I²S
- Reference projects: github.com/technologiestiftung/talking-treebot-exhibit, xinnan-tech/xiaozhi-esp32-server, Generative Agents, MemGPT/Letta, Andy Ayrey's Infinite Backrooms