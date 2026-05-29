# Operational Rules and Configuration

Single source of truth for runtime parameters and Claude Code's behavioral defaults. The Android app reads parameters relevant to its operation at session start. Claude Code reads risk tiers and safe-commit patterns when processing the inbox.

---

## Timer Intervals

- **Micro-batch sync trigger:** 3 minutes OR 10 queued items, whichever comes first.
- **Heartbeat update:** Every 3 minutes (piggybacks on micro-batch push). Written to GitHub Gist / Repository Variable — NOT to this repo.
- **Heartbeat staleness threshold:** 12 minutes. If `SESSION_STATE` status is `active` but last heartbeat > 12 minutes, Claude Code assumes a crash and proceeds.
- **HTTP timeout (Claude API calls):** 15 seconds (strict).

## Sliding Window

- **Default window size for Haiku scribe:** 800 tokens.
- **Tunable:** yes — edit this file, app reads at session start.
- **Token source:** Claude API response usage data (no client-side tokenizer required).

## Model Defaults (Hardcoded for v1)

- **Primary conversation model:** Claude Sonnet (latest at app build time).
- **Scribe model:** Claude Haiku (latest at app build time).
- **In-app selector:** deferred to v2 per Gemini review.

## Latency Targets

- **End-to-end voice turn-around:** 3–5 seconds (median).
- **Soft alert:** anything beyond 5s should surface a TTS hint like "slow connection."

## Token Spend Guardrails

- **Per-session input-token cap (Sonnet + Haiku combined):** 250,000.
  - When hit, session cleanly ends with TTS notice. User can start a new session.
- **Monthly spend soft alert:** USD $100.
  - Alert mechanism: TBD (see `DISCOVERY.md`).
- **Monthly spend hard cutoff:** USD $200.
  - Sessions refuse to start until next month.

## Voice Commands

| Command | Effect |
|---------|--------|
| `"Ignore that last answer"` | Invalidates previous exchange: `{"action": "invalidate", "target_id": "..."}` |
| `"Pause"` | Halts agent and Claude Code activity: `pause_requested` |
| `"Revert"` | Requests Claude Code revert last commit: `revert_requested` |
| `"End session"` | Triggers final cleanup pass and closes session |

## "Deaf Mode" (Multi-Passenger Driving)

- **Toggle on/off:** double-tap steering-wheel Next-Track button. Pauses Vosk STT entirely.
- **Resume:** single tap on the same button.

## Phone Call Handling

- App relinquishes audio focus on incoming call.
- Session state preserved locally in SQLite.
- **No auto-resume** when the call ends — user must physically tap screen or press a media button to re-engage.

## Refusal Handling

- On Sonnet safety-filter trigger:
  - TTS: `"I can't answer that."`
  - Write to `/inbox`: `{"type": "refusal", "raw_prompt": "...", "timestamp": "..."}`
  - Continue session normally.

## Risk Tier Definitions

Claude Code classifies every task before executing.

### Low Risk — Auto-Commit to Main

- Documentation updates
- Adding/updating unit tests
- Typo or formatting fixes
- Dependency version bumps with passing tests
- Adding comments to existing code

### Medium Risk — Feature Branch, Await Review

- New feature implementation
- Refactoring existing code
- New API endpoints
- Database schema changes

### High Risk — Hold for Explicit Approval

- Architectural changes
- Security-related modifications
- Anything touching authentication or credentials
- Irreversible data operations

## Safe-Commit Patterns

- Every processed inbox batch produces a single commit with a summary message.
- Commit message format: `Process inbox batch <earliest-ts>..<latest-ts> (<N> entries) — <one-line summary>`
- Commit hash appended to `WORKLOG.md` entry for that batch (enables targeted `git revert` on Haiku semantic-failure recovery).
- Processed inbox files moved to `/archive` in the same commit.

## Credential & Token Management

- **Claude API key:** Android Keystore (hardware-backed encryption). Cloud-encrypted backup daily.
- **GitHub token:** Fine-grained personal access token scoped strictly to this repository (`arcscribe-engine`). Classic PATs are NOT used.
  - Token expiry: 1 year max (GitHub policy). Rotation procedure documented in `DISCOVERY.md` as an open item.
- **No credentials in repo, ever.**
