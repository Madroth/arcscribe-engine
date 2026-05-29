# Decisions Log

Chronological log of finalized architecture and product decisions. Append new entries at the bottom. Reference source review documents where applicable.

---

## 2026-05-28 — v4.0 Architecture Finalization (Claude + Gemini, three rounds)

Source: `CarToCode_Architecture_Document_v4.md`

Headline locked decisions:

- **Dual-Model.** Sonnet for primary conversation; Haiku for silent JSON extraction.
- **Inbox/Outbox Single-Writer Pattern.** Phone writes only to `/inbox`; Claude Code is sole writer to core files. Eliminates Git concurrency conflicts.
- **Token-Bounded Sliding Window.** Default 800 tokens for Haiku input window (configurable in `BEHAVIORS.md`).
- **Micro-Batch Sync.** Push to GitHub every 3 minutes OR when queue reaches 10 items.
- **Bluetooth Steering-Wheel Controls Required for MVP.** Not optional — noisy car environments make silence detection unreliable without this fallback.
- **Android Keystore for Credentials.** Hardware-backed encryption, cloud-encrypted backup.
- **Phone Independence.** Car session works regardless of Linux box status.
- **Heartbeat + 12-Minute Staleness Threshold.** Crash detection primitive.
- **Risk-Tier Autonomous Execution.** Low risk auto-commits, medium goes to feature branch, high holds for approval.

---

## 2026-05-28 — Post-v4.0 Revisions (Gemini Review)

Source: `CarToCode_Architecture_Review_Gemini_v1.md`

### STT Engine

- **Replaced:** Native Android `SpeechRecognizer` → **Vosk** (offline, continuous-listening, low battery drain, fast partial-result streaming).
- **Implementation:** STT Abstraction Layer in Kotlin so the engine is swappable.
- **Fallback:** Whisper.cpp held behind the abstraction layer if Vosk underperforms in Phase 0 spike.

### State Management

- **`SESSION_STATE` heartbeat moved OUT of repo.** Will live in a dedicated GitHub Gist or GitHub Repository Variable, updated via API.
- **Rationale:** A 3-minute commit cadence for the heartbeat would pollute repo history with thousands of meaningless commits.

### Rollback Discipline

- **Claude Code appends Git commit hash to `WORKLOG.md`** for every processed inbox batch.
- **Rationale:** If Haiku produces a semantically wrong extraction that lands in `DECISIONS.md`, user can run a targeted `git revert <hash>` to undo.

### Refusal Handling

- On Sonnet safety-filter trigger:
  - TTS: `"I can't answer that."`
  - Write `{"type": "refusal", "raw_prompt": "...", "timestamp": "..."}` to `/inbox`.
  - Continue session normally.

### Multi-Passenger Driving ("Deaf Mode")

- **Toggle on/off:** double-tap steering-wheel Next-Track button. Pauses Vosk STT entirely.
- **Resume:** single tap on the same button.

### Phone Call Handling

- App relinquishes audio focus on incoming call; preserves session state locally.
- **No auto-resume.** User must physically tap screen or press media button to re-engage after call ends.

### Inbox Retention

- **No cap, no rotation.** Claude Code processes timestamped files chronologically regardless of backlog size.

### Latency Target

- **Adjusted to 3–5 seconds median** end-to-end (from <3 seconds in v4.0). Acknowledges realistic cellular variability.

### Token Spend Guardrails

- **Per-session input-token cap:** 250,000 (Sonnet + Haiku combined).
- **Monthly soft alert:** USD $100.
- **Monthly hard cutoff:** USD $200.

### GitHub Token

- **Fine-grained personal access token only.** Scoped strictly to `arcscribe-engine`.
- **Classic PATs abandoned.** Overscoped (grants access to all repos owned by user).

### Sliding Window

- **Configurable via `BEHAVIORS.md`.** Default 800 tokens for Haiku input; tunable if reference resolution proves weak.

### MVP Scope Cuts (deferred to v2.0)

- **Context-refresh polling** (assumes user is driving, not having someone code concurrently at desktop).
- **Rolling Haiku summarization** (if session hits token cap, app cleanly ends session and forces a new one).
- **Model selector UI** (Sonnet + Haiku hardcoded for v1).

Additional cuts adopted per Claude Code's review:

- **Backpressure handling** (unlikely to trigger until later phases).
- **"Any updates from the desktop?" voice command** (covered implicitly by injection mechanism).

---

## 2026-05-28 — Phase 0 STT Spike Required Before Phase 1 Build

Source: `CarToCode_Architecture_Review_ClaudeCode_v1.md` (proposal); Gemini accepted in v1 response.

Before any Phase 1 implementation work begins, a Phase 0 spike validates:

1. **Vosk transcript accuracy in real car conditions.** Pass: ≥90% understandable transcript with no manual correction across a 20-minute drive.
2. **End-to-end voice turn-around latency.** Pass: median ≤4 seconds across 20 exchanges.
3. **Bluetooth steering-wheel Next-Track barge-in reliability** in user's actual vehicle. Pass: reliable across 20 attempts.

If any criterion fails, architecture is revised before continuing.
