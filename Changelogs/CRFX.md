# Changelogs
### [v2.1]
- New parameter in MTK gpufreq v1/v2 tuning — max frequency lock, disabled thermal/OC/low-battery frequency limiting
- New value in PowerVR (Imagination) GPU parameter tuning via `pvrsrvkm` and apphint debug nodes
- New parameter in Adreno driver optimization — power level pinned to max, disabled vsync/fsync/throttling/adrenoboost, fixed clock to 1114MHz, disabled kgsl profiling and adreno idler
- Added Mali driver optimization — DVFS enable, clock range tuning, job scheduling set to `full` serialize mode
- Partial deletion in task scheduling optimization — renice SurfaceFlinger, HWUI, gpuservice, and composer to -20, moved to top-app cpuset group
- Added background cgroup demotion for `netd` and hardware media codec processes
- Added MSM performance module tuning — `gpu_oc`/`gpu_ov` enabled, `gpu_uc`/`gpu_uv` disabled, touch boost enabled
- Added SurfaceFlinger `core_graphics` device_config overrides — refresh rate consistency, commit-not-composited handling, reduced unnecessary trace logging
- Added ART `art_performance` device_config overrides — fast baseline compiler, register allocator spill slot reuse, app image startup cache
- Added ADPF `game` device_config overrides — GPU actual work duration reporting enabled, SurfaceFlinger GPU hint session enabled, power-efficiency preference disabled in favor of performance
