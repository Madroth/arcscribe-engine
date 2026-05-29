# Outstanding Discovery Items

Open questions and unresolved items, ordered by dependency. The Android app surfaces top items first in car sessions.

Format: `[ ] Item — context / dependency`

---

## Phase 0 Spike — Pending Outcomes

- [ ] Vosk transcript accuracy in real car conditions (target: ≥90% understandable, no manual correction).
- [ ] End-to-end median voice turn-around latency (target: ≤4 seconds).
- [ ] Bluetooth steering-wheel Next-Track barge-in reliability in user's vehicle (target: reliable across 20 attempts).

## High Priority — Resolve Before Phase 2 Build

- [ ] **Token-spend alert delivery mechanism.** Email? In-app TTS at next session start? GitHub Issue? Pick one.
- [ ] **Cloud backup provider for credentials.** Drive / Dropbox / S3 / other?
- [ ] **Heartbeat storage choice:** GitHub Gist vs. GitHub Repository Variable. (Repo Variable simpler if it works for our use case; Gist more flexible.)

## To Resolve During Build (carried from v4.0 §15, adjusted)

- [ ] Exact Haiku system prompt design for consistent JSON output across all sync types.
- [ ] Optimal Vosk configuration for car environment (replaces v4.0's "SpeechRecognizer config" item).
- [ ] GitHub API rate limit handling specifics — what counts as "too fast"?
- [ ] Android background service behavior across Android 12+ and battery-saver modes.
- [ ] Inbox JSON schema final design.
- [ ] Context document format optimization (see `CONTEXT_TEMPLATE.md`).
- [ ] GitHub issue template design.
- [ ] Fine-grained GitHub token rotation procedure (1-year expiry).

## Lower Priority — Address as Encountered

- [ ] Should `INJECTIONS.json` polling cadence be user-configurable?
- [ ] What format should `/summaries` files use? (Daily-summary template TBD.)
- [ ] How should `WORKLOG.md` size be managed long-term? (Currently no cap.)
