# System Architecture Summary

For full detail, see `CarToCode_Architecture_Document_v4.md` and the revision document `CarToCode_Architecture_Review_Gemini_v1.md`. This file is the operational summary Claude Code consults during build.

---

## Components

- **Android Voice Bridge App** — primary built component. Handles voice interface, dual-model API calls, micro-batch GitHub sync, local backup, context loading, Bluetooth control handling.
- **GitHub Repository** — single source of truth. Inbox/Outbox single-writer pattern.
- **Claude Code (desktop)** — intelligence layer. Reads context, processes inbox, organizes content, executes development tasks autonomously per risk tier.
- **External services** — Claude API (Sonnet + Haiku), GitHub Gist or Repository Variable (heartbeat).

## Data Flow

```
[User voice] → [Vosk STT] → [Sonnet API] → [TextToSpeech]
                              ↓
                          [Haiku API] → [JSON validation] → [Local SQLite queue]
                                                            ↓
                                            [Micro-batch push to /inbox]
                                                            ↓
                                             [Claude Code reads /inbox]
                                                            ↓
                                  [Updates core files, commits, archives inbox]
```

## Single-Writer Discipline

| Writer | Writes to |
|--------|-----------|
| Android app | `/inbox/*.json` (only) |
| Claude Code | All core `.md` files, `/archive`, `/summaries`, `/tests` |
| Android app (state) | GitHub Gist or Repository Variable (heartbeat) — NOT this repo |

This eliminates Git concurrency conflicts by design.

## Key Constraints (load-bearing)

- Hands-free absolute requirement — if it can't be done while driving safely without touching the screen, it's not in the workflow.
- Phone independent of Linux box.
- Token minimization at every layer.
- Inbox/Outbox single-writer pattern.
- All credentials in Android Keystore.

## Headline Revisions from v4.0

See `DECISIONS.md` for the full revision log.

- STT engine: Vosk via STT Abstraction Layer (replaces native Android `SpeechRecognizer`).
- `SESSION_STATE` heartbeat moved out of repo (Gist / Repo Variable).
- Context polling and rolling summarization deferred from MVP.
- Fine-grained GitHub token replaces classic PAT.
- Latency target relaxed to 3–5 seconds.
- "Deaf Mode" added for multi-passenger driving.
- Refusal handling defined.
