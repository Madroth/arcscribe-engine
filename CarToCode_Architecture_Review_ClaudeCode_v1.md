# Car-to-Code Architecture Review — Claude Code Perspective

**Reviewer:** Claude (Claude Code, Opus 4.7, 1M context)
**Review date:** 2026-05-28
**Intended audience:** Google Gemini and Claude.ai (next review round)

**Documents reviewed:**
- `CarToCode_Architecture_Document_v4.md` (v4.0 finalized architecture)
- `CarToCode_Development_Roadmap.md` (8-phase build plan)

**Purpose of this document:** Fresh-eyes critique from Claude Code, contributed for consideration in subsequent collaborative review rounds. The architecture has already been through three rounds of Claude + Gemini review and is marked as finalized for build. The notes below identify areas where, in my view, residual risk or design ambiguity warrants re-examination *before* Phase 1 begins. This is intentionally a critique pass, not a validation pass — strengths are summarized briefly so reviewers can focus on the contested points.

---

## 1. Strengths worth preserving

These design decisions are, in my view, load-bearing and correct. Subsequent reviews should be cautious about altering them:

- **Inbox/Outbox single-writer pattern.** The cleanest decision in the document. Git is a poor message queue, but constraining the phone to write only to `/inbox` and Claude Code to write only to core files eliminates the entire class of merge-conflict pain that typically kills approaches like this.
- **Sonnet + Haiku dual-model split.** Economically sound and latency-aware. Asking Sonnet to summarize its own transcript would introduce spikes; offloading that to Haiku is correct.
- **Phase 2 voice-loop validation before anything else.** Validating "can voice actually work in a car" before building the rest of the system is the right sequencing.
- **Phone independence from the Linux box.** Removes a real single point of failure. Right call.
- **Heartbeat + 12-min staleness detection.** Simple, robust coordination primitive between phone and desktop.

---

## 2. Concerns ranked by impact

### 2.1 Android SpeechRecognizer in real car conditions is the biggest unvalidated risk

The document lists "optimal SpeechRecognizer configuration for car environment" as a discovery item to resolve *during* the build. I believe this should be a **Phase 0 spike** before Phase 1.

**Why this matters:**
- Android's native `SpeechRecognizer` API is designed for short utterances, not continuous dialogue. Its accuracy on multi-minute continuous speech with road noise, HVAC, music bleed-through, and accent variability is materially worse than the document assumes.
- If STT proves unreliable in real driving conditions, the entire architecture changes — you would need Whisper on-device, or a cloud STT (Deepgram, Google Cloud Speech, AssemblyAI) — which has cost, latency, and credential-management implications that ripple through every layer.

**Recommendation:** Run a single weekend test (one Android prototype, one drive cycle, no other features) before Phase 1. If STT fails, every downstream decision needs revisiting.

### 2.2 Phase 2's "<3 second latency" target is aggressive

The end-to-end chain is: STT finalization → cellular network round-trip → Claude API processing → TTS startup. Cellular variability alone routinely exceeds 1 second. Hitting <3s consistently in mixed coverage is tight.

**Recommendation:** Budget more honestly. 3–5 seconds is probably the realistic range. Validate "what feels broken" in user testing before the latency target is locked.

### 2.3 Haiku-as-scribe has a silent semantic-failure mode

The validation layer catches **malformed JSON** (retry once, then flag unparsed). It does not catch **semantically wrong extractions** — Haiku confidently outputting "user decided X" when the user did not actually decide X. Those land in `DECISIONS.md` and Claude Code acts on them.

**Recommendation:** Add a periodic audit mechanism. Options:
- Sample 5–10% of Haiku extractions for cross-check by Sonnet at end-of-session.
- Surface a "summary of what I extracted" at session end via TTS so user can flag wrong decisions before they reach the desktop.
- `WORKLOG.md` entry listing all decisions extracted, for periodic user review at desktop.

### 2.4 30-minute context-refresh polling is too slow for a real-time loop

If Claude Code makes a decision at minute 0 of a drive, the car session will not see it for up to 30 minutes. For a system designed around bidirectional collaboration, this latency undermines the loop.

**Recommendation:** Either shorten polling to 5–10 minutes, or rely entirely on the existing `INJECTIONS.json` event-driven pattern and remove polling for core-file updates.

### 2.5 Token cost is not projected anywhere

The document repeats "minimize token usage" but never projects expected monthly spend. Rough back-of-envelope: a 1-hour drive with Sonnet conversation + Haiku scribe + 15-minute summarization is approximately $2–5. Daily commutes = $60–150/month.

This is not prohibitive, but it should be **explicitly budgeted before build**, with a guardrail (e.g., session-token cap, monthly spend alert) added to BEHAVIORS.md.

### 2.6 PAT approach will age badly

Classic PATs with `repo` scope grant access to **every repo the user owns**. They are also being deprecated in favor of fine-grained tokens, which have a 1-year maximum lifetime.

