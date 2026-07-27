# Changelog

## [v1.5]
- Fixed missing function closures and undefined KEYWORDS/PROCESS array references that previously prevented the script from executing at all
- Fixed the dispatch bug where --execute called disable_logging with disable_logging_root passed as a string argument instead of actually invoking disable_logging_root
- Added KEYWORDS (process-name fragments to kill) and PROCESS (dropbox tags to demote) arrays, later extended with tombstoned, dumpstate, bugreport, diag, statsd, and additional AOSP dropbox tags (SYSTEM_BOOT, SYSTEM_RESTART, system_app_wtf, system_server_lowmem, data_app_anr, data_app_native_crash, data_app_crash, lowmem)
- Expanded the device_config keyword filter with diagnostic, verbose, audit, statsd, bugreport, eventlog, and perfetto
- Added disable_logging_root() for kernel/proc-level log and trace disabling, cache wiping, and service stopping
- Added cache clearing for /data/data, /data/user_de, and /sdcard/Android/data, plus pm trim-caches
- Restored the device_config blanket-disable loop and removed the redundant per-flag cmd device_config get call
- Fixed the kernel tracing loop to write the final value once per file instead of 101 sequential writes
- Backgrounded the per-app pm/dropbox loop and device_config scan so module install no longer blocks on hundreds of process spawns
- Optimized the post-fs-data debug-disable script from ~30 repeated full /sys and /proc scans down to a single combined find pass
- Switched logcat -c to logcat -b all -c to clear every log buffer, and added logcat -G 16K -b all to shrink buffer sizes
- Added dumpsys SurfaceFlinger --latency-clear
- Added battery stats silencing via battery_stats_constants and dumpsys batterystats disable full-history/no-auto-reset/pretend-screen-off
- Made the dropbox_max_files setting an unconditional put instead of a guarded one
- Updated the license header to GNU GPL v3.0 or later with 2026 copyright notice
- Added the global_setting_exists() guard in the C port so settings put global calls only run when the key already exists, matching the shell's grep -q pattern
- Ported the entire module to a C binary (celestial_disable_log_kzyo) with matching functionality and daemon-based background detachment
