# FEAT-168 Character functional layout and obsolete asset audit

Status: done. Closed 2026-09-26.

## Approved scope and result

User expanded the initial 51 Scan/Construction moves to the complete Infiltrator and Executive functional layout. Final roots are Gameplay, Lobby and _Shared. Main character Blueprints sit at Gameplay root; FirstPerson and Body each own their animations. Ability-specific GAS, audio and effects sit under Abilities/Scan or Blink. Drone Deployment and Turret/Construction own their resources. Only actually shared body packages use _Shared/Body. Native class directories and asset names were preserved. No retargeting occurred.

Unreal AssetTools completed 287 moves (123 Infiltrator,164 Executive), including39 shared body packages. Final inventory303 packages:136 Infiltrator,167 Executive. Captured material scalar/vector values match exactly after migration and cold restart. Native constructor paths, editor path guards, existing tests, audio validators, GameplayCueNotifyPaths and serialized external consumers were updated. Two external postprocess actor packages in SpaceStation and TestMap were resaved solely to update material references; no scene geometry or persistent test fixtures added. TestMap was not used for PIE.

Mapping: FEAT-168-asset-mapping.json. User review: FEAT-168-retained-assets.md. One confirmed obsolete bare scan wave deleted after registry/disk/code/config/script checks and null legacy sound properties.22 uncertain directly unreferenced assets and their dependency groups retained for the user's later decision. A duplicate scan NPC left by the interrupted first migration was consolidated;7 rename copies also consolidated. Scoped redirectors and101 empty obsolete directories removed.

## Verification

Evidence root: D:/UnrealWork/CharacterLayout.

- build.log: Development Editor Win64 Succeeded20.98s, no new compiler warnings.
- cold.json:1534 checks,287 moved targets, exact material values, valid generated classes and effect/audio bindings, no old aliases or scoped redirectors, no dirty packages. The sole dependency difference is removal of the preexisting nonexistent BP_Infiltrator_Old dependency during Blueprint compilation.
- scan-runtime.json:16 actual SpaceStation checks passed: black/blue smoke through walls, range and camera culling/reentry, lit/unlit silhouette alpha difference0, normal GAS close, smoke/audio/tick cleanup.
- drone-runtime.json:24 actual SpaceStation checks passed: E preview/LMB dispatch, survey before reveal, half-scale low-hover turret construction, combat gating, retained turret and returned following drone, cleanup/unpossess.
- lobby-runtime.json:9 checks passed for both roles, shared body loading and ready/relaxed poses.
- audio.json: both updated scan/drone audio manifests passed.
- Final fresh editor startup: no missing-package/object or Blueprint compile errors. BuildMiddle, OccludedSmoke and Lobby screenshots inspected.
- final-cleanup.json: no dirty maps/content, counts136/167. Owned test editor30352 exited; unrelated UE58Blank editor untouched.

## Resolved migration issues

The first partial migration left a duplicate Niagara parameter collection; consolidation preserved the source namespace/MPC. Native-loaded rename copies and live cached Gameplay Cue roots required fresh build/restart. MCP redirector fixup timed out, so its outcome was not trusted; exact external packages were saved through Unreal Python, referencers rechecked after registry update, then scoped redirectors removed. Recompiling scan materials/functions cleared historical cached MPC references without changing captured palette values; final cold startup is clean.

Early test attempts encountered background throttling and a collapsed viewport. The owned test editor's background throttle was disabled; the culling test used a target180 degrees behind the camera. No production performance parameters were changed for these tests. This feature makes no new FPS claim.

## Handoff

Safety checkpoints47c9b30,2fe8cc4,d4215fe protect earlier completed work and scoped pre-migration assets/source. Final changes remain uncommitted; no push, packaging or version increment (v0.6 retained).

FEAT167 remains user-paused. Its unbuilt candidate is parked at D:/UnrealWork/CharacterLayout/FEAT167-candidate.cpp. Verified47c9b30 16-iteration source was restored before this build; the directory task did not introduce the untested solver optimisation.
