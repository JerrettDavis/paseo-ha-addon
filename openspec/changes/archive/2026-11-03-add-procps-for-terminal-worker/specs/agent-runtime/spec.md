# Spec Delta

## ADDED Requirements

### Requirement: Terminal programs survive daemon restarts
The image SHALL provide the process tools that Paseo's terminal worker needs to verify process identities (procps `ps` with `-p`, `-A`, `-ww` and `-o lstart=,args=,sid=`), so that the daemon starts and terminal programs keep running when the daemon restarts or crashes inside a running add-on.

#### Scenario: Daemon starts with the terminal worker
- **WHEN** the add-on starts
- **THEN** the daemon launches its terminal worker and its health endpoint returns success

#### Scenario: Daemon restart keeps terminals
- **WHEN** a program is running in a Paseo terminal and the daemon is restarted with `paseo daemon restart`
- **THEN** the program is still running and the restarted daemon lists its terminal

#### Scenario: Daemon crash keeps terminals
- **WHEN** a program is running in a Paseo terminal and the daemon's supervisor and worker processes are killed
- **THEN** the add-on restarts the daemon without stopping the terminal worker, the program is still running and the new daemon lists its terminal

#### Scenario: Add-on restart closes terminals
- **WHEN** the add-on is restarted or updated
- **THEN** its terminals are closed, because the whole container restarts
