# FEAT-292 Enemy unused-content cleanup

## Authorization and baseline

User explicitly requested deleting unused Enemy content across more than animation. Initial 2026-10-14 inventory: 1,179 registered assets under `/Game/Enemy`; 117 have no serialized asset referencers. C++/config string references and gameplay entrypoints must also be checked because zero Asset Registry references alone is insufficient. Existing worktree has roughly 896 dirty paths; local non-mutating WIP checkpoint `13ac3199f7117fa37122c6f37eeaaee65978d0b5` was created before cleanup.

## Scope rules

- Remove assets only after checking serialized referencers, code/config string loads, and production entrypoints.
- Retain dynamically loaded ambient BTs, gameplay cue assets resolved by tags, and explicit test/production entrypoints even when Asset Registry reports zero referencers.
- Update the Medusa production Mesh AnimClass and tests before deleting obsolete ABPs, then re-run a cold registry audit and relevant tests.
- No retargeting in this project. Use Unreal Editor asset deletion; record Perforce delete/add actions in a dedicated pending changelist. Do not submit without a new request.

## Evidence

Asset Registry inventory: `D:/UnrealWork/EnemyCleanup/registry.json` and `summary.json` (external work files).

## Cleanup performed

- `BP_Medusa` Mesh default Anim Class was changed from the old `ABP_Medusa` to the production `ABP_Medusa_Locomotion`. Template and hit-Montage tests now inspect production Medusa/Void graphs, so the old Medusa ABP and temporary `ABP_Medusa_EnemyTemplateExample` could be removed. The unused `BS_Medusa_Movement` 1D BlendSpace was also removed.
- Four iterative editor deletion passes removed 166 assets: 102 initially unreferenced assets, then 29 and 34 newly orphaned dependencies, then one obsolete Phantom reaction sequence used only by a stale test expectation. This includes old Phantom legacy/mannequin and OriginalRifle animation prototypes, unused Phantom textures and meshes, unused Void clips/body-only mesh, an unused shared ability Blueprint, and the non-production Void GroundImpact WIP system/modules/meshes/materials. Every editor deletion succeeded. 165 tracked assets are marked `delete` in pending Perforce CL 120361; the remaining one was the unsubmitted template example add, abandoned from CL 120332. The stale attached-reaction expectation was updated to the current Phantom heavy-left sequence as the one source edit in CL 120361.
- Removed obsolete Medusa editor helper methods for pre-migration ABP/1D BlendSpace creation and old source-project importing/copying. Current directional/turning/linked-layer authoring helpers remain.
- Cold Enemy Asset Registry and disk both contain 1,013 assets. All 166 deleted packages are absent and surviving Enemy assets have zero dependencies on them. The 15 remaining zero-serialized-referencer assets were reviewed: dynamic C++ behavior trees, Gameplay Cue assets resolved by tag, production/test entrypoints, and the native-loaded Phantom muzzle Niagara system.
- Removed 17 empty Enemy directories left by those deletions, including the retired Void GroundImpact and Phantom OriginalRifle animation branches.
- `Documentation/Enemy/EnemyAnimationTemplate.md`, `EnemyHitReactionModes.md`, Enemy development standard Markdown/HTML and relevant architecture notes now describe the current production paths. FEAT272's paused but unused GroundImpact WIP deletion is documented in `arch/07-character-classes.md`.

## Verification

UE 5.7 Development Editor build passed after C++ test/helper edits. `TheManTest.Player.Weapons.AttachedLimbReaction` passed in normal RHI after the stale test expectation was updated. The full `TheManTest.Enemy` suite ran 25 tests: 21 passed (including animation template/formal integration, movement, engagement, hit Montage, Medusa and Void) and four Phantom tests failed. A second Phantom suite under normal RHI reproduced the same four failures (`PatrolWeaponDummy`, `PIESmoke`, `PIETacticalApproach`, `PIETacticalRetreat`). Errors concern test-dummy configuration, projectile impact, RepairGun muzzle Niagara visibility, and tactical lateral movement; no deleted package was requested in the load log, and registry dependency audit found none. These failures remain unresolved and should be investigated separately; their pre-cleanup status was not measured. Reports: `D:/UnrealWork/EnemyCleanup/EnemyTests/index.json`, `PhantomNormalTests/index.json`, and `AttachedTestFinal/index.json`.

Pending Perforce CL 120361 contains exactly 165 asset deletes and one source edit. It has not been submitted. `BP_Medusa` production AnimClass edit stays in CL 120351; template and Montage test/doc edits stay in their earlier CLs 120332 and 120341. The working tree also contains unrelated pre-existing changes.

## 2026-10-14 Phantom regression repair

User asked to resolve the four failing tests. Investigation showed the production Phantom still patrolled and reached its points, while `BP_Phantom_WeaponTest` had lost its `bPatrolWeaponTest=true` and plain `AAIController` defaults. The saved Blueprint now has those defaults; full `PatrolWeaponDummy` PIE passes including projectile damage, slow, death and ragdoll. `TestMap` spawns an Executive whose sole initial weapon is ExecutiveSlowRifle, so the Phantom smoke test's immediate RepairGun Niagara assertion was invalid; removed that unrelated assertion. The separate asset configuration check still verifies RepairGun's Niagara system and scale.

The shared `KeepPlayerDistance` preset previously stopped within its distance band and moved radially outside it, removing Phantom's combat sidestep. Added an optional strafe setting, enabled on BT ground-preset tasks while direct component callers retain their previous default. It selects NavMesh reachable lateral goals (350 cm step), with diagonal approach/retreat. Real PIE measured lateral speeds 256.8 cm/s at 700 cm, 150.7 cm/s while approaching, and 125.8 cm/s while retreating. All seven `TheManTest.Enemy.Phantom` tests pass.

The first complete Enemy rerun passed 24/25; the remaining `GroundPresets` failure was a random waypoint too close to produce the test's minimum 100 cm travel after MoveTo acceptance. Random-point selection now requires a useful minimum travel where the configured radius allows it. Final Development Editor build passed; independent `GroundPresets` PIE passed. Intermediate reports: `D:/UnrealWork/EnemyCleanup/PhantomRepairedTests2/index.json`, `EnemyRepairedTests/index.json`, `GroundPresetRepairedTest/index.json`; build `phantom_repair_build3.log`.

Final acceptance supersedes the preceding partial-build result: the entire `TheManTest.Enemy` suite was rerun on the final build and **all 25/25 tests passed**, including GroundPresets, all seven Phantom tests, Medusa, Void, movement, animation and hit reactions. Report: `D:/UnrealWork/EnemyCleanup/EnemyFinalTests/index.json`; automation exit code 0.

CL 120361 now contains 165 tracked asset deletes and six edits: the earlier attached-reaction test, ground preset header/implementation/task, Phantom feature tests, and weapon-test Blueprint. It remains pending, not submitted. The Phantom feature test source also contains earlier FEAT-292 asset-path updates in the same file; audit its complete diff before submission.


## Perforce delivery (2026-10-22)

After the owner reviewed the deletion categories and explicitly approved them, the pending cleanup was submitted as Perforce CL 121856. The server changelist contained 170 files: 165 Enemy asset deletes and five source edits. The earlier six-edit note above was a planning snapshot; the submitted server file list is authoritative. Final Enemy automation evidence remains D:/UnrealWork/EnemyCleanup/EnemyFinalTests/index.json (25/25). No Git push.
