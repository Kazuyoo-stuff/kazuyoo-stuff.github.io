# Changelogs
### [v2.4]

**Core Architecture**
- Adaptive poll interval: 0.3s when unstable → 5s when stable after STABLE_THRESHOLD consecutive polls
- Screen state guard: reset `last_ft` on screen off so daemon reoptimizes fresh on wake
- Adaptive settle after rate change: rate DOWN (game launch) 0.15s fast settle, rate UP (game exit) 0.4s normal settle
- Thrash protection: >6 consecutive transitions → minimal 0.1s settle + reset counter
- Hard vsync period filter: only accept values within device range (FT_MIN=7.9ms, FT_MAX=17.5ms)
- FT_CHANGE_THRESHOLD 1ms: prevent false triggers from measurement noise

**Offset Calculation**
- Separate refresh rate buckets for 144/120/90/60Hz with individually tuned values per tier
- Consistent GPU window principle: (late% - sf_l%) × ft = 50% ft across all rates
- SF compositor durations calculated directly from ft, not derived from early/late values
- Min early-late gap enforcement at 8% ft: prevent SF scheduling collision
- Clamp early [90%, 115%] and late [78%, 93%] of ft
- `phase_thresh = ft × 110%` > ft: SF always skips to next vsync → gets full composition time
- Per-bucket `base_const` for proportional idle timer at each Hz tier
- Layer caching timeout: 8 frames in ms + 60ms base, capped [80, 200] ms

**Bug Fixes**
- Fixed phase_gap unit mismatch: old code divided by 1000 (producing µs) then compared against dimensionless value 1-4 → net effect was zero. Now operates purely in nanoseconds
- Fixed settle direction bug: other versions compared `ft > last_ft` after `last_ft = ft` was already updated → always false. Fix: save `prev_ft` before updating `last_ft`
- Fixed `transition_animation_scale` 1.1 → 0.9: transitions were 10% slower than default
- Fixed `frame_rate_multiple_threshold` from hardcoded 60 to dynamic per bucket (60/90/120/144)
- Fixed `phase_thresh = (early + late) / 2`: this caused SF to aggressively attempt current vsync it could not make, increasing miss count. Reverted to `ft × 110%`
- Fixed SF `early.sf.duration` and `late.sf.duration` previously derived from early/late (unstable) → now derived directly from ft (stable and independent)

**Performance & Overhead Reduction**
- Removed `get_static_data` entirely: App Phase = SF Phase always on this device → correction always 0 → eliminated full dumpsys call (~200-500ms) on every rate change
- Batched `surfaceflinger_autoset` in C binary: from 13-19 individual `system()` calls (~520ms) → 2 batched `system()` calls (~40-80ms) → fixes 0.5 second freeze on game launch
- Batched `apply_static_tweaks` in C: from ~10 individual system() → 2 batch calls
- Batched `tune_sf_tasks` in C: all per-thread `settaskprofile` calls in a single `system()` invocation
- Polling frequency reduced: from 288,000 dumpsys calls/day (constant 0.3s) → 17,280/day (5s when stable)
- `get_static_data` moved from inside rate change handler to once at daemon startup (later removed entirely)

**SurfaceFlinger Props**

- `debug.sf.use_phase_offsets_as_durations 1`: interpret phase offsets as durations
- `debug.sf.auto_latch_unsignaled true`: smart latching based on fence signal prediction
- `debug.sf.latch_unsignaled 1`: SF latches frames without waiting for GPU fence completion
- `debug.sf.disable_backpressure 1`: SF does not stall waiting for GPU fence → reduces missed vsync
- `latch_unsignaled` toggled between 0 and 1 across iterations based on tradeoff between missed frame count and visual quality (text texture artifacts during scroll)
- `commit_not_composited true`, `cache_when_source_crop_layer_only_moved true`, `use_known_refresh_rate_for_fps_consistency true`, `add_sf_skipped_frames_to_trace false`
