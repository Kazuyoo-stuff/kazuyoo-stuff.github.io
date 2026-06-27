# Changelog

## [v1.4]
- Switched preload flag from `-A`/`-R` to `-Q` for more aggressive combined fadvise + readahead prefetch
- Removed unstable `mlock`/`-d -l -w` daemon-lock mode — preloaded pages are now touch-only and remain evictable under RAM pressure
- Reduced `RAM_BUDGET_PCT` from 75% to 40% to prevent excessive memory pinning on lower-RAM devices
- Added `MAX_LOCK_MB` hard cap (300MB) as an absolute ceiling regardless of available RAM percentage
- Reduced `MAX_PARALLEL` from 3 to 2 to lower concurrent fork overhead during multi-game preload
- Increased `GAME_CHECK_INTERVAL` from 5s to 8s to reduce polling overhead from `cmd activity stack list`
- Added foreground-aware refresh loop — detects active game via foreground package check and pauses preload while a tracked game is running
- Added separate idle-mode and gaming-mode polling behavior within the daemon loop
- Removed `cleanup_stale_locks` routine — no longer needed since lock files are no longer created
- Renamed log field "Locked" to "Touched" to accurately reflect the new non-locking preload behavior
- Ported updated logic 1:1 into C binary (`GAP.c`) with equivalent priority handling via `sched_setaffinity`, `setpriority`, and `ioprio_set`
- Built two parallel variants for comparison testing — `GAP.sh`/`GAP.c` using `-R` (lightweight async readahead) and `GAP_Q.sh`/`GAP_Q.c` using `-Q` (combined fadvise + readahead, more aggressive)
