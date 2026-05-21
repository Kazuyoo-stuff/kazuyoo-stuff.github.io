# Changelog

## [v1.3]
**Core Preload Logic**
- Fixed `find -exec stat {} +` pipe subshell issue causing `raw_*.txt` to always be empty on Android sh
- Fixed `pm path` output not stripping `\r` causing APK path check to always fail
- Fixed `sort` tab separator not working on Android sh, switched to space-separated format with safe `awk` field stripping
- Fixed `for apk in $apk_paths` word-splitting corrupting multi-line APK paths, replaced with file-based iteration
- Added multi-directory scan covering `/data/user/0`, `/data/data`, `/storage/emulated/0/Android/obb`
- Added OBB file inclusion alongside APK in preload list

**vmtouch Command Sequence**
- Replaced `-t` with `-Q` / `-R` (device-dependent) for warm-up phase before lock
- Added `-f` flag across all vmtouch calls to follow symbolic links, fixing incomplete resident pages on devices with symlinked APK/OBB paths
- Removed `-d` (daemon mode) from lock phase — `-d` causes vmtouch to fork and exit immediately, making `resident` check always return N/A
- Lock now runs via background subshell `( ) &` with `ulimit -l unlimited` / `setrlimit(RLIMIT_MEMLOCK, RLIM_INFINITY)` — root cause of resident never reaching 100%
- Resident check moved to after warm-up and before background lock, giving accurate page cache reading
- Switched resident parsing from `-o kv` `ResidentPages=` / `Pages=` (wrong keys) to `-v` verbose `Resident Pages:` line

**SystemUI Preload**
- Replaced single `xargs -r vmtouch -dl -f -P` with proper 3-step sequence matching game preload: warm-up → resident check → background lock with `ulimit`

**Daemon**
- Fixed `refresh_loop` defined in parent script not accessible when spawned via `setsid sh -c`, replaced with `( trap '' HUP; refresh_loop ) &`
- Added smart refresh: daemon now detects foreground package and skips refresh cycle when a game is actively running

**Duplicate Execution Fix**
- Fixed `while (fgets(...) || line[0])` loop condition in C causing last package to execute twice on EOF
- Added in-memory seen-list dedup for gamelist processing (shell: variable string, C: array) — no temp files

**C Port**
- Replaced all hardcoded vmtouch path checks (`/system/bin/vmtouch`, etc.) with `command -v vmtouch` via `popen`, matching shell behavior and supporting any install path
- Replaced `taskset` / `renice` / `ionice` shell wrappers with native `sched_setaffinity`, `setpriority`, `syscall(SYS_ioprio_set)`, and `sched_setscheduler`
- Removed all `static` qualifiers from functions per project style