**Recommendation:**
- Use a fine-grained token scoped only to `arcscribe-engine`.
- Document rotation procedure now, in `BEHAVIORS.md` or a `SECURITY.md`.
- Plan for graceful failure when the token expires — the system should not lock up mid-drive.

### 2.7 Continuous listening does not handle "user is talking to a passenger"

Audio Focus handles incoming audio (GPS, calls). It does not handle the **inverse** problem — conversations with passengers, podcast resumption, ambient speech — all of which get transcribed and sent to Sonnet.

**Recommendation:** Add either:
- A wake-word convention ("Hey Claude…").
- A push-to-talk default with the steering-wheel button as confirmation.
- A heuristic ("ignore utterances under N words when the prior exchange is more than M minutes old").

### 2.8 MVP scope (Section 11) is large for a solo PM project

Section 11 lists 19 features for MVP. Phase 5 alone (GitHub sync + Claude Code inbox processing + invalidation logic) is multi-week work. Several features could be deferred without breaking the core loop:

- Model selector UI (defaults are fine for MVP; selector is post-MVP)
- Backpressure handling (Haiku desync is unlikely until later phases anyway)
- "Any updates from the desktop?" voice command (covered by polling; nice-to-have)
- Rolling Haiku summarization (only needed for multi-hour drives; defer until Phase 6)

A leaner MVP gets to "working car-to-desk loop" faster and surfaces real problems sooner.

---

## 3. Things that look easier than they appear

These are not deal-breakers but the roadmap treats them as smaller than they likely are:

- **Bluetooth media events vary significantly by vehicle and headunit.** Phase 7 line "test across different vehicle Bluetooth implementations" is one line that could be days of debugging. Real-world media-button intercept is messy on modern Android.
- **Android 12+ background service restrictions.** Battery-saver modes and Doze mode kill background processes aggressively. Sustained background sync across the duration of a drive is harder than the document implies.
- **Heartbeat-as-Git-commit creates noisy history.** Every 3 minutes for the duration of every drive = thousands of commits per year that are pure heartbeat. Consider writing heartbeat to a non-committed location (GitHub Issues API, Gist, or even a dedicated branch that's force-pushed).
- **The 800-token sliding window for Haiku may be too tight for reference resolution.** Conversational references ("yes, the one I mentioned earlier") can span longer than 800 tokens. Worth tuning.

---

## 4. Open questions for the next review round

For Gemini / Claude.ai consideration:

1. **STT alternative path.** What is the fallback if Android SpeechRecognizer fails the Phase 0 spike? Should the architecture include an STT abstraction layer from day one to make swapping easier later?
2. **Refusal handling.** Claude occasionally refuses or hedges on benign topics. What is the in-car UX when Sonnet refuses? Does it tell the user, retry, or silently fail?
3. **Multi-passenger driving.** If the user occasionally drives with a passenger, does the system pause entirely, switch to push-to-talk, or attempt to filter? Not addressed in v4.0.
4. **Session resumption after phone call.** A 5-minute phone call mid-drive: does the session resume automatically, or does the user need to re-start it? Crash resilience is addressed but graceful interruption is not.
5. **Inbox file retention.** What happens to `/inbox` files if Claude Code is not run for a week? No cap, no rotation. Worth defining.
6. **`DECISIONS.md` revision history.** If Haiku extracts a wrong decision and Claude Code acts on it, what is the rollback procedure beyond `git revert`? Worth documenting.

---

## 5. Recommended action before Phase 1

A "Phase 0 spike" — 1 to 2 days of work — to validate the assumptions most likely to invalidate the architecture:

1. **STT in a real car.** Hello-world Android app, push-to-talk to start, 20-minute drive, transcript review. Pass/fail: can user understand 90%+ of transcript with no manual correction?
2. **End-to-end latency.** Same prototype, hardcoded Sonnet call, measured time from speech-end to TTS-start across 20 exchanges. Pass/fail: is the median under 4 seconds?
3. **Bluetooth media-button intercept.** Test the steering-wheel Next-Track barge-in on the user's actual vehicle. Pass/fail: does it work reliably?

If all three pass, Phase 1 proceeds with confidence. If any fail, the architecture document needs revision before committing to the 8-phase build.

---

## 6. Summary position

The v4.0 architecture is well-considered and the Inbox/Outbox pattern is the right backbone. The largest residual risks are all concentrated in **Phase 2 assumptions** (STT, latency, hands-free reliability), not in the elegant parts of the design. A short Phase 0 validation spike would meaningfully de-risk the full build before significant investment.

The MVP scope is, in my view, somewhat over-broad for a solo PM with limited build time, and I would encourage the next review round to identify cuts.

I have no objections to the core architecture. The concerns above are calibrations, not rewrites.

---

*Submitted for review by Gemini and Claude.ai. Open to discussion and revision.*
