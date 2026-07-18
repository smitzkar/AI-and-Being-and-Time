# KI und Sein und Zeit — Handoff Summary 3 (desktop-companion)

## 1. Goal
- Build a customisable "desktop friend": a voice device to chat with (treebot-style), that can also
  - remind the user of a list of tasks, and
  - call the functions of the user's separate "status update device".
- Longer-term: give it an internal agency loop so it feels more "real".
- This chat's concrete objective narrowed to: **get the INMP441 mic + PCM5102 DAC working together over I²S on the Raspberry Pi 5** as a first hardware test, having chosen Xiaozhi as the intended software architecture.

## 2. Context & constraints
- Detour/warm-up project after a 4-week gap; deliberately picking the treebot as a starting point because it "already exists".
- Fits into the user's broader interest in expanding a "status update device".
- Budget for any new hardware: **< €100**, factoring in already-owned gear.
- Already owned: Raspberry Pi 5 8GB (primary compute), Pi Zero 2, various ESP32 and nRF devboards, INMP441 mic, PCM5102 DAC (confirmed working).
- Must be **fully customisable** — ready-made closed ecosystems (Amazon Alexa etc.) explicitly ruled out.
- Bonus wants: rotary knob, other interesting input options, and/or a screen.
- No heavy on-device compute required — the compute lives on a server (the Pi 5), device is a thin client.
- Prior treebot work by the user: minor cross-device comms, power handling, connectivity troubleshooting; had also started (unfinished) a migration to OpenAI's Realtime API, threading/task fixes, and a mic upgrade (replace Bluetooth mic with wired to cut latency; possibly a mic array / beamforming).
- Treebot repo: github.com/technologiestiftung/talking-treebot-exhibit (Python ~90%, C++ ~8%, Shell). Original ran on a Pi 4 4GB; readme warns multithreading + audio playback is too heavy for a Pi Zero.

## 3. Key decisions made
- **Software architecture = Xiaozhi** (ESP32-S3 thin client + self-hosted Python server). Reason: it *is* the evolved treebot architecture (thin audio client + server-side LLM brain), MIT-licensed, large community, OpenAI-compatible LLM backend, Function Call + MCP support, local memory. Directly maps to all three goals.
- **First physical test = INMP441 + PCM5102 on the Pi 5's I²S bus**, before touching ESP32/Xiaozhi. Reason: de-risk the audio layer on known hardware first.
- **Use the `googlevoicehat-soundcard` overlay** as the single overlay describing both I²S input (mic) and output (DAC). Reason: one Pi I²S interface + one soundcard overlay must describe both directions; the voiceHAT overlay is the bidirectional card that matches this topology, and is the path of least resistance.
- **INMP441 L/R pin tied to GND** → drives the left channel slot. Reason: pin must be committed (floating = garbage); left is the conventional choice.
- **Record as S32_LE**, not S16_LE. Reason: INMP441 outputs 24-bit data left-justified in a 32-bit slot.
- Rejected: **`hifiberry-dac`** overlay — output-only, can't also provide mic input.
- Rejected: **stacking `hifiberry-dac` + another overlay** for input — a single I²S card must be described by one overlay.
- Rejected (for now): **`simple-audio-card`** — kept only as a manual fallback if googlevoicehat gives silence.
- Rejected: **Amazon Alexa / closed assistants** — not customisable.
- Considered but deprioritised vs. Xiaozhi: **Home Assistant Voice PE** (~€60; has the rotary knob, button, LED ring, dual mic + XMOS, Grove port) — good off-the-shelf option and natural function-dispatch hub, but heavier HA dependency and awkward for a custom internal loop. Buy only if the user decides to lean on HA as the dispatch layer.
- Porting treebot Python → MicroPython on ESP32 judged **unnecessary** if going the Xiaozhi route — Xiaozhi replaces the firmware layer; the unfinished Realtime-API migration and threading fixes become "configure the backend" instead of "rewrite firmware".

## 4. Open questions / unresolved
- Whether `googlevoicehat-soundcard` will play back cleanly without the amp-enable GPIO the original voiceHAT expected. If card enumerates but playback is silent, that GPIO is the suspect → fall back to `simple-audio-card`.
- **Waveshare 3.5" LCD (`waveshare35a`) vs I²S pin clash**: the LCD overlay may claim GPIO18 (shared BCLK). Not yet verified. Recommendation: check `/boot/firmware/overlays/README` / `dtoverlay -h waveshare35a`, and comment the LCD overlay out for first audio bring-up to remove the variable (testing is headless over VNC anyway).
- Exact card/device numbers for `arecord`/`aplay` — to be read off `arecord -l` after reboot.
- Whether the user wants the `simple-audio-card` fallback config and/or a Python `sounddevice` capture-level sanity snippet (offered, not yet produced).
- Which final device form-factor: bare ESP32-S3 + I²S parts, reSpeaker Lite kit, or T-Circle-S3 (screen). Not committed.
- Server-side design of the internal loop / task reminders / status-device function-calls around the Xiaozhi backend (offered as next step, not started).

## 5. Artifacts produced
- **This handoff document** — `desktop-friend-handoff.md`.
- **Cleaned-up config.txt audio section** (not a full file, a replacement block):
  ```
  dtparam=i2s=on
  dtparam=audio=off

  # Single overlay providing BOTH I2S input (INMP441) and output (PCM5102):
  dtoverlay=googlevoicehat-soundcard

  # Make sure these are NOT active — they conflict / are output-only:
  #dtoverlay=hifiberry-dac
  #dtoverlay=simple-audio-card
  ```
