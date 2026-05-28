# OpenClaw Native Hook Relay Hotfix

Date: 2026-05-28
Host: Daniel's Steam Deck
OpenClaw version observed: 2026.5.26

## Symptom

Native Codex tools intermittently fail with:

```text
Command blocked by PreToolUse hook: Native hook relay unavailable.
```

The Gateway itself can still be healthy at the same time:

- `openclaw-gateway.service` is `active/running`
- Gateway HTTP health is `200`
- Telegram/OpenClaw dynamic tools may still work
- Native shell/file tools are blocked

Overnight failures were seen around:

- 2026-05-28 00:48
- 2026-05-28 01:47
- 2026-05-28 02:48
- 2026-05-28 03:47
- 2026-05-28 04:48
- 2026-05-28 05:47
- 2026-05-28 06:40/06:41 during manual investigation

Diagnostic snapshots were written under:

```text
/home/deck/.openclaw/logs/gateway-diagnostics/
/home/deck/.openclaw/logs/gateway-diagnostic-last.json
```

## Likely Root Cause

The native hook relay registration includes a per-turn `generation`.
The Codex app-server appears to reuse a Codex thread between turns, and that
thread can keep an older hook command/configuration containing an old
`--generation`.

When a later native tool call uses the stale generation, OpenClaw rejects the
bridge request as stale registration and surfaces it as the generic error:

```text
Native hook relay unavailable
```

In short:

```text
per-turn native hook relay generation
+ reused Codex app-server thread
+ old hook command still in that thread
= stale generation rejection on next native tool call
```

## Files Involved

Installed OpenClaw dist file patched locally:

```text
/home/deck/.nvm/versions/node/v22.22.3/lib/node_modules/openclaw/dist/native-hook-relay-AN6S_wz5.js
```

Related CLI file inspected, but not patched:

```text
/home/deck/.nvm/versions/node/v22.22.3/lib/node_modules/openclaw/dist/hooks-cli-Cy_Rqd6n.js
```

Related Codex plugin files inspected:

```text
/home/deck/.openclaw/npm/node_modules/@openclaw/codex/dist/run-attempt-D8Vxo-Jm.js
/home/deck/.openclaw/npm/node_modules/@openclaw/codex/dist/vision-tools-DqpLmF5H.js
```

No clean config switch was found to disable thread reuse or shared app-server
behavior. Observed Codex plugin config:

```json
{
  "enabled": true,
  "config": {
    "codexDynamicToolsLoading": "searchable",
    "codexDynamicToolsExclude": []
  }
}
```

## Backup

Before patching, backups were saved here:

```text
/home/deck/.openclaw/backups/openclaw-hotfix-2026-05-28/native-hook-relay-AN6S_wz5.js.before-generation-hotfix
/home/deck/.openclaw/backups/openclaw-hotfix-2026-05-28/hooks-cli-Cy_Rqd6n.js.before-generation-hotfix
```

Original checksums:

```text
8fe8887423607f1d5f67912a0b0c5824653a652cebf6794d79d0251a89a61e11  native-hook-relay-AN6S_wz5.js
0a1dca01fb570d98bf2d1d847f7aac86da908d64fb187e96e63a40ab9ea6698c  hooks-cli-Cy_Rqd6n.js
```

Patched checksum:

```text
364fd1d592d6a54801408b5aa1fb9fa01431d538f9cee28125868bab0d1ccf0d  native-hook-relay-AN6S_wz5.js
```

## Patch Applied

Only one line was changed in `handleNativeHookRelayBridgeRequest`.

```diff
 result: await invokeNativeHookRelay({
   ...payload,
-  requireGeneration: true
+  requireGeneration: false
 })
```

At the time of patching this was line 422:

```text
/home/deck/.nvm/versions/node/v22.22.3/lib/node_modules/openclaw/dist/native-hook-relay-AN6S_wz5.js:422
```

This keeps the bridge/provider/relay checks in place, but stops rejecting an
otherwise valid local relay request solely because the reused Codex thread still
has an older generation value.

## Restart

After patching, restart the Gateway so the running app-server loads the patched
dist file:

```bash
systemctl --user restart openclaw-gateway.service
```

Observed successful restart:

```text
openclaw-gateway.service active since 2026-05-28 06:52:26 CEST
MainPID: 293377
```

## Verification

Check service and patch:

```bash
systemctl --user is-active openclaw-gateway.service
systemctl --user show openclaw-gateway.service -p ActiveEnterTimestamp -p MainPID --value
grep -n "requireGeneration: false" /home/deck/.nvm/versions/node/v22.22.3/lib/node_modules/openclaw/dist/native-hook-relay-AN6S_wz5.js
```

Run any simple native shell command through Codex. If it succeeds, the native
tool path is at least currently functional.

Then watch for recurrence around the next heartbeat/turn:

```bash
journalctl --user -u openclaw-gateway.service --since '2026-05-28 06:52:20' --no-pager -o short-iso
```

Search for:

```text
Native hook relay unavailable
native hook relay bridge stale registration
hook relay stale before restart
```

## Rollback

Restore the backed-up dist file and restart the Gateway:

```bash
cp -a /home/deck/.openclaw/backups/openclaw-hotfix-2026-05-28/native-hook-relay-AN6S_wz5.js.before-generation-hotfix \
  /home/deck/.nvm/versions/node/v22.22.3/lib/node_modules/openclaw/dist/native-hook-relay-AN6S_wz5.js
systemctl --user restart openclaw-gateway.service
```

## Caveats

This is a local installed-package hotfix. It may be overwritten by an OpenClaw
update or reinstall. The cleaner upstream fix would be for OpenClaw/Codex to
ensure the hook command is refreshed for reused Codex threads, or to make relay
generation handling stable across thread reuse.
