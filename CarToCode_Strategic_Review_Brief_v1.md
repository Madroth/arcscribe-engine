# Strategic Review Brief — Tier 3 & 4 Questions

**For:** Claude.ai and Gemini, kicking off the next collaborative review round
**From:** Claude Code (Opus 4.7) on behalf of the user (PM)
**Date:** 2026-05-29
**Repo:** https://github.com/Madroth/arcscribe-engine

---

## Project Context

`arcscribe-engine` is a workflow automation system bridging hands-free voice discovery during car commutes with active Claude Code development at a Linux desktop. The v4.0 architecture is finalized after three rounds of collaborative Claude + Gemini review. MVP build has not started — currently at Phase 1 (repo structure done) with Phase 0 (STT spike) pending.

If you need full context, read these in the repo:

| File | Purpose |
|------|---------|
| `CarToCode_Architecture_Document_v4.md` | Finalized v4.0 architecture |
| `CarToCode_Architecture_Review_Gemini_v1.md` | Gemini's prior review with locked revisions |
| `CarToCode_Architecture_Review_ClaudeCode_v1.md` | Claude Code's critique pass |
| `DECISIONS.md` | Chronological decision log (current state) |
| `BEHAVIORS.md` | Operational config |
| `DISCOVERY.md` | Outstanding items |

---

## What This Round Needs to Resolve

Six strategic / design decisions that benefit from adversarial review before being encoded. Listed in priority order — #1 informs everything else; #2 and #3 are load-bearing for Phase 4.

### 1. First Dogfood Project *(highest priority — answer this first)*

`arcscribe-engine` is supposed to be built *using* the voice-to-Claude-Code workflow it itself creates. That is circular — you cannot dogfood the system before it exists.

**Decision needed:**
- Is `arcscribe-engine` the first test subject (with manual transcription bootstrapping until the system works)?
- Or is there a separate first project the workflow drives once built (giving us a real-world target while `arcscribe-engine` is being developed)?

**Why it matters:** This shapes what content seeds `CONTEXT_TEMPLATE.md`, what the first car sessions are actually about, and what success looks like at MVP.

### 2. Inbox JSON Schema Design *(load-bearing for Phase 4)*

The central data structure of the entire system. Every Haiku-extracted sync message uses this schema. Currently listed as "resolve during build" in v4.0; Claude Code pulled it forward because Phase 4 (Haiku scribe) and Phase 5 (sync processing) both depend on it.

**Decisions needed:**

- **Complete list of legal sync types.** Known candidates: `decision`, `feature`, `discovery`, `refusal`, `invalidation`, `pause_requested`, `revert_requested`. Others?
- **Required fields per type.** At minimum: `id`, `timestamp`, `type`, `payload`, `source_models`. What else?
- **ID and reference scheme.** How do later entries supersede earlier ones? Monotonic IDs? Content-addressed? Explicit `supersedes: <id>` field?
- **Invalidation semantics.** When `"ignore that"` triggers, what gets dropped — just the target exchange, or also subsequent items that built on it? How far back is too far?
- **Schema version field** for future evolution.

**Output format requested:** A worked example JSON file showing one of each sync type.

### 3. Haiku System Prompt Design *(load-bearing for Phase 4; depends on #2)*

The prompt that instructs Haiku to extract structured JSON of the schema types defined in #2, from the latest exchange + the 800-token sliding window context.

**Decisions needed:**

- Worked input → output examples for each sync type
- Guidance for disambiguation ("user said yes — yes to what?" — how does Haiku resolve references?)
- Behavior when no extractable content is found (empty array? `null`? skip output?)
- How to handle the `"ignore that"` voice command specifically
- Should Haiku ever ask for clarification, or always commit to its best guess?

### 4. Desktop Workflow Trigger *(affects Phase 5)*

When the user arrives home from a drive, how does Claude Code know to process `/inbox`?

**Options to consider:**

- **Manual:** user opens Claude Code and says "process inbox"
- **Cron:** hourly check
- **inotify / file watch:** instant trigger on new files
- **GitHub webhook → systemd service**
- **Pull-based:** Claude Code checks at every session start

**Tradeoffs:** resource usage, responsiveness, user friction, security surface area.

### 5. Privacy Posture *(affects schema, refusal handling, and repo settings)*

Car conversations may capture sensitive content: names, addresses, business strategy, personal information, third-party names. Everything currently flows to GitHub.

**Decisions needed:**

- Should there be a redaction step before push? If yes, who runs it — Haiku, a separate filter, on-device regex?
- Should `/inbox` be encrypted at rest in GitHub (e.g., age-encrypted blobs that Claude Code decrypts before processing)?
- Should the repo be private as a baseline (currently public)?
- Any topics or trigger phrases that should automatically pause the session?

### 6. Mid-Drive Failure Mode UX

When the app freezes or the API hangs while driving, the user cannot safely debug. Recovery flow needs to be hands-free.

**Options to consider:**

- Voice command: `"reset session"` triggers a clean restart
- Automatic restart after N seconds of silence (with risk of false positives)
- Kill switch via Bluetooth button hold
- Fail silently, surface error in `WORKLOG.md` at desktop, end session

**Goal:** identify the failure modes worth handling explicitly vs. those that can fall back to "end session, restart at next stoplight."

---

## Output Format Requested

A single markdown doc following the pattern of the prior reviews:

- **Suggested filename:** `CarToCode_Architecture_Review_Gemini_v2.md` (or `CarToCode_Strategic_Decisions_v1.md`)
- Each decision clearly stated with rationale
- Schema-style decisions (#2, #3) should include concrete worked examples
- Format consistent with prior review docs in the repo

Drop the result into the repo (commit + push, or paste into the next Claude Code session and Claude Code will commit it).

---

## Constraints to Honor

- **MVP scope cuts** from Gemini v1 review remain locked: no context polling, no rolling summarization, no model selector UI for v1.
- **All architecture decisions** from v4.0 + Gemini v1 + the current `DECISIONS.md` remain in force unless explicitly revisited and overridden.
- **Hands-free absolute requirement** — if a proposed UX cannot work while driving without screen touches, it is not in scope.
- **Token minimization** at every layer remains a guiding principle.

---

## Next Step After This Review

Once this doc lands in the repo, Claude Code will:

1. Encode the schema as a real file (e.g., `schemas/inbox_v1.schema.json`).
2. Update `DECISIONS.md`, `BEHAVIORS.md`, `BUILDORDER.md` to reflect the new decisions.
3. Move resolved items out of `DISCOVERY.md`.
4. Begin Phase 0 STT spike planning (assuming Android Studio install is complete).
