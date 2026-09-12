# Changelogs

### [v2.5]

- Replaced the old 4-anchor percentage/lerp duration curve (interpolated between 60/90/120/144Hz anchors) with a formula grounded in the device's own live `dumpsys SurfaceFlinger` values instead of guessed or borrowed device-tree constants
- Set SF duration to `vsync_period - 1ms`, matching both the AOSP fallback formula and the device's own measured value
- Set app duration to `1x vsync_period` (reverted from an earlier `2x period - 1ms` attempt that reduced missed frames but introduced noticeable input/scroll latency in daily use)
- Unified early / late / GL duration variants into a single value each (SF and app), matching the device's own dump which showed no distinction between them
- Set HWC min duration to `0`, matching the device's native value instead of a period-scaled floor
- Removed the legacy high-refresh-rate (`FPS > 60`) raw phase-offset property block, since the device's live dump showed no separate high-fps configuration path
- Left `debug.sf.phase_offset_threshold_for_next_vsync_ns` untouched (default), instead of forcing an N+2-vsync override that caused discrete pacing jumps
- Raised `debug.sf.set_idle_timer_ms` from a period-scaled 12–65ms formula to a fixed 1000ms, preventing hardware VSYNC from suspending/resuming during ordinary loading or menu transitions (previously caused a hitch exactly at scene changes)
- Raised `debug.sf.layer_caching_active_layer_timeout_ms` from a period-scaled 80–200ms formula to a fixed 3000ms, so transient/short-lived static layers (loading screens, menus) no longer get cached and then require a costly cache-invalidation right when the scene changes
- Added a 3-sample jitter/stability filter before re-applying tuning (previously re-tuned on any single vsync-period reading >1ms off, even from measurement noise, causing spurious re-tunes mid-session)
