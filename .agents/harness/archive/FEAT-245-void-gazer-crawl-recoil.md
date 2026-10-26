# FEAT-245 Void Gazer crawl recoil

Status: done
Closed: 2026-10-06
User approved repair of residual crawl pelvis travel. Checkpoint60cf6889 preserves FEAT244 and diagnosis; user config/rifles/map actors excluded. Source and target four-clips backed up externally in D:/UnrealWork/VoidGazerRecoil/before. All animation correction in TMIIR, target finalized clips only.

Source helper RemoveCrawlTravel subtracts linear first-to-last pelvis translation over each cycle, including sub-cm depth drift for a closed positional seam. Recenters XY mean onto Idle reference; keeps depth mean, all rotations/scales and other tracks. External source helper Development Editor build4.10s succeeded; initial build blocked by still-open target Live Coding, then succeeded after graceful clean editor shutdown (dirty0). No target C++/Blueprint edits planned.

All four source clips validated at65samples: corrected pelvis first/last position exactly matches; all other local tracks including tentacles and pelvis rotation/scale unchanged to1e-4. AssetTools migrated only four with ignore_dependencies=True/OVERWRITE; SHA256 confirms remaining34VoidGazer packages unchanged. Cold target38packages loaded, four positional seams0cm. Runtime loop/look checks underway.

Actual transient TestMap four-direction2.6cycle runs passed8assertions, max frame-to-frame actor-relative pelvis steps2.18/2.19/1.82/1.68cm after transition. Original85-98cm wrap reset absent. Cycle counts derive elapsed gameplay time and clip duration (fixed1x playback); max-step guard covers every frame including actual boundaries. Initial instrumentation tried unavailable Python GetCurrentTime; replaced with elapsed-time/full-frame measurement, no production workaround.

Three-direction look/return/resume regression437frame/phase assertions passed. Left/Right/Back screenshots saved. Corrected crawl positional seam0cm in fresh-process target; cold validation0errors0warnings. No Blueprint/default/native changes in target, hence no new target compile required. Existing target compiled code unchanged.

Final editor26824, sole primary monitor, TestMap SIE temporary square crawl plus left/back/right/back glance demonstration. Existing Ledge lean/recovery retained, tentacle contact correction still deferred. All final assets saved; no map writes. Evidence/scripts D:/UnrealWork/VoidGazerRecoil, including external before copies, source-validation.json, cold-loops.json, runtime-loops.json, runtime.json and demo.py. Source TMIIR editor closed. Source helper changes compiled; original vendor animations untouched. Only four finalized asset hashes changed;34other scoped assets unchanged. Final changes uncommitted, no push/package/version change.

Final dirty content/maps0. Corrected external demo enum typo introduced by work-folder text replacement; restarted callback, gameplay assets unaffected. Final.png visually inspected.
