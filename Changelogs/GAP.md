# Changelog

## [v1.6]

- Fixed cache ratio reporting by initially adding a native `mincore()`-based residency check (`cachecheck` helper) as a workaround, later replaced once the underlying vmtouch bug itself was found and fixed
- Removed `.apk` from the engine-tier file detection entirely (previously `install_dir`'s base APK could consume `file_limit`/`budget_mb` slots that are better spent on `lib`/`oat`/`.dm`/`.vdex`/`.odex`)
- Diagnosed why cache ratio always read exactly 100%: the measurement was taken immediately after the touch call on the same file list, which is tautological (touch guarantees residency at that instant) rather than reflecting real-world residency at game-launch time
- Added (then, per user request, removed again for simplicity) a `recheck_cache_ratio()` feature that re-measured residency of the previously preloaded file lists at the moment a tracked game actually launched, logging results to a separate non-rotating log file
- Reviewed and rejected a "global preload" variant proposed by the user containing:
  - A fatal bug: process substitution (`< <(...)`) which is not supported by Android's `/system/bin/sh`
  - A budget-per-fleet-of-games design instead of per-game budget, which could starve smaller games
  - `MIN_FILE_SIZE_MB=5` raised from the previous 512KB threshold, shown to exclude most real asset files for lighter games based on prior log data
- Reviewed and fixed a bug in a `get_fg_pkg()` rewrite where `refresh_loop` still called the old, now-undefined function name (`get_foreground_pkg`), silently breaking foreground-game detection entirely
- Fixed `get_fg_pkg()` to keep a fast-path query (`cmd activity stack info 1 0`) with fallback to `cmd activity stack list`, restoring a `visible=true` filter in the fallback path that had been dropped
- Extracted and reviewed the actual packaged Magisk module (zip) and diagnosed `build_game_list()` in `service.sh`:
  - Found `dumpsys game` output was treated as sufficient on its own, short-circuiting the comprehensive `pm list packages -3 | grep -Ff` scan whenever `dumpsys game` returned even one entry — since `dumpsys game` only reflects currently/recently active games, this caused only 1 game to ever be detected
  - Fixed by always running the comprehensive scan and unioning it with `dumpsys game`'s results instead of treating the latter as an early-exit condition


# vmtouch — motified
  
- Took the pristine, unmodified upstream source (Doug Hoyte, v1.3.1) as a clean baseline and confirmed the `stdout`-reopened-twice / missing-`stderr` bug and the `INT64_MAX`/`double` comparison warning both already existed upstream, not introduced by prior modifications
- Built a "high performance, stable, no added overhead" version from that clean baseline:
  - Added `posix_fadvise(POSIX_FADV_SEQUENTIAL | POSIX_FADV_WILLNEED)` before touch-mode mmaps
  - Added `MAP_POPULATE` for touch-mode mmaps, with fallback to a plain `mmap()` if it fails
  - Added `madvise(MADV_DONTDUMP)` on all mappings and `MADV_WILLNEED` for touch mode
  - Made the touch loop reuse the `mincore()` result that was already computed (no extra syscall) and only re-touch pages that are verifiably *not* resident yet — avoiding the double-work of touching pages `MAP_POPULATE` already faulted in, without blindly trusting that it always fully succeeds
  - Added a `__builtin_prefetch()` hint scoped only to pages that actually need touching
  - Applied the same heap-allocated `npath` fix, `reopen_all()` fix, and `parse_size()` cast fix as above
  - Verified with a full smoke test cycle: evict → cold verify → touch+verbose → hot verify → `-o kv` output → nested-directory recursion, all compiling with zero warnings
  
