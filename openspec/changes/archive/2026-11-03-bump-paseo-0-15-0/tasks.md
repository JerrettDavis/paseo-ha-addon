# Tasks

## 1. Bump
- [x] 1.1 Pin Paseo 0.15.0 in paseo/build.yaml and both ARG lines of paseo/Dockerfile; add-on version 0.15.0-1; CHANGELOG entry
- [x] 1.2 paseo-anchors.sh --check and check-providers.sh pass for 0.15.0; --diff unchanged

## 2. Verification
- [x] 2.1 docs test, shellcheck, YAML parse
- [x] 2.2 docker build; smoke, direct-port and providers suites pass
- [x] 2.3 agent-config suite passes
- [x] 2.4 In the built image, a terminal created through the daemon keeps running across an s6 restart of the paseo service
