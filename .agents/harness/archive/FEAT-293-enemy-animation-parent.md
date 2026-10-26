# FEAT-293 Shared Enemy Animation Blueprint C++ parent

## Scope

Unify the native animation data contract for Enemy AnimBPs while keeping each species' authored pose graph. No retargeting in TheManTest.

## 2026-10-14 implementation

- Before-write Git checkpoint: `4b01731f2c183b1c0237dfe4f9d284a59c37a985`. Perforce pending CL 120381 contains new native files and newly opened headers; some pre-existing FEAT-288/289/291 files remain in their earlier pending CLs.
- Added abstract `UEnemyAnimInstance : UBaseLocomotionAnimInstance` with common `bIsMoving` derived from planar movement data. Phantom's `UHumanoidEnemyAnimInstance`, Medusa's special-pose `UMedusaAnimInstance`, and Void's `UVoidGazerAnimInstance` derive from it. The skeleton-free `TABP_Enemy_Locomotion` was reparented to it; Medusa's production child recompiles against the template.
- Preserved Phantom's turn/aim IK logic, Medusa's turn layer and Void's native pose proxy and procedural tentacles. Existing common `DefaultSlot` and `HitReactionSlot` paths remain in all production graphs.
- Editor commandlet recompiled and saved template, Medusa main/special layer, Humanoid/Phantom and Void Animation Blueprints. Development Editor build succeeded. Full Enemy suite: 24/25 passed under `-nullrhi` (`D:/UnrealWork/EnemyAnimUnify/Tests`). The only failure was Phantom PIESmoke expecting a spawned Niagara component. Same test reproducibly failed under `-nullrhi` and passed with rendering enabled (`D:/UnrealWork/EnemyAnimUnify/SmokeGpu`). BGA_PhantomBurst CDO and base both have the expected Niagara system. No animation regression detected.

## Final status — done 2026-10-14

Development Editor / Win64 builds succeeded before and after test changes. All relevant animation, movement and Void PIE checks passed. Phantom PIESmoke passed with rendering enabled; NullRHI cannot serve as proof of Niagara component spawning. Production Animation Blueprints compiled/saved: template, Medusa locomotion/special layer, Humanoid parent/Phantom child and Void. Perforce CL 120381 contains the new class, Humanoid and Medusa header edits, and Phantom AnimBP resave; dependent asset/source files remain in CLs 120332, 120341 and 120351. All pending, unsubmitted. No retargeting was performed.

The common C++ parent shares movement data; species-specific AnimGraph movement pose selection remains in their own graphs. A future graph-host migration would require preserving Phantom's state-machine/IK graph and Void's Sequence Evaluator/proxy; this feature does not claim those graphs were replaced by one graph.
