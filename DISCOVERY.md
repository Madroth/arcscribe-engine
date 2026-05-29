# Outstanding Discovery Items

Open questions and unresolved items, ordered by dependency. The Android app surfaces top items first in car sessions.

Format: `[ ] Item — context / dependency`

---

## Phase 0 Spike — Pending Outcomes

- [ ] Vosk transcript accuracy in real car conditions (target: ≥90% understandable, no manual correction).
- [ ] End-to-end median voice turn-around latency (target: ≤4 seconds).
- [ ] Bluetooth steering-wheel Next-Track barge-in reliability in user's vehicle (target: reliable across 20 attempts).

## Pending Strategic / Design Decisions

Best resolved in adversarial review with Claude.ai + Gemini before encoding here.

- [ ] **First dogfood project.** Is `arcscribe-engine` itself the test subject (circular — can't dogfood before it exists), or is there a separate first project the workflow will drive?
- [ ] **Desktop workflow trigger.** How does Claude Code know to process inbox after a drive — manual command, cron, inotify watch, autotrigger? Affects Phase 5 design.
- [ ] **Privacy posture.** Any redaction step before car-conversation content reaches GitHub? Topics or terms to filter?
- [ ] **Mid-drive failure mode UX.** If the app freezes or API hangs while driving, what's the recovery flow? Voice command? Reboot at a stoplight? Kill switch?
- [ ] **Inbox JSON schema design.** Legal sync types, required fields, ID/reference scheme, supersede/invalidate semantics. **Pulled forward from "resolve during build" — finalize before Phase 4 starts.**
- [ ] **Haiku system prompt design.** Same — design before Phase 4 begins. Depends on inbox JSON schema.

## Pending Setup Actions

See `DECISIONS.md` → "Carryover Pending Actions" for the canonical list (Android Studio install, dedicated API key generation, GitHub token, Gist creation).

## To Resolve During Build (carried from v4.0 §15, trimmed)

- [ ] Optimal Vosk configuration for car environment (gain control, noise suppression, partial-result handling).
- [ ] GitHub API rate limit handling specifics — what counts as "too fast"?
- [ ] Android background service behavior across Android 12+ and battery-saver modes.
- [ ] Context document format optimization (see `CONTEXT_TEMPLATE.md`).
- [ ] GitHub issue template design.
- [ ] Fine-grained GitHub token rotation procedure (1-year expiry).

## Lower Priority — Address as Encountered

- [ ] Should `INJECTIONS.json` polling cadence be user-configurable?
- [ ] What format should `/summaries` files use? (Daily-summary template TBD.)
- [ ] How should `WORKLOG.md` size be managed long-term? (Currently no cap.)
