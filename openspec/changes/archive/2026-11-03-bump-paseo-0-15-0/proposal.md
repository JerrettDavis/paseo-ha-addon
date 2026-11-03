## Why

Paseo 0.15.0 is out (stable, 2026-11-03). Daemon restarts and crashes now keep terminal programs running (terminals move into a separate terminal worker process); an opt-in setting resumes interrupted agents in their existing conversations after a host restart; usage is fetched only while displayed, with a host setting to turn usage checks off; new worktrees can copy ignored files listed in `.worktreeinclude`; the daemon honours `HTTP_PROXY`/`HTTPS_PROXY`/`NO_PROXY`; plus many fixes (rewinding Claude conversations, Pi tool-result images, Codex history paging, relay frames for large responses). The add-on bundles 0.14.0, so users only get these once the pin moves. Pi 1.5.0 is the latest stable Pi (the add-on installs the latest Pi at build time); HA CLI 5.6.0 and gh 2.103.3 are already current.

## What Changes

- Pin Paseo 0.15.0 in `paseo/build.yaml` and both `ARG PASEO_VERSION` lines of `paseo/Dockerfile`; add-on version `0.15.0-1` in `paseo/config.yaml`.
- CHANGELOG entry naming the bundled Paseo **0.15.0** and Pi **1.5.0**, mentioning the procps addition from `add-procps-for-terminal-worker`, and noting that restarting or updating the add-on still closes terminals (the container restarts) and that going back to an older add-on version needs the pre-update backup (Paseo 0.15.0 saves agent-resume and usage-check settings in `config.json`).

The bump itself does not change the add-on's specified behaviour (nothing under `paseo/rootfs/` changes; anchors and providers checked below). Paseo 0.15.0 does need one image change to start at all: its terminal worker runs procps `ps` options that BusyBox lacks. That fix is its own change in the same PR, `add-procps-for-terminal-worker` (installs `procps-ng`, smoke check f), `agent-runtime` spec delta).

- `paseo-anchors.sh --diff 0.14.0 0.15.0`: anchors unchanged. `paseo-anchors.sh --check 0.15.0`: all ingress anchors present.
- `check-providers.sh 0.15.0`: built-in `claude codex copilot omp opencode pi`, plugin `antigravity hoplite muse`, hoplite excluded; the providers option matches.
- `npm pack` diff of `@getpaseo/server` 0.14.0 → 0.15.0: `dist/server/server/web-ui.js` is byte-identical; `web-ui/index.html` only changes the entry bundle hash in its root-absolute `<script src="/_expo/static/js/web/index-<hash>.js">` (rewritten by nginx `sub_filter '"/_expo/'`). The server's set of `PASEO_*` environment variables only gains `PASEO_OK`, `PASEO_SKILL_OK`, `PASEO_TERMINAL_WORKER_PROTOCOL_OVERRIDE` and four `PASEO_OPENCODE_*` names (OpenCode v2 service reuse); every variable the add-on sets is still read. The persisted config schema only gains `daemon.usage.enabled`, `agents.resumeOnStartup` and `agents.resumePrompt`; the add-on writes only `agents.providers` and `worktrees.root` through `paseo daemon config`, so it never overrides the new settings.
- `@getpaseo/cli` 0.14.0 → 0.15.0 changes `daemon stop` (new `--keep-terminals`, rejected together with `--force`), `daemon restart` (new `--kill-terminals`) and `daemon status` (an optional `terminals` field). The add-on's service script calls `paseo daemon status --json` (reads `.localDaemon`, unchanged) and `paseo daemon stop --force --timeout 15` (still valid; it closes terminals, which is what a leftover-instance cleanup wants). Its `pkill` pattern (`^Paseo (Supervisor|Daemon)$` and the supervisor/daemon-worker script paths) does not match the new terminal worker (`$PASEO_HOME/terminal-worker/<version>-<hash>/terminal-worker.mjs --paseo-terminal-worker <id>`), so when s6 restarts a crashed daemon the new daemon takes the running terminals back, as upstream intends. The worker copies its bundle and node-pty into `$PASEO_HOME/terminal-worker/` (under `/data`) and prunes copies no worker uses at daemon start.
- Release notes: no breaking, removed or changed entries. The three upgrade notes were checked against the add-on:
  - "The first upgrade from 0.14.0 still closes terminals": an add-on update or restart replaces or restarts the container, so terminals always close then; terminal preservation applies to daemon restarts inside a running add-on. Covered by the CHANGELOG note.
  - "Agent recovery is off by default": a user setting in Host settings → Agents, stored in `config.json` under `/data`; the add-on does not set or override it.
  - "Before downgrading to 0.14.0, restore a backed-up `config.json`": covered by the CHANGELOG note (Home Assistant's pre-update add-on backup holds `/data`).

## Capabilities

### New Capabilities

None.

### Modified Capabilities

None (pin-only bump, `skip_specs`, AGENTS.md Rule 1.4).

## Impact

- `paseo/build.yaml`, `paseo/Dockerfile`, `paseo/config.yaml`, `paseo/CHANGELOG.md`.
- Published images `ghcr.io/farajfarook/{amd64,aarch64}-addon-paseo:0.15.0-1`.
