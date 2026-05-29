# Car-to-Code Architecture Review — Gemini Perspective & Final Decisions

**Reviewer:** Google Gemini & User (Product Manager)
**Review Date:** May 2026
**Target Audience:** Claude Code (for immediate implementation)

**Purpose:** This document serves as the formal response to `CarToCode_Architecture_Review_ClaudeCode_v1.md`[cite: 3]. It resolves all open questions, defines the Phase 0 spike, and officially limits the MVP scope so development can begin.

---

## 1. The Phase 0 Spike & STT Resolution
We agree with Claude that the acoustic environment of a vehicle is the highest risk factor[cite: 3]. 
*   **Decision:** We are bypassing Android's native `SpeechRecognizer`[cite: 3]. 
*   **Action:** Claude Code will build an **STT Abstraction Layer** in Android (Kotlin)[cite: 3]. For Phase 0, we will implement **Vosk** (via their Android SDK) as the primary offline, continuous-listening engine because of its low battery drain and fast partial-result streaming. 
*   **Fallback:** If the Phase 0 spike proves Vosk struggles with technical jargon, the abstraction layer will allow us to easily swap to `Whisper.cpp` locally.

## 2. Resolving "Git Noise" & State Management
Claude rightly pointed out that a 3-minute heartbeat pushed to GitHub will destroy the repository's commit history[cite: 3].
*   **Decision:** The `SESSION_STATE.json` heartbeat will **not** be committed to the repository. 
*   **Action:** The Android app will write the heartbeat/state to a dedicated GitHub Gist or update a GitHub Repository Variable via the API. Claude Code will check this variable to determine if a session is active, keeping the core repo history clean.

## 3. Answers to Claude's Open Questions
1.  **Refusal Handling:** If Sonnet hits a safety filter, the Android app will catch the error and trigger TTS: *"I can't answer that."*[cite: 3] It will then write a `{"type": "refusal", "raw_prompt": "..."}` block to the `/inbox` for the user to review later at the desktop.
2.  **Multi-Passenger Driving:** We are implementing a "Deaf Mode." A double-tap on the steering wheel's Bluetooth media button will pause the Vosk STT engine entirely. A single tap will resume it[cite: 3].
3.  **Phone Call Interruptions:** If a call comes in, the app saves state locally and relinquishes audio focus[cite: 3]. When the call ends, it does **not** auto-resume. It waits for a physical screen tap or media button press to re-engage[cite: 3].
4.  **Inbox Retention:** No capping or rotation is needed[cite: 3]. If the desktop goes unused for a week, Claude Code simply processes the timestamped `/inbox` files chronologically[cite: 3]. 
5.  **DECISIONS.md Rollback:** To mitigate Haiku's semantic-failure risk[cite: 3], Claude Code must append the Git commit hash to the `WORKLOG.md` entry for every processed inbox batch. If a hallucination is caught, the user can easily run a targeted `git revert`[cite: 3].

## 4. Calibration, Budgeting & Security
*   **Latency Target:** We accept Claude’s calibration[cite: 3]. We are adjusting the target to a realistic **3–5 seconds** for end-to-end voice turn-around in real-world cellular conditions[cite: 3].
*   **Token Guardrails:** Claude Code will add a monthly token spend alert system and session-cap logic to `BEHAVIORS.md` to prevent unexpected API costs[cite: 3].
*   **Security Tokens:** We are abandoning classic Personal Access Tokens (PATs)[cite: 3]. The app will use a fine-grained GitHub token scoped strictly to the `architect-and-scribe` repository[cite: 3].
*   **Sliding Window Tuning:** The token-based window size for the Haiku scribe will be made a configurable parameter in `BEHAVIORS.md` so we can easily tune it if reference resolution becomes an issue[cite: 3].

## 5. MVP Scope Reductions
To get to a working core loop faster, we are aggressively deferring the following features to v2.0[cite: 3]:
*   **Context Polling:** Dropped[cite: 3]. The MVP assumes the user is driving, not having someone code concurrently at the desktop[cite: 3].
*   **Rolling Summarization:** Dropped[cite: 3]. For the MVP, if a drive hits the token limit, the app cleanly ends the session and forces a new one[cite: 3].
*   **Model Selector UI:** Dropped[cite: 3]. Claude Sonnet and Claude Haiku will be hardcoded into the API calls for v1.0[cite: 3].

---
**Next Step for Claude Code:** Review these decisions and execute the Phase 0 STT Spike using Vosk[cite: 3].
