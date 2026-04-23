
- Merged `gms_appops_feature` and `gms_doze_featute` into a single unified `gms_optimize` function, eliminating duplicate iteration over the same package list
- Removed duplicate `appops set GET_USAGE_STATS ignore` that appeared twice in the appops block
- Moved `appops write-settings` and `appops read-settings` outside the loop so they are called once after all packages are processed instead of per-iteration
- Converted per-package loop from sequential `for` to `xargs -n1 -P3` for parallel execution across 3 workers simultaneously
- Fixed `&> /dev/null` to `>/dev/null 2>&1` for POSIX sh compatibility
- Moved variable declarations `GMS`, `GMS1`, `GMS2`, `sdk` to top-level scope with consistent indentation
- Separated `wait_until_boot_completed;acquire_lock` from single chained line into two separate lines
- Added `appops set CAPTURE_CONSENTLESS_BUGREPORT_ON_USERDEBUG_BUILD ignore`
- Changed `cmd activity set-standby-bucket` value from `50` to `restricted`
