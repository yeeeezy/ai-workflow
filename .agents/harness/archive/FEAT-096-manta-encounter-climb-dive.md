# FEAT-096 Manta encounter climb and dive

Status: done. User requested source UE58Blank climb then descent after beacon.

- Checkpoint e331ad9 preserves FEAT-095. Source read-only. Reference Enemy/CoreMorph/CoreMorphBoss.cpp OrbitPosition/OrbitVelocity/PrepareDive/FlightPosition and CoreMorphTests intact-dive assertions. Source-extracted position math already exists in FCoreMorphFlightPath and is reused unchanged.
- New Entry GA cancels ambient flight without resetting Motion, uses source orbit from t3.55 aligned to live pose with1.5s velocity/clock blending, adapts ground target to saved landing. At release t13.4 hands off to existing Reassemble GA. Combat tree waits; cancelled/failed entry holds.
- Build/assets/PIE pending.

- Development Editor Win64 build succeeded (D:/UnrealWork/MantaEntry/build.log). BP_CoreMorphBoss compiled/saved with native EncounterEntry default ability; SpaceStation Station_Manta opts into ClimbAndDiveOnActivation. Author editor exited 0.
- Cold rendered PIE full sequence passed (pie.json): live patrol -> beacon -> +46087cm climb -> 23817cm intact descent -> reassembly -> Scorpion at saved landing and running combat, one ASC. No combat during entry. Gameplay completed and PIE ended; editor shutdown subsequently returned 0xC0000005 after log close (also seen in prior external Python automation). Separate lifecycle/active-entry exit checks pending.

- Lifecycle PIE passed after correcting external Python API names (no gameplay code fix): exact pose preserved on direct activation; cancel during climb clears flag and holds without combat restart; reactivation followed by lethal GE cancels entry and shuts down flight. Evidence lifecycle.json. Source reference hashes unchanged (source-unchanged.json).

- Active-entry PIE exit passed from a second patrol position (trigger at12s, exit9s into entrance); editor exit0, secondary monitor verified. All done criteria complete. No source project edits, permanent test fixtures, final commit or push.
- Harness check: FEAT-096 is unique across indexes; pre-existing FEAT-080 appears in both indexes and was left outside this feature scope.
