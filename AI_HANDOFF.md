# AI Handoff & Session Resume Brief

**Purpose:** Single onboarding doc any AI (Claude Code, Claude.ai, Gemini) can read at session start to get up to speed on `arcscribe-engine` and know what to do next. Update this doc at the end of any working session.

**Last updated:** 2026-05-29 by Claude Code
**Repo:** https://github.com/Madroth/arcscribe-engine
**Project owner:** Chris Ellis (PM)

---

## 60-Second Project Summary

`arcscribe-engine` is a workflow automation system that bridges hands-free voice discovery during car commutes with active Claude Code development at a Linux desktop. The user (a PM) holds voice conversations with Claude Sonnet via an Android app while driving; Claude Haiku runs silently in parallel as a scribe, extracting structured JSON and pushing it via micro-batch to a GitHub `/inbox`. Claude Code at the desktop processes the inbox, updates core repo files, and executes development tasks autonomously per risk tier.

Architecture is finalized (v4.0 + Gemini revisions); build has **not yet started**. Currently between Phase 1 (repo structure — done) and Phase 0 (STT spike — not started).

---

## Quick State Snapshot

| Item | Status |
|------|--------|
| Architecture v4.0 | ✅ Finalized after three rounds Claude + Gemini review |
| Gemini revisions v1 | ✅ Locked in `DECISIONS.md` |
| Claude Code critique | ✅ Committed |
| Phase 1 repo structure | ✅ Done — folders, core markdown files, labels, INJECTIONS.json |
| Tier 1 + 2 logistics | ✅ Resolved (dev env, Vosk model, billing, heartbeat storage, backup, alerts) |
| Tier 3 + 4 strategic review | 🔄 **Currently in flight** — see `CarToCode_Strategic_Review_Brief_v1.md` |
| Android Studio install | ⏸️ Pending (user action — sudo via PuTTY) |
| Phase 0 STT spike | ⏸️ Waiting on Android Studio install + strategic review completion |
| Phases 2–8 | ⏸️ Not started |

---

## The AI Roles

| AI | Surface | Role |
|----|---------|------|
| **Claude Code (Opus 4.7)** | Linux box CLI | Execution. Edits the repo directly, encodes decisions into files, commits and pushes. Has persistent auto-memory across sessions. |
| **Claude.ai** | claude.ai web/desktop | Strategic and architectural discovery. Voice-friendly. No file system access. Produces review docs. |
| **Gemini** | Google AI Studio / Gemini app | Adversarial review partner to Claude.ai. The user runs Claude + Gemini in collaborative review rounds — this is how v4.0 was produced and how Tier 3+4 will be resolved. |

**Handoff pattern that works for this project:**

1. Strategy and design happen in Claude.ai + Gemini (multi-round adversarial review).
2. The output lands in the repo as a markdown doc.
3. Claude Code reads the doc, encodes decisions into operating files (`DECISIONS.md`, `BEHAVIORS.md`, schemas, code), commits, and pushes.

Each AI sees the repo. The repo is the shared memory.

---

## Where to Read In (priority order)

For any AI joining fresh, read these in order:

1. **`README.md`** — repo map
2. **This file (`AI_HANDOFF.md`)** — current state and what to do next
3. **`DECISIONS.md`** — chronological decision log; most current state of the project
4. **`BEHAVIORS.md`** — operational config (timer intervals, risk tiers, model defaults, etc.)
5. **`DISCOVERY.md`** — open questions, what's pending
6. **`CarToCode_Architecture_Document_v4.md`** — full v4.0 architecture (long read; reference as needed)
7. **`CarToCode_Architecture_Review_Gemini_v1.md`** — locked revisions to v4.0
8. **`CarToCode_Architecture_Review_ClaudeCode_v1.md`** — Claude Code's critique pass
9. **`CarToCode_Strategic_Review_Brief_v1.md`** — the active brief for Tier 3+4 review

For schema, build order, MVP scope:
- `BUILDORDER.md`, `FEATURES_MVP.md`, `FEATURES_FUTURE.md`, `CONTEXT_TEMPLATE.md`

---

## How to Resume Each Session

### Resuming Claude Code on the Linux box

You don't need to brief Claude Code — it has auto-memory at `/home/linuxbox/.claude/projects/-home-linuxbox/memory/` that persists across sessions. Just open a new session and say:

> "Continue with arcscribe-engine. What's our current state?"

Claude Code will read this file, the memory entries, and `DECISIONS.md` to pick up.

### Resuming Claude.ai

Start a new chat and paste the repo URL plus this prompt:

> "Continue working on arcscribe-engine with me. Read https://github.com/Madroth/arcscribe-engine/blob/main/AI_HANDOFF.md first, then https://github.com/Madroth/arcscribe-engine/blob/main/CarToCode_Strategic_Review_Brief_v1.md, then orient yourself. We are mid-way through the Tier 3+4 strategic review."

### Resuming Gemini

Same pattern as Claude.ai. Open a fresh session, share the repo URL and the handoff doc URL, ask Gemini to read the brief and the latest review docs, then continue the back-and-forth.

---

## Active Work — What Each AI Should Do Next

### Claude.ai + Gemini

Work through `CarToCode_Strategic_Review_Brief_v1.md` in the listed priority order:

1. First dogfood project
2. Inbox JSON schema design
3. Haiku system prompt design
4. Desktop workflow trigger
5. Privacy posture
6. Mid-drive failure mode UX

Produce a `CarToCode_Architecture_Review_Gemini_v2.md` doc with decisions and rationale, push to the repo (or paste to user → Claude Code commits).

### Claude Code

Waiting for:
1. The Gemini v2 doc to land in the repo
2. User confirmation that Android Studio is installed

Once both arrive, will:
- Encode the inbox schema as `schemas/inbox_v1.schema.json`
- Update `DECISIONS.md`, `BEHAVIORS.md`, `BUILDORDER.md`
- Move resolved items out of `DISCOVERY.md`
- Begin Phase 0 STT spike planning (Kotlin scaffolding for the Vosk prototype)

### User (Chris)

1. Install Android Studio on Linux box: `sudo snap install android-studio --classic` in PuTTY.
2. Take `CarToCode_Strategic_Review_Brief_v1.md` to Claude.ai + Gemini for collaborative review.
3. When that round produces a v2 decisions doc, drop it back into the repo (push directly, or paste to Claude Code on next session).

---

## User Profile

Chris is a PM — non-developer, AI-directing. Prefers:

- Voice-based discovery (cars are the natural environment)
- Keyboard work at the desktop
- One question at a time when walking through decisions — do not bundle
- No interactive commands (sudo / menus / passwords) via the `!` shell — those go to PuTTY
- Direct, terse responses — no fluff, no padding

Treat suggestions as starting points, not mandates. Chris redirects when needed.

---

## Maintenance Discipline

This file should be updated at the end of any working session that changes state:

- Update the **Quick State Snapshot** table
- Update the **Active Work** section
- Update **Last updated** timestamp at top
- Commit and push

If any AI reads this file and finds it stale, the first task is to update it before continuing.
