# Changelogs

### [EOL - REVISED-4]
- Reworked the lock mechanism from a permanent touch-file flag to an `mkdir` + script-hash check, so the optimization automatically re-applies whenever the script is updated instead of staying skipped forever
- Added a `trap` on `INT`/`TERM` during lock acquisition so an interrupted run doesn't leave a stale lock behind, with the lock only finalized via `commit_lock()` after the full optimization completes successfully
- Added `restrict_playstore()` to apply lightweight appops restrictions (background run, location, usage stats, wake lock) and standby bucket throttling to Google Play Store (`com.android.vending`)
- Added `am force-stop` for Play Store at the end of `restrict_playstore()` to free its RAM immediately, since it's rarely used
- Generalized the optimization status/CPU check script (`CHK_OPT`) to accept a package name and label as parameters, and added a second call for Play Store alongside Google Play Services
- Added a revert/undo script that restores both GMS and Play Store appops, standby bucket, jobscheduler, and Doze whitelist state back to default
