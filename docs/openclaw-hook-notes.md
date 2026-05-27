# OpenClaw Hook Notes

Observed failure:

```text
Native hook relay unavailable
```

Impact:

- Blocks shell commands.
- Blocks Git commands.
- Blocks `apply_patch`.
- Blocks local diagnosis commands like `ps`, `find`, and `openclaw doctor`.

Working hypothesis:

The local OpenClaw PreToolUse hook depends on a native relay exposed by the Gateway/runtime. When that relay is unavailable, the hook fails closed and prevents tools from running. The target tools are usually not the cause; GitHub, SSH, and the repo can be healthy while the hook still blocks all access.

Evidence from 2026-05-27:

- `git ls-remote` failed while the hook was down.
- Local reads failed while the hook was down.
- After the runtime recovered, `git ls-remote` and shell commands worked again.
- n8n on the Steam Deck remained reachable from LAN, so the whole host was not down.
- Port `18789` for OpenClaw Gateway appeared unavailable from the phone, suggesting the Gateway/relay was down or only bound locally.

Next diagnosis window:

1. Run `openclaw doctor --non-interactive`.
2. Inspect OpenClaw/Gateway process model.
3. Locate logs mentioning hook, relay, gateway, native relay, or pretool.
4. Determine whether the Gateway is launched by a terminal, systemd user service, npm process, or OpenClaw manager.
5. Document a stable restart command.
