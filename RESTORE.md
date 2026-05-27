# Restore Guide

Use this repo to rebuild Barthimaeus on a replacement machine.

## First Steps

1. Install OpenClaw/Codex according to the current upstream instructions.
2. Clone this private repo.
3. Copy or merge the files in `identity/` into the new OpenClaw workspace, preserving any newer local runtime defaults.
4. Restore avatar files from `avatars/`.
5. Read `memory/MEMORY.safe.md` and merge only still-relevant memories into the new workspace memory.

## What This Repo Does Not Restore

Secrets and local integrations must be recreated manually:

- OpenAI/GitHub/API tokens
- Home Assistant token
- n8n token and workflows
- device credentials
- private profile details
- local service configuration

That is intentional. A rescue repo should make recovery possible, not make compromise convenient.
