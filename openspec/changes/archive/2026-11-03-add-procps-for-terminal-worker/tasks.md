# Tasks

## 1. Image
- [x] 1.1 Install `procps-ng` in the runtime stage of paseo/Dockerfile, with a comment naming the `ps` options Paseo needs
- [x] 1.2 Confirm the cause: BusyBox `ps -p 1 -ww -o lstart= -o args=` fails (`unrecognized option: p`) and the 0.15.0 daemon crash-loops with "Terminal worker cannot read its own process identity"; with procps-ng the command works and the daemon starts
- [x] 1.3 Confirm the s6 service script's `pkill` pattern matches `Paseo Supervisor`/`Paseo Daemon` but not the terminal worker

## 2. Tests
- [x] 2.1 Smoke check f): `ps` supports the procps options; a terminal program survives `paseo daemon restart` and a SIGKILL crash of the supervisor and daemon, and the new daemon lists the terminal and shows its output
- [x] 2.2 shellcheck the smoke suite; smoke suite passes on the built image

## 3. Documentation
- [x] 3.1 paseo/DOCS.md: terminals keep running across a daemon restart or crash; an add-on restart or update closes them
