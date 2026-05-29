# Post-MVP Feature List

**Last updated:** 2026-05-28

---

## Deferred from v1.0 MVP

### Per Gemini review

- **Context-Refresh Polling.** Phone polls GitHub every N minutes for changes to core context files; injects updates into Sonnet's context if Claude Code has modified them. MVP assumption: user is driving, not having someone code concurrently at desktop.
- **Rolling Haiku Summarization.** Every 15 minutes, Haiku summarizes older transcript portions to keep Sonnet's context window bounded. MVP behavior instead: session cleanly ends when token cap is hit.
- **Model Selector UI.** Settings screen with primary/scribe dropdowns persisting in Android SharedPreferences. MVP uses hardcoded Sonnet + Haiku.

### Per Claude Code review

- **Backpressure Handling.** Pause conversation if Haiku queue exceeds 5 exchanges behind Sonnet.
- **"Any updates from the desktop?" Voice Command.** Explicit injection trigger (covered implicitly by polling in v1 → not strictly needed once polling exists).

## Original Future Features (from v4.0 Section 12)

- Support for additional LLM providers (Gemini, GPT-4o) via provider abstraction layer.
- Jira integration alongside GitHub Issues.
- Multi-user support with proper access controls.
- Full cloud agent deployment with GitHub Actions.
- Session history and replay within the Android app.
- Push notifications when Claude Code completes tasks.
- Dashboard UI for monitoring Claude Code activity from phone.
- iOS version of the Android app.
- Voice Bridge packaged as standalone app for other users.
- Custom voice profiles for TTS.
