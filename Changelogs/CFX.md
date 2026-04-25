# Changelogs
### [v2.3]
- Replaced `taskset -p ff` and `renice -n 19` shell calls with native `sched_setaffinity()` and `setpriority()` C syscalls
- Replaced `sleep 0.3` with `nanosleep()` for nanosecond precision
- Replaced `for t in /proc/$p/task/*` shell glob with native `opendir()` + `readdir()`
- Replaced `${diff#-}` shell abs value with ternary expression in C
- Daemon loop via native `fork()` with stdout/stderr redirected to `CFX_LOG`
- Added `scheduler_policy fifo 2` for persistent SCHED_FIFO priority from process spawn
- Added `writepid /dev/cpuset/foreground/tasks` for Android 12+ cpuset compatibility
- Removed `debug.sf.frame_rate_multiple_threshold 60` from CFX binary `surfaceflinger_autoset`
- Added `debug.sf.frame_rate_multiple_threshold=0` to system.prop
