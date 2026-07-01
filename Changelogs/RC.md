# Changelog

## [v1.7-EOL]
- Initial release — threshold-based RAM compaction daemon with screen-aware polling
- Added RAM usage calculation via `/proc/meminfo` — uses MemTotal, MemFree, Buffers, Cached, Shmem for accurate active memory percentage
- Added screen state detection via `debug.tracing.screen_state` with fallback to `dumpsys window displays`
- Added foreground package detection via `cmd activity stack list` with visible=true priority fallback
- Added background app detection — excludes foreground app, system services, launcher, input method, and common persistent apps
- Added `compact_bg_apps` — runs `cmd activity compact full` and `send-trim-memory COMPLETE` per background process
- Added four-tier threshold logic — CRITICAL (>85%), MODERATE (>75%), LOW (>65%), NORMAL (≤65%) with different actions and wait intervals per tier
- CRITICAL tier — `cmd activity compact system` + `cmd activity kill-all`, no per-app compact to avoid wasted cycles before kill
- MODERATE/LOW tier — per-app compact only, no kill-all to preserve user experience
- NORMAL tier — per-app compact with longest wait interval (60s)
- Added screen-off handling — skips all compaction and defaults to 60s sleep when screen is off
- Fixed missing `WAIT` fallback when screen is off — prevents undefined `sleep` behavior on first iteration
- Replaced `sleep 1m` with `sleep 60` for compatibility with toybox `sleep` on Android
- Daemon launched with `trap '' HUP` + `disown` for proper session detachment
