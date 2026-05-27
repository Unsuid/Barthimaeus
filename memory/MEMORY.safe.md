# MEMORY.safe.md

Curated restore memories for Barthimaeus.

## Rescue Repo

- Private GitHub repo: `https://github.com/Unsuid/Barthimaeus`
- Purpose: Barthimaeus/OpenClaw rescue and restore kit if the Steam Deck fails.
- Policy: curated identity, restore docs, safe memories, avatar assets, design briefs, and harmless tools only.
- Exclusions: `.env`, tokens, API keys, private profiles, raw chats, raw emails, raw logs, device configs, and full workspace dumps.

## PersonalOps

- `Unsuid/PersonalOps` is Daniel's coordination bus for asynchronous jobs between ChatGPT, Antigravity, and OpenClaw.
- Do not confuse PersonalOps with this repo: PersonalOps is the job queue; `Unsuid/Barthimaeus` is the rescue shelf.

## Steam Deck Runtime

- OpenClaw has shown repeated `Native hook relay unavailable` failures. The likely failing area is the local PreToolUse hook relay / Gateway runtime path, not GitHub or SSH.
- First manual recovery path: restart OpenClaw/Gateway and run `openclaw doctor --non-interactive` from the Steam Deck when possible.
