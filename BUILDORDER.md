# Build Order

Recommended phase sequence. Adapted from `CarToCode_Development_Roadmap.md` with the Phase 0 STT spike inserted and MVP-cut phases trimmed.

Dependencies are strict — do not skip ahead. Each phase has a validation gate before the next begins.

---

## Phase 0 — STT Validation Spike

**Goal:** Validate that Vosk STT works in real car conditions before committing to the architecture.

**Tasks:**
- Minimal Kotlin Android prototype with STT Abstraction Layer.
- Vosk integration only — no GitHub sync, no Haiku, no Sonnet.
- Hardcoded "press button, speak, transcript printed to screen" loop.

**Validation gates (all three must pass):**
1. Vosk transcript accuracy ≥90% in 20-minute real drive.
2. End-to-end median latency ≤4 seconds across 20 exchanges.
3. Bluetooth steering-wheel Next-Track barge-in reliable across 20 attempts in user's vehicle.

**If any gate fails:** revise architecture before Phase 1.

---

## Phase 1 — Foundation: GitHub Repository Setup *(in progress)*

**Goal:** Establish the source of truth.

**Tasks:**
- Folder structure: `/inbox`, `/archive`, `/summaries`, `/tests`. ✅
- Core markdown files seeded with current content. ✅
- `INJECTIONS.json` initialized. ✅
- GitHub Issue labels configured.
- Fine-grained GitHub token generated, scoped to this repo.
- Credential management approach documented.

**Validation:**
- All folders exist and tracked in Git.
- All core files have seed content.
- Token created, stored securely on workstation, not in repo.

---

## Phase 2 — Voice Bridge Skeleton

**Goal:** Hello-world single-turn voice conversation with Sonnet, in a car.

**Tasks:**
- Android Studio project setup.
- Single "Start Session" button.
- Vosk STT (built on Phase 0 abstraction).
- Android `TextToSpeech` for output.
- Single hardcoded Sonnet API call.
- Claude API key in Android Keystore.

**Validation:**
- User can tap, speak, hear Sonnet respond.
- Tested in driving conditions.
- Median latency in 3–5 second band.
- API key never in logs or version control.

---

## Phase 3 — Conversation Loop

**Goal:** Multi-turn dialogue with context loaded from GitHub.

**Tasks:**
- Multi-turn conversation memory.
- Pull core markdown files from GitHub at session start.
- Assemble dynamic context document per `CONTEXT_TEMPLATE.md`.
- Load context as system prompt.
- Token tracking via Claude API response deltas.
- Token-based sliding window (configurable from `BEHAVIORS.md`).
- Audio Focus management.
- 15-second HTTP timeout with TTS feedback.
- Local SQLite save before every API call.

**Validation:**
- Multi-turn conversation flows naturally.
- Context loads correctly.
- Token usage bounded.
- Audio Focus correctly ducks/pauses for system interruptions.
- App restart after crash recovers in-progress session.

---

## Phase 4 — Haiku Scribe

**Goal:** Silent parallel JSON extraction, queued locally (no GitHub push yet).

**Tasks:**
- Parallel Haiku API call per exchange.
- Haiku system prompt for structured JSON extraction.
- JSON schema validator.
- Retry-once-then-flag-unparsed on validation failure.
- Queue valid outputs to local SQLite (no push yet).
- Model telemetry in every JSON sync block.

**Validation:**
- Haiku consistently produces valid JSON.
- Schema validation catches malformed output.
- Retry logic works.
- Cost analysis confirms ~85–90% Sonnet / ~10–15% Haiku token distribution.

---

## Phase 5 — GitHub Sync: Inbox Pattern Activation

**Goal:** Phone → `/inbox`; Claude Code processes and archives.

**Tasks:**
- Micro-batch sync trigger (3 min OR 10 items).
- GitHub API integration for `/inbox` push.
- Offline queue resilience.
- `SESSION_STATE` heartbeat to Gist / Repository Variable (NOT repo).
- Claude Code logic: chronological inbox processing.
- Claude Code respects `SESSION_STATE` (waits if active+recent; proceeds if heartbeat >12 min).
- Claude Code moves processed files to `/archive`.
- Claude Code appends commit hash to `WORKLOG.md`.
- `"Ignore that"` voice command with invalidation JSON.

**Validation:**
- Car session data flows reliably from phone to `/inbox`.
- Claude Code correctly processes inbox post-session.
- Core markdown files update correctly.
- Heartbeat staleness triggers crash recovery.
- No Git conflicts.

---

## Phase 6 (Slim) — Bidirectional Injection + End-of-Session Cleanup

**Note:** Original Phase 6 was largely deferred to post-MVP (rolling summarization, context polling). Only injection + cleanup remain.

**Tasks:**
- App polls `INJECTIONS.json` queue.
- Surface injections at the moment Sonnet TTS finishes (deterministic timing).
- Distinct audio cue before any injection.
- End-of-session final cleanup pass — Haiku reviews full conversation, catches missed items.

**Validation:**
- Injections never interrupt conversation flow.
- Final cleanup catches items missed by real-time sync.

---

## Phase 7 — Hardware Integration: Bluetooth Steering Wheel

**Goal:** Reliable physical override in noisy car.

**Tasks:**
- Bluetooth media event listener.
- Next-Track during Sonnet speech → barge-in.
- Next-Track during user speech → manual send.
- Double-tap → Deaf Mode toggle.
- Ensure mappings don't conflict with other media apps.

**Validation:**
- Barge-in halts Sonnet mid-response.
- Manual send overrides silence detection reliably.
- Deaf Mode toggle works reliably.
- Works in user's actual vehicle.

---

## Phase 8 — Polish & Production Readiness

**Goal:** Daily-use stability.

**Tasks:**
- Daily encrypted cloud backup of credentials/configs.
- User guide based on actual built system.
- Update `ARCHITECTURE.md` for any deviations from v4.0 spec.
- Operational procedures: what to do when things break.
- Claude Code terminal session summary with grouped activity output.
- Daily and weekly `/summaries` generation by Claude Code.
- Final security review.
- Full end-to-end stress test.

**Validation:**
- System runs reliably across consecutive daily drives.
- Backups verified via simulated phone-loss recovery.
- Documentation enables another person/AI to operate the system.
- Security review identifies no critical vulnerabilities.

---

## Critical Path Dependencies

- Phase 0 must pass before Phase 1 implementation work.
- Phase 1 must complete before any other phase.
- Phase 2 must validate voice loop before adding conversation features.
- Phase 3 must establish conversation stability before adding scribe.
- Phase 4 must validate Haiku output quality before pushing to GitHub.
- Phase 5 must validate inbox pattern before later phases.
- Phases 6 (slim) and 7 may proceed partially in parallel; both require Phase 5.
- Phase 8 must come last.
