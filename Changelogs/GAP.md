# Changelog

## [v1.2]
- Fixed report field `$budget` → `$budget_mb` and `$ram_free`/`$ram_total` → `$ram_avail_mb`/`$ram_mb` (were undefined variables in original).
- Added `Resident` field to prefetch report using `vmtouch -o kv` structured output instead of awk regex on human-readable text.
- Replaced redundant `-R` and `-Q` vmtouch passes with single `-t` (blocking mmap+touch) before `-dlw` lock, reducing vmtouch spawns from 3 to 2.
- Moved `ulimit -l unlimited` before the touch pass so mlock does not fail.
- Fixed `file_limit` integer truncation by replacing `(ram_gb * 12) + 12` with `(ram_mb * 12) / 1024 + 12` to avoid precision loss from integer division on devices reporting RAM below exact GB boundaries.
- Raised `file_limit` hard cap from 96 to 128.
- Removed duplicate scan paths: dropped `/data/data/$pkg` (alias of `/data/user/0`) and `/sdcard/Android/...` (alias of `/storage/emulated/0/Android/...`).
- Added `/data/user_de/0/$pkg` (Direct Boot storage, used by Unity on Android 7+).
- Replaced static data directory scan with dynamic install directory resolution via `pm path`, now scanning `oat/`, `lib/arm64-v8a/`, `lib/arm/`, `*.apk`, `*.dm`, `*.vdex`, `*.odex` as Priority 1.
- Removed `/storage/emulated/0/Android/data/$pkg` from scan entirely, replaced by install dir.
- Expanded ENGINE_NAMES with: `libmain.so`, `libunity.so`, `libUE4.so`, `libUE5.so`, `libgame.so`, `libcocos2dcpp.so`, `libCryEngine.so`, `libFMOD*.so`, `.obb`, `.xapk`, `main.obb`, `patch.obb`, `data.unity3d`, `boot.data`, `bundle.data`, `.bundle`, `.manifest`, `.ab`, `.assetbundle`, `.zippak`, `app_data.db`, `game_data.db`.
- ENGINE_NAMES grep now uses `-i` flag (case-insensitive) to catch variants like `GameAssembly.so`.
- Removed `cp LOGFILE LOGFILE.prev` from refresh loop (reverted to match latest shell version).
