# arcscribe-engine

Car-to-Code Workflow Automation — bridges hands-free voice discovery during car commutes with active Claude Code development at the desktop, via a GitHub-backed Inbox/Outbox pattern.

## 👉 Start here

**Any AI joining the project:** read `AI_HANDOFF.md` first. It contains the current state, what each AI should do next, and how to resume sessions.

## Repository Map

### Architecture & Review Documents

- `CarToCode_Architecture_Document_v4.md` — finalized v4.0 architecture
- `CarToCode_Development_Roadmap.md` — 8-phase build plan
- `CarToCode_Architecture_Review_ClaudeCode_v1.md` — Claude Code's critique pass
- `CarToCode_Architecture_Review_Gemini_v1.md` — Gemini's response & locked decisions

### Core Operating Files (Claude Code is the sole writer)

| File | Purpose |
|------|---------|
| `ARCHITECTURE.md` | System architecture summary |
| `BEHAVIORS.md` | Operational rules, timer intervals, risk tiers, model defaults |
| `DECISIONS.md` | Chronological log of finalized decisions |
| `DISCOVERY.md` | Outstanding questions, prioritized |
| `FEATURES_MVP.md` | MVP feature list |
| `FEATURES_FUTURE.md` | Post-MVP feature list |
| `BUILDORDER.md` | Recommended build sequence |
| `CONTEXT_TEMPLATE.md` | Template for session context document |
| `WORKLOG.md` | Auto-updated log of inbox processing batches |
| `AGENT_STATUS.md` | Last run status |
| `CONFLICT_LOG.md` | Detected data conflicts |
| `INJECTIONS.json` | Desktop-to-car priority message queue |

### Folders

- `/inbox` — phone-written JSON sync files (single writer: phone)
- `/archive` — processed inbox files moved here by Claude Code
- `/summaries` — daily/weekly activity summaries
- `/tests` — test specifications per feature

### Notes

- `SESSION_STATE.json` heartbeat is intentionally **not** in this repo — it lives in a GitHub Gist or Repository Variable to keep commit history clean.
- Phone writes ONLY to `/inbox`. Claude Code writes to everything else. This single-writer discipline eliminates Git concurrency conflicts.
