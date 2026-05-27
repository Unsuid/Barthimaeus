# Security Policy

This repository must remain safe to clone onto a new device without leaking Daniel's private data.

## Never Commit

- `.env` files or environment dumps
- API keys, tokens, cookies, passwords, recovery codes, SSH private keys
- Private profiles or contact dossiers
- Raw messages, emails, calendar exports, browser history, or session logs
- Home Assistant, n8n, Samsung TV, or other device configs with credentials
- Full copies of `/home/deck/.openclaw` or other local workspaces

## Allowed

- Curated identity/persona files
- Redacted operational notes
- Avatar and UI assets
- Restore instructions
- Safe project briefs and non-secret helper scripts

When unsure, leave it out and write a sanitized summary instead.
