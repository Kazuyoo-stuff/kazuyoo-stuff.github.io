# Changelog

## [v1.3]
- Aligned get_game_list(), get_fg_pkg(), is_screen_on(), and notify() with the reference daemon's structure and re-fetch pattern.
- Replaced plain pidof process lookup with a more robust pgrep -x + /proc/*/cmdline fallback, matching the reference daemon's pid-detection approach.
- Added the missing --stop command.
- Fixed unreliable foreground-package detection that could return garbage (e.g. bounds=[...]) instead of the actual package name.
- Reduced notification latency by firing the notification right after the core boost settings, deferring the heavier background-app freeze step to run asynchronously.
- Added binder-thread priority boost (bt-inherit-rt, bt-skp-prio-restore) for the foreground game process to reduce UI/render stutter.
- Added a persistent com.android.systemui UI-thread and RenderThread boost, independent of game-mode state.
- Fixed a grep: no REGEX error caused by an empty foreground-package variable producing a malformed exclusion pattern.
- Fixed a ps: bad -p error by validating that a pid is strictly numeric before passing it to ps -AT -p.
- Made service shutdown graceful: sends a termination signal and waits before forcing, and now fully reverts all active boosts, static groups, and pins before exiting instead of leaving them applied.
- Added log rotation to cap the service log file size
- Added periodic re-checking of the systemui process id so the boost is automatically re-applied if systemui restarts.
