# Changelog

## [v1.6]
- Fixed foreground app being included in background app compaction — `compact_bg_apps` now receives `fg_pkg` and skips it explicitly.
- Fixed empty `fg` variable causing invalid grep regex — base exclusion list built first, `fg` prepended only when non-empty.
- Fixed daemon not staying alive — replaced bare `&` with `trap '' HUP` + `disown` to properly detach from parent shell.
- Changed RAM thresholds from 60/70/80 to 65/75/85.
- Changed `kill-all` threshold from >70% to >75%.
- Changed sleep intervals to consistent integers in seconds.
- Added `--stop` command to cleanly kill daemon and remove PID file.
- Added `get_fg_pkg` fallback — tries `visible=true` first, falls back to `head -n1` if empty.
- Thanks to @Java_nih_deks for fix check pid on action.
