# FEAT-291 Formal Medusa and Void animation integration

## Authorization

2026-10-14 user asked to follow through on both the previously suggested Medusa formal animation template migration and Void integration. Do not perform animation retargeting in TheManTest.

## Initial audit

- `ABP_Medusa` currently uses `UMedusaAnimInstance : UBaseLocomotionAnimInstance` for smoothed MovementSpeed, MovementDirection, RotationRate and TurnWeight. Its graph blends directional and turning BlendSpaces before DefaultSlot and HitReactionSlot. The FEAT288 shared-template child is only a movement and skill playback example, not the formal Medusa AnimClass.
- `AVoidGazer::BeginPlay` forces `UVoidGazerAnimInstance : UAnimSingleNodeInstance`, using a custom proxy that evaluates authored wall/ground sequences and deforms native tentacles. Gameplay code directly drives `PlayCrawl`, play rate, clip time, wall leap blending and native pose weights. A direct swap to the generic locomotion template would discard this behavior.
- User clarified that Void tentacles are procedural and should remain so; its other animation playback should be unified into an AnimBP. Proposed boundary: AnimBP owns sequence evaluation and shared skill/hit slots, while the existing native proxy continues the post-evaluation body/tentacle deformation. This requires refactoring the proxy base from `FAnimSingleNodeInstanceProxy` to `FAnimInstanceProxy` and exposing clip/time inputs to the new AnimBP.
- Broad pre-existing worktree retained. WIP checkpoint before changes: `d98217d22d722beb290d04d79153c0d77f20a0a6`.
- Baseline cold `TheManTest.Enemy.Animation.TemplateInheritance` currently fails because FEAT289 inserted `HitReactionSlot` after `DefaultSlot`, while FEAT288's test still selects an arbitrary Slot node and expects that node to be `DefaultSlot`. This is a stale test, not an observed graph failure. Correct the assertions to identify both named slots and their actual links during this feature.

## Implementation

- Medusa production `BP_Medusa` now sets `ABP_Medusa_Locomotion`, a new child of `TABP_Enemy_Locomotion` with `BS_Medusa_Directional` override. `AMedusa::BeginPlay` enables the original speed/direction smoothing and links `ABP_Medusa_SpecialPose`, which mixes the authored `BS_Medusa_Turning` against the incoming pose using native `RotationRate`/`TurnWeight`. The original `ABP_Medusa` remains a reference asset; the FEAT288 example remains a separate teaching asset.
- Void production `BP_VoidGazer` now sets `ABP_VoidGazer`. `UVoidGazerAnimInstance` is a full `UAnimInstance` with selected Sequence/time variables. The graph evaluates that clip and passes it through `DefaultSlot` and `HitReactionSlot`. Its `FAnimInstanceProxy` evaluates the graph first, then preserves existing authored-pose transition, upper-body offsets and procedural tentacle deformation. Gameplay still chooses ground, wall and attack clips through its existing state logic.
- Void skeleton registers `HitReactionSlot` in `HitReactionGroup`. No IK Retargeter, source rig/skeleton, or retargeted animation was created in TheManTest.
- Project instructions updated in `Documentation/Enemy/EnemyAnimationTemplate.md`; architecture updated in `arch/06-animation.md` and `arch/12-anim-blueprint.md`.

## Verification

- UE 5.7 Development Editor/Win64 UHT and native build passed.
- Editor Python created, compiled and saved the three AnimBPs; production Blueprint CDO class assignments were saved and verified in a separate cold load. All four relevant AnimBP CDOs report `RootMotionFromMontagesOnly`.
- `TheManTest.Enemy.Animation.TemplateInheritance` and `TemplatePlaybackPIE` passed after correcting the stale DefaultSlot test for FEAT289's second slot.
- `TheManTest.Enemy.Animation.FormalBossIntegration` passed in PIE for production classes, linked Medusa turning layer, Void native interface/selected clip and evaluated poses; final run also passed skill and death Montage playback assertions.
- Void PIE regressions `GroundCombat`, `TentacleGAS`, `TentacleGrab`, and `WallLeap` passed, zero errors. Expected fixture warnings about default pawn/NavMesh remain.
- New work is isolated in pending Perforce CL 120351. The updated teaching document and stale test are already opened in earlier pending CL 120332. No CL submitted in this task.
