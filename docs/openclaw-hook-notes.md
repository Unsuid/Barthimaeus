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
- Port `18789` appeared unavailable from the phone because the Gateway was bound to localhost (`127.0.0.1` / `::1`), unlike n8n which listened on LAN. Phone reachability is therefore not a reliable Gateway health check.
- The Gateway systemd user service was active after recovery and spawned Codex app-server children.
- OpenClaw docs say `Native hook relay unavailable` means the Codex-native tool hook could not reach an active OpenClaw relay through the local bridge or Gateway fallback.
- OpenClaw docs recommend `/new` or `/reset`; if it persists, restart Gateway so stale app-server threads and hook registrations are dropped.

Most likely cause:

The Gateway/app-server/native-hook-relay chain gets stale across Gateway update/restart cycles or long-running sessions. The Gateway can still be running and Telegram messaging can still work, while the Codex-native PreToolUse hook registration used by shell/Git/apply_patch is no longer reachable. Because PreToolUse fails closed, native tools are blocked with `Native hook relay unavailable`.

Supporting details:

- Journal showed repeated Gateway restarts around update/service events.
- A restart handoff file existed from `gateway-update` / `update.run`.
- Doctor found and repaired one stale Codex session route.
- Doctor warned the Gateway service is installed through an nvm-managed Node path, which can be brittle after updates.
- Current relay state lives under `/tmp/openclaw-native-hook-relays-1000/` and is tied to the current Gateway PID.

Fixes applied:

- Initialized and pushed `Unsuid/Barthimaeus` rescue repo.
- Ran `openclaw doctor --fix`; it repaired one Codex session route from legacy `openai-codex/*` routing to `openai/*` while preserving auth pins.

Next diagnosis window:

1. Run `openclaw doctor --non-interactive`.
2. Inspect OpenClaw/Gateway process model.
3. Locate logs mentioning hook, relay, gateway, native relay, or pretool.
4. Determine whether the Gateway is launched by a terminal, systemd user service, npm process, or OpenClaw manager.
5. Document a stable restart command.

Known stable restart path:

```bash
systemctl --user restart openclaw-gateway.service
```

Fresh-session path:

```text
/new
```

or

```text
/reset
```
