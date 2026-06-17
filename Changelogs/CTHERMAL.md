# Changelogs
### [v1.5]
- Rewrote `stopped_thermal_process` to stop services via `init.svc.*thermal*` prop parsing first, then kill remaining matches only after verifying PID > 2 and `/proc/PID/exe` exists, avoiding blind kills of kernel threads or init
- Fixed `sed` pattern for vendor service name extraction (`s/init\.svc\.vendor\.//; s/init\.svc\.//`) for POSIX shell compatibility
- Rewrote `disable_thermal_protection` to use `mode → disabled` and raise `trip_point_*_temp` to 95000 instead of `chmod 000` on thermal zone files, avoiding kernel write errors from a protected sysfs path
- Fixed CPU core_ctl glob from `cpu[0,4,7]` (matched literal comma) to `cpu[047]`
- Consolidated the three separate service-stop methods (`stop`, `setprop ctl.stop`, MIUI `IMQSNative` service call) into a single loop over services found via init file scan
- Replaced unbounded `find /sys/ -name enabled` scan with 4 direct msm_thermal paths, removing a full filesystem tree walk
- Merged 3 separate `find` passes over `/proc/sys/` and `/sys/` (panic/debug flags, printk, printk_devkmsg) into a single pass with conditional value handling based on target filename
- Added `grep -E 'thermal'` pre-filter in `stopped_thermal_prop` to skip non-thermal getprop lines, cutting subshell spawns per iteration
- No changes to GPU-related logic, charging-related logic, or values — all changes scoped to thermal process/prop handling and script performance