- **Verification commands** (post-reboot):
  ```bash
  aplay -l        # expect sndrpigooglevoi card for output
  arecord -l      # expect same card as a capture device

  arecord -D plughw:CARD=sndrpigooglevoi,DEV=0 -f S32_LE -r 48000 -c 2 -d 5 test.wav
  aplay   -D plughw:CARD=sndrpigooglevoi,DEV=0 test.wav
  ```
- **Wiring table** (see §7 / below) — INMP441 and PCM5102 to Pi 5 40-pin header.
- No code committed to any repo yet; all inline in this chat.

### Wiring reference
Pi 5 fixed I²S pins:
- GPIO18 → I²S CLK (BCLK) — header pin 12
- GPIO19 → LRCLK (WS/FS) — header pin 35
- GPIO20 → DIN (data *into* Pi = from mic) — header pin 38
- GPIO21 → DOUT (data *out* of Pi = to DAC) — header pin 40

INMP441:
- VDD → 3.3V (pin 1 or 17) — **3.3V only, not 5V**
- GND → GND
- SCK → GPIO18 (pin 12)  [shared with DAC BCK]
- WS  → GPIO19 (pin 35)  [shared with DAC LRCK]
- SD  → GPIO20 (pin 38)
- L/R → GND (left channel)

PCM5102 (already working):
- BCK → GPIO18 (pin 12)  [shared]
- LRCK → GPIO19 (pin 35) [shared]
- DIN → GPIO21 (pin 40)
- SCK (board's master-clock input pin) → GND (use internal PLL; usually already jumpered)
- VIN → 3.3V/5V per board, GND → GND

## 6. Current state
- **Done:** architecture chosen (Xiaozhi); PCM5102 DAC confirmed working; I²S physical-layer model explained; wiring determined; config.txt fix identified; verification plan written.
- **In progress / immediate next step:** user to wire the INMP441 per the table, apply the cleaned config.txt block, (ideally) comment out `waveshare35a` for the test, reboot, then run `arecord -l` / record-and-playback to confirm capture.
- **After that:** move to Xiaozhi — deploy `xinnan-tech/xiaozhi-esp32-server` (Docker) on the Pi 5, choose device hardware, wire in function-calling for the status device + task reminders, then the internal loop.

## 7. Terminology / shared vocabulary
- **I²S / I2S** — Inter-IC Sound; synchronous serial audio protocol, clock on its own wire.
- **BCLK** (a.k.a. SCK/SCLK) — bit clock, one pulse per bit (~3.072 MHz at 2ch×32bit×48kHz).
- **LRCLK** (a.k.a. WS / word select / FS / LRCK) — frame clock, toggles once per sample at the sample rate; level selects channel (low=left, high=right in Philips I²S).
- **SD** — serial data; **one direction per wire**. Mic's data = source → Pi (DIN/GPIO20). DAC's data = Pi → sink (DOUT/GPIO21).
- **MCLK** — optional high-freq master/reference clock (256×/384× Fs); **not wired here** — PCM5102 has internal PLL, INMP441 doesn't need it.
- **Clock master / slave** — exactly one device drives BCLK+LRCLK. Here the **Pi is master**; both mic and DAC are slaves sharing the Pi's clocks → full-duplex on one bus, separate data wires only.
- **L/R pin (INMP441)** — selects which stereo slot the mono mic fills; GND=left, VDD=right, floating=garbage.
- **"status update device" / "status-update device"** — user's separate device whose functions the friend should be able to call.
- **Thin client + server brain** — the shared architecture pattern (treebot, Xiaozhi): device does audio capture/wake, server does STT/LLM/TTS.
- **Xiaozhi** — `78/xiaozhi-esp32` (firmware) + `xinnan-tech/xiaozhi-esp32-server` (self-hosted backend).

## 8. Anything surprising or non-obvious
- The mic and DAC **physically share the two clock pins (GPIO18/19)**; only the data wires differ. That shared-clock trick is the whole basis for running input+output simultaneously on one I²S bus — easy to miss and easy to mis-wire.
- You **cannot** get input by stacking a second overlay onto an output-only one — one Pi I²S card = one overlay; the bidirectional `googlevoicehat-soundcard` is the workaround.
- INMP441 data is **24-bit left-justified inside a 32-bit slot** — recording as S16_LE loses/garbles it; must grab S32_LE and read the upper bits.
- With L/R→GND, **only the left channel has audio**; the right channel being silent/noisy is expected, not a fault. Pull left in software.
- A **loud constant tone or full-scale noise** on capture ≈ L/R floating or BCLK/WS swapped — check those two first.
- `dtparam=audio=off` is correct and intentional on Pi 5 / Pi Zero 2: the analog jack was removed and HDMI audio is handled by the VC4 KMS driver, so disabling it frees the PCM pins (GPIO18–21) for I²S.
- The **`waveshare35a` LCD overlay may silently claim GPIO18** (shared BCLK) — a latent, intermittent-failure-prone clash that's miserable to debug; pre-empt by disabling the LCD during audio bring-up.
- `googlevoicehat-soundcard` historically expected an **amp-enable GPIO**; usually fine without it, but it's the first suspect for "card appears, playback silent".
- Xiaozhi's **OpenAI-compatible backend requirement dovetails with the user's separate "provider-independent LLM client" principle** — the same swappable-backend discipline applies to both the desktop-friend and the user's other ("Sein und Zeit") project, and the internal loop can live in the same server-side layer.
- Going Xiaozhi means the user's **unfinished treebot work (Realtime-API migration, threading fixes) is largely obviated** rather than needing completion.
