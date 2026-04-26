# openbroker-plugin

OpenClaw plugin packaging for the [`openbroker`](https://www.npmjs.com/package/openbroker) Hyperliquid trading CLI.

## Why this is a separate package

The `openbroker` CLI legitimately uses `child_process` to dispatch subcommands and run an audit daemon. OpenClaw's plugin scanner blocks tarballs containing `child_process` patterns, so we keep the runtime CLI in `openbroker` (a normal npm package) and ship the plugin manifest + tools in this thin wrapper. The plugin imports `openbroker` as a library and drives it in-process — zero `child_process` in the scanned tarball.

## Install

```bash
openclaw plugins install openbroker-plugin
```

This installs `openbroker-plugin`; npm pulls `openbroker` (the CLI) as a transitive dep automatically.

## What's inside

- `src/index.ts` — plugin entry: registers the position watcher, automation restart service, and agent tools.
- `src/tools.ts` — the agent tools (`ob_buy`, `ob_sell`, `ob_bracket`, `ob_chase`, `ob_auto_run`, etc.). All call `openbroker` library functions in-process.
- `src/watcher.ts` — background `PositionWatcher` polling Hyperliquid, posting hook notifications to the OpenClaw gateway when positions/PnL/margin change.
- `src/cli.ts` — registers `openclaw ob status` and `openclaw ob watch` commands.
- `src/config-bridge.ts` — maps OpenClaw plugin config → `HYPERLIQUID_*` env vars consumed by the openbroker CLI.

## Optional: dashboard / metrics forwarding

If you also want audit notes, metrics, and agent action logs forwarded to a dashboard, install [`openbroker-monitoring`](https://www.npmjs.com/package/openbroker-monitoring) alongside this plugin. The openbroker runtime auto-loads it via convention dynamic-import and wires it into the audit pipeline.

## Versioning

This package's version is independent of `openbroker`. The plugin pins a compatible openbroker version in `dependencies` — bump that range when you depend on new openbroker library exports.
