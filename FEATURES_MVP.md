# MVP Feature List

**Status:** Active scoping
**Last updated:** 2026-05-28
**Source:** `CarToCode_Architecture_Document_v4.md` Section 11; revised per `CarToCode_Architecture_Review_Gemini_v1.md` Section 5.

---

## Voice & Conversation

- Android app opens, loads context from GitHub, starts hands-free voice session with one tap.
- Sonnet conducts intelligent discovery conversation.
- Multi-turn conversation with persistent context.

## STT / TTS

- **Vosk STT** (offline, continuous) via STT Abstraction Layer in Kotlin.
- Whisper.cpp held as fallback (not required for MVP — abstraction layer makes it swappable).
- Android `TextToSpeech` for output.

## Scribe & Data Capture

- Haiku runs silently in parallel with token-bounded sliding window (800 tokens, configurable).
- Every Haiku output validated against JSON schema before queuing.
- Local SQLite backup written before every API call (crash recovery).

## Sync to GitHub

- Micro-batch sync to `/inbox` every 3 minutes OR 10 items.
- Network failure handling with 15-second timeout and local queuing.
- Bidirectional injection queue with distinct audio cue (from `INJECTIONS.json`).

## Hands-Free Controls

- Bluetooth steering-wheel media buttons for barge-in and manual send.
- **"Deaf Mode"** — double-tap steering wheel to pause STT entirely; single tap to resume.
- Audio Focus management for GPS and phone-call interruption.
- Phone call: **no auto-resume** — user must physically tap to re-engage.

## State & Resilience

- `SESSION_STATE` heartbeat written to GitHub Gist or Repository Variable (NOT to repo).
- Refusal handling: TTS `"I can't answer that."` + `{"type": "refusal"}` block to `/inbox`.

## Voice Commands

- `"Ignore that last answer"` — flags previous exchange as invalidated.
- `"Pause"`, `"Revert"`, `"End session"` — control commands.

## Desktop Side (Claude Code)

- Processes `/inbox` chronologically, respects `SESSION_STATE`.
- Risk-tier autonomous execution with pause/revert via voice command.
- Appends Git commit hash to `WORKLOG.md` for every processed batch (rollback support).

## Security

- Android Keystore credential storage with cloud-encrypted backup.
- **Fine-grained GitHub token** scoped only to this repository.

## Cost Controls

- Per-session input-token cap.
- Monthly spend soft alert + hard cutoff (thresholds in `BEHAVIORS.md`).

---

## Deferred to Post-MVP

See `FEATURES_FUTURE.md`. Items pulled from MVP per Gemini review:

- Context-refresh polling
- Rolling Haiku summarization
- Model selector UI

Items pulled from MVP per Claude Code review:

- Backpressure handling (Haiku desync threshold)
- "Any updates from the desktop?" voice command
