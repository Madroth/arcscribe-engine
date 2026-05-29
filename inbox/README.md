# /inbox

Android Voice Bridge app writes timestamped JSON sync files here.

**Single writer:** the phone. Claude Code only **reads** from this folder, then moves files to `/archive` after processing.

File naming convention: `<ISO8601-timestamp>_<batch-id>.json`.
