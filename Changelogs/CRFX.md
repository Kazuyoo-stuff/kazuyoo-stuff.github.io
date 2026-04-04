# Changelogs
### [v1.9]
- Fixed `optimize_gpu_frequency` v2 — volt extraction variable from `$freq` to `$gpu_freq` which was undefined
- Fixed `optimize_gpu_temperature` — threshold check raised from `< 90000` to `< 95000` to match written value
- Fixed `change_task_nice` — removed redundant `renice +40` and `renice -19` before setting target value
- Fixed gamelist parsing — now skips commented `#` lines and empty lines
- Changed `optimize_mali_driver` — `dvfs_enable` from `0` to `1`, `min_clock` raised from `100000` to `350000`
- Added `mali1_dir` — `power_policy always_on` to prevent Mali from entering power-down state during burst
- Added `composer` to `change_task_nice -20` and `cpuset top-app`
- Added Adreno — `devfreq/governor performance`, `idle_timer=80`, `nap_allowed=0`
- Added FPSGO — `fpsgo_enable`, `rescue_enable_by_pid`, `boost_affinity=100`
