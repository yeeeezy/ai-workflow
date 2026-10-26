# FEAT193 Scan surface detail and contrast

Status: reverted at user request2026-09-27. Historical implementation below is inactive.

User approved surface detail, local contrast and reduced competing glow, explicitly retaining blur.

FEAT193 implemented2026-09-27: violet surface luminance detail/contrast pass before original grid composition, with original blur/chromatic filter unchanged byte-for-byte. Bounded four-neighbour luminance ratio; Surface-Detail1.0, Surface-Contrast1.22, Surface-Brightness1.4; parent neutral0/1/1. Competing grid alpha .2->.15 and boost20->12, Outline/Fresnel boost2->1.4; enemy Sparkle OutlineIntensity1000->650, source effect/RGB/width retained. Four material packages only. Live compile/save and shader-error check pass; real TestMap comparison with temporary station panels,10 current-exposure GAS/runtime/on-off checks pass; independent cold16parameter checks pass. Disk BP exposure0 discovered vs stale editor1.5: clean BP reloaded and final paired comparison at user0. BP and29total unaffected scan/BP packages preserve baseline hashes. No C++/map/filter/BP writes, no build required; PIE ended and dirty content/maps0. Evidence D:/UnrealWork/ScanDetailContrast, CurrentExposureBefore/After.png. Modest local contrast improvement; retained radial blur still softens peripheral fine detail. Four additional scene samples; no measured FPS claim. Checkpoint72717b16 and full33package backups permit rollback. Final changes uncommitted; no push/package/version change.

## Implementation and verification

One Custom node between original Desaturation0 and Multiply1.A uses existing distorted scene UV, four clamped4pixel neighbours, bounded luminance-ratio enhancement(.65..1.55), bounded tonal gain(.7..1.3), and deepest-shadow protection. Original hue/desaturation, scan mask, reverse, native lifecycle, audio and downstream dizziness unchanged. Three scalar parameters added under 0-Surface Detail, neutral parent allows existing postprocess fading. Original five changed graph nodes are Multiply1 and four vector alphas; four new expression nodes total,267nodes cold. Enemy parent/MI only scalar intensity change.

Before/After.png are initial same-camera passes at stale editor exposure1.5. Final CurrentExposureBefore/After.png use one paused PIE scene at disk exposure0; baseline is recreated with neutral Surface parameters, original glow alphas/intensity on transient MIDs only. Final settings restored on MIDs before after capture, then real GAS stops scan and cleans up. Temporary station panels face camera and retain original materials/normal maps, never saved. Material shader logs have no new compilation errors. Independent final commandlet exits0 with0errors/0warnings,16exact values checked.

Initial comparison probe used unavailable k2_get_vector_parameter_value accessor; corrected to read MIC via MaterialEditingLibrary, rerun passes10checks. No production failure left. Final cold values in cold.json/author.json. Preservation29/33unchanged confirms both Filter packages, BP, Niagara references and exposure untouched.

## Rollback and controls

Exact four changed package backups are listed in D:/UnrealWork/ScanDetailContrast/backup.json; restore only those packages while editor closed, respecting subsequent user edits. Checkpoint72717b16 preserves entire prior purple trial plus saved user exposure0. Do not replay older FEAT192 palette scripts over the current BP. MI_Infiltrator_ScanGrid parameters Surface-Detail1.0, Surface-Contrast1.22, Surface-Brightness1.4. Prior comparison scripts/evidence outside project; no production testing code added. No performance measurement.


## User-requested rollback

2026-09-27 user rejected FEAT193 and requested previous version. Restored exactly four grid/enemy-outline material packages from verified pre-FEAT193 backups after editor exited. All33 scan-material/BP baseline SHA256 hashes match, including unchanged original blur/filter and user BP exposure0. Independent cold load confirms Surface-* nodes/parameters absent, original glow alpha and enemy intensity1000 restored. Previous FEAT192 violet palette active. Safety checkpoint ed5532e2 preserves rejected detail version. No new PIE run; no C++/map/BP/filter changes, no push/package. Evidence D:/UnrealWork/ScanDetailContrast/rollback.json and rollback-cold.json. Editor left closed as found.
