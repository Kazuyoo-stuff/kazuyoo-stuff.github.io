# Changelog

## [v1.1]
- Initial release — APK quickload per-package into page cache.
- Added native libs `.so` preload from `/lib/arm64-v8a/`.
- Added `.dm` dex metadata preload.
- Added `.bytes` disguised native libs preload (e.g. Mobile Legends comlibs).
- Added OBB files preload for large game assets.
- Added shallow scan for known paths — `comlibs`, `lib`, `libs`, `assets/comlibs`, `files/dragon2017/assets/comlibs`.
- Added launcher silent preload without log using vmtouch `-R`.
- Added UFW preload hint per-package via `cmd ufw settings set-preload-enable`.
- Added universal launcher detection via `pm list packages` keyword match.
- Added per-package preload report log with APK count, size, RAM, resident %.
- Added RAM budget check before preload — skip if budget below 128MB.
- Added parallel preload with `MAX_PARALLEL=3`.
- Added refresh cycle every 3600s with screen-off skip.
- Added `--status` command showing PID, games in list, last preload time.
- Sort files by size descending — largest files preloaded first.
- RAM budget set to 65% of available RAM.
- vmtouch_kzyo: added `-A` advise, `-R` readahead, `-Q` quick mode flags.
- vmtouch_kzyo: added `madvise SEQUENTIAL+WILLNEED` hint during `-t` touch mode.
- Game preload uses vmtouch `-Q -f` for most aggressive prefetch.
- Launcher preload uses vmtouch `-R` async readahead — lightweight, no log.
- Fixed `GAMELIST_FILE` path from `/data/local/tmp/gap_gamelist.txt` to `/data/local/tmp/gamelist.txt`.
- Fixed vmtouch launcher preload from `-R` to `-Q -l` to lock pages after prefetch.
- Fixed vmtouch game preload from `-Q -f` to `-Q -l -f` to properly lock pages in RAM.
- Fixed vmtouch `-Q -l` replaced with `-t -l` so pages are fully resident before locking, not async.
- Fixed initial `run_preload` now runs before `nice/ionice` is set so preload is not throttled by idle priority.
- Changed `MAX_PARALLEL` from `3` to `2` to reduce I/O contention on entry-level devices.
- Changed `RAM_BUDGET_PCT` from `65` to `75` to allocate more RAM for locked game assets.
