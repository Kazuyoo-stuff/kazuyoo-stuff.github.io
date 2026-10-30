# Changelog

## [v1.8-EOL-REV]
- Trigger changed from RAM usage % to PSI memory pressure (full/some avg10), with a MemAvailable fallback when PSI is not readable
- iOS-like jetsam logic: do nothing under normal pressure, compact under medium, selectively kill under high
- Enable the cached app freezer once at start (suspend-like behavior)
- Medium tier: send trim-memory (MODERATE) then compact cached apps, oldest first, max 5 per cycle
- High tier: kill cached apps one by one (oldest first), re-check pressure after each kill and stop as soon as it drops
- Swap-aware: escalate to kill when zRAM free space is low, since compacting into a full zRAM is useless
- Per-package compact cooldown (120s) to avoid repeatedly compacting the same apps
- Cached system apps are now targeted too; critical packages (launcher, IME, dialer, SystemUI, Bluetooth, NFC, etc.) stay whitelisted
- Root only (experimental): reclaim large native processes (via /proc/<pid>/reclaim or process_madvise), never killed; core daemons are skipped
- Root and non-root supported; root-only features disable themselves automatically
- Removed memory-factor override and kill-all (ignored the whitelist)
- Lower overhead: no foreground lookup, no dumpsys while the screen is off, hysteresis when pressure is normal
