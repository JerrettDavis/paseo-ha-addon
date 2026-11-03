## Why

Paseo 0.15.0 moves terminals into a separate terminal worker process that outlives daemon restarts. Before the worker reports ready, it checks its own process identity with `ps -p <pid> -ww -o lstart= -o args=` (and lists processes with `ps -A -ww -o pid=,ppid=,sid=,lstart=`), always under `LC_ALL=C`. The add-on image is Alpine, whose only `ps` is BusyBox: it has no `-p`, `-A`, `-ww` or `lstart` (`ps: unrecognized option: p`). With the Paseo 0.15.0 pin the worker therefore fails with "Terminal worker cannot read its own process identity: no usable snapshot", the daemon worker exits before readiness, and the add-on stops after six crashes in 300 s: the daemon never starts.

## What Changes

- `paseo/Dockerfile`: install Alpine's `procps-ng` in the runtime image (adds `ps`, `pgrep`/`pkill`, `top`, `free` and friends from procps-ng in place of the BusyBox applets), with a comment saying why.
- Side effect, checked: the service script's leftover cleanup (`pkill -9 -f '^Paseo (Supervisor|Daemon)$|@getpaseo/server/dist/(scripts/supervisor-entrypoint|server/daemon-worker)'` in `s6-rc.d/paseo/run`) now runs procps `pkill`, which matches the renamed `Paseo Supervisor`/`Paseo Daemon` processes as written. It does not match the terminal worker (`node $PASEO_HOME/terminal-worker/<version>-<hash>/terminal-worker.mjs --paseo-terminal-worker <id>`), so after a crash the restarted daemon takes the running terminals back. No change to the script is needed.
- `paseo/tests/smoke/run.sh`: new check f): the image's `ps` supports the procps options Paseo uses; a program started in a Paseo terminal keeps running across `paseo daemon restart` and across a crash of the supervisor and daemon (killed with SIGKILL, restarted by s6), and the new daemon lists its terminal and still shows its output.
- Spec delta for `agent-runtime`: new requirement "Terminal programs survive daemon restarts".

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `agent-runtime`: ADDED requirement "Terminal programs survive daemon restarts".

## Impact

- `paseo/Dockerfile` (shipped; released with the version bump of the `bump-paseo-0-15-0` change in the same PR), `paseo/tests/smoke/run.sh` (not shipped).
- Image size grows by about 1 MB (`procps-ng` and `libproc2`).
