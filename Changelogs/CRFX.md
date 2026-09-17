# Changelogs

### [v2.3]
- Added dynamic thermal trip point ceiling (`GPU_TEMP_TRIP_CEILING=82000`) as a configurable variable, replacing the previous hardcoded `95000`, to keep software throttling as a functional early warning instead of deferring it until near the physical thermal limit
- Removed `force_no_nap` from the Adreno driver optimization path, since forcing the GPU to never enter light-idle between frames caused unbroken heat accumulation over long sessions, leading to abrupt hard-throttle drops instead of gradual scaling
- Added `optimize_game_thread()`: detects packages listed in the shared `gamelist.txt` (same source of truth used by CGO) and promotes their render-related threads (`RenderThread`, `UnityMain`, `GameThread`, and any thread matching `*RenderThread*`) to higher priority
- Converted `optimize_game_thread()` from a bounded retry loop (fixed `MAX_PID_WAIT` / `MAX_THREAD_SCAN` attempts) into a persistent daemon loop with no time limit, so game detection and thread promotion keep running for as long as the service is alive
- Introduced dual polling intervals inside the daemon loop: a longer `IDLE_POLL_INTERVAL` while waiting for a game to launch (resource-conscious), and a shorter `ACTIVE_SCAN_INTERVAL` while a game is running, to catch newly spawned render threads (e.g. after a scene/activity reload)
- Added self-priority throttling to the game-thread daemon itself (`taskset`, `renice -n 19`, `ionice -c3`) so the watcher process stays lightweight and doesn't compete with the game it's optimizing
- Added `chrt -f -p 1` (SCHED_FIFO) to the thread-promotion routine, on top of the existing `renice`/`taskset` calls, as an additional priority mechanism for detected render threads
- Reworked the game-thread daemon spawn logic to use a double-fork pattern (`( setsid sh "$0" --_game_thread & ) &`) so the daemon gets reparented to `init` (PPID 1), fully detaching it from the invoking shell's process tree — fixes the daemon being reaped when launched from short-lived shell sessions (e.g. QuickShell/adb one-shot invocations) that clean up their descendant process tree on exit
- Added post-spawn liveness verification: after starting the daemon, the script waits briefly and checks `kill -0` on the recorded PID, logging whether the daemon actually stayed alive instead of assuming success from a written PID file alone
- Added logging output for the game-thread daemon (`crfx_gamethread.log`) instead of discarding stdout/stderr, to make failures diagnosable
- Added `--stop` command to terminate the running game-thread daemon and clean up its PID file
- Added existing-instance check on `--execute` (via PID file + `kill -0`) to avoid spawning duplicate daemon instances on repeated `--execute` calls
