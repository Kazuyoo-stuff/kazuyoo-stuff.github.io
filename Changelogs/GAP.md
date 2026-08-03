# Changelog

## [v1.5]

- Added debug instrumentation to diagnose persistent `Cache Ratio: 0/0`: raw `vmtouch` output logging, per-stage file-count logging, deferred cleanup of temp lists on suspicious results
- Received user-provided custom `vmtouch.c` fork; reviewed `-Q`/`-R`/`-t` option-validation logic
- Received a newer, restructured `GAP.c` (fork/exec-based, `locked_list`/`extra_list` split) and `vmtouch.c` from user
- Fixed a token-parsing bug in `measure_cache_ratio()`: the parser grabbed the *last* `/`-containing token (byte-size string) instead of the *first* (page-count ratio)
- Added fallback inclusion of the APK file into Tier 1 candidates, based on comparison with an older shell version that worked
- Determined the APK fallback did **not** fix the issue — `vmtouch` still reported `0/0` even for a single real APK file
- Fixed `run_capture()` to merge child stderr into the captured pipe (previously only stdout was captured, hiding `vmtouch` warnings)
- Root cause found: in `vmtouch.c`, `o_max_file_size` defaults to `SIZE_MAX`; casting it to `(int64_t)` overflows to `-1`, making the `len_of_file > (int64_t)o_max_file_size` check always true — every file, regardless of size, was skipped as "too large"
- Fixed the bug in `vmtouch.c` by comparing in the unsigned domain: `(uint64_t)len_of_file > (uint64_t)o_max_file_size`
- Verified the fix: `Cache Ratio (-Q)` now reports real percentages (confirmed via live device logs and a local reproduction test)
- Removed the APK fallback from `GAP.c` (no longer needed — real `.so`/`.oat` files are readable now that the underlying `vmtouch` bug is fixed)
- Merged the two-line `Cache Ratio (-Q)` / `Cache Ratio (-R)` report fields into a single `Cache Ratio` line (prefers locked/-Q value, falls back to readahead/-R, else `N/A`)
- Progressively removed all temporary debug logging added during root-cause investigation (per-stage file counts, raw vmtouch dumps, parsed-ratio confirmations), keeping only genuinely useful failure-path logs, then removed those too once confirmed stable
- Optimized `GAP.c` for resource usage without changing logic:
  - `read_pid_file()` rewritten from `FILE*`/`fscanf` to direct `open()`/`read()`/`strtol()`
  - added a matching lightweight `write_pid_file()`, replacing inline `fopen`/`fprintf`
  - `print_status()` deduplicated to reuse `read_pid_file()` instead of its own copy of the logic
  - `get_foreground_pkg()` changed from a 128KB `malloc`/`free` per call to a `static` buffer (called every few seconds for the life of the daemon)
  - `measure_cache_ratio()` changed from a 64KB `malloc`/`free` per call to a `static` buffer (safe — parallelism is via `fork()`, not threads)
- Optimized `vmtouch.c`'s `write_pidfile()`: fixed a latent bug where the `size_t` return-value check could never detect a write failure, and converted it to lightweight `open()`/`write()`
