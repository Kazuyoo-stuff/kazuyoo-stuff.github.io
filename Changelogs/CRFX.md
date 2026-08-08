# Changelogs

### [v2.2]
- Added a `fix_target_opp_index` write ("00") inside the `GPUF_PATH2` block of `optimize_gpu_frequency()`, locking the GPU to its highest-performance OPP index after the freq/volt pin, to prevent governor oscillation between OPP states
- Merged a separate set of `device_config` overrides into a new `optimize_device_config()` function, covering `windowing_frontend`, `display_manager`, `systemui`, `backstage_power`, `statsd`, `art_performance`, `interaction_jank_monitor`/`latency_tracker`, `input`, `runtime_native_boot`, `window_surfaces`, `system_performance`, `windowing_tools`, and `game`/ADPF namespaces
- Resolved a conflicting `adpf_hwui_gpu` value between two source scripts by standardizing on `true` (GPU-accelerated UI rendering) for consistency with the performance-oriented direction of the merge
- Prefixed all `log()` output with `[FlowX]` for clearer identification in logs
- Added `2>/dev/null` suppression to the `umount` call inside `mask_val()` to avoid noisy errors on paths that aren't currently mounted
- General formatting, indentation, and comment consistency cleanup across all functions
- Fixed inverted default_pwrlevel in Adreno tuning that pinned the GPU to its lowest power state by default, causing ramp-up stutter under load
- Guarded the Adreno 610 hardcoded clock override (1114800000) to only apply when the frequency actually exists in the device's OPP table, preventing conflicts with other Adreno GPUs
- Scoped system_server cpuset assignment to render-critical threads only (display/anim/UI/vsync/input) instead of moving the entire process into top-app, reducing CPU contention with foreground app threads
- Removed the no-op hwui nice priority call, since hwui runs as in-app threads and never matches a top-level process
- Changed Mali serialize_jobs from full to none, since full is a kbase debug flag that disables GPU job-slot parallelism rather than a performance tweak
- Replaced hardcoded Mali max_clock/min_clock values with dynamic detection from the device's available_frequencies table
- and other setprop additions
