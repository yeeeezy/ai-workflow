# FEAT-296 Unified Enemy animation blueprint architecture

Status: review_pending
Started: 2026-10-15

## User authorization

Unify the formal Enemy main AnimBP graph, reusing the movement and Montage routing. Do not keep one C++ AnimInstance subclass per Enemy; VoidGazer was explicitly named. The user reviewed and approved a skeleton-free shared main graph with species pose providers, optional special pose, `DefaultSlot`, and `HitReactionSlot`.

## Baseline

- Formal Phantom uses a skeleton-bound child of skeleton-free `ABP_HumanoidEnemy`, with `UHumanoidEnemyAnimInstance`.
- Medusa uses a child of `TABP_Enemy_Locomotion` plus a linked special-pose ABP, with `UMedusaAnimInstance` on its special layer.
- Void uses its own `ABP_VoidGazer`, with `UVoidGazerAnimInstance`. Its native proxy evaluates a selected sequence, transition, body offsets, wall leap poses and procedural tentacle chains. Gameplay drives the instance through `AVoidGazer` and `UVoidTentacleComponent`.
- All three already expose `DefaultSlot -> HitReactionSlot`, but the main graph ownership is split. Shared `ABP_Enemy_HitReaction` remains the configurable Post Process provider.
- Worktree contained extensive pre-existing unrelated edits. Scoped pre-write Git checkpoints `d9fc580e` (existing animation classes and ABPs) and `7faf8df3` (Void consumers and tests) preserve the baseline. No UE editor process was running at the start. Neither checkpoint is a final delivery commit.

## Work plan

1. Introduce a shared skeleton-free main graph with a base-pose layer, optional special-pose layer, and both existing Montage slots.
2. Move species state collection and procedural evaluation out of the Phantom, Medusa and Void C++ AnimInstance subclasses. Use shared `UEnemyAnimInstance` for the main graph and species-owned components or graph nodes for specialized behavior.
3. Reparent the formal child ABPs, wire species pose layers, and remove obsolete instance classes only after references are migrated.
4. Build Development Editor, compile/save assets in UE, run graph and rendered PIE regressions for Phantom, Medusa and Void, and update architecture docs.

## Progress

- Read harness, active FEAT-287 archive, animation architecture, UE asset workflow and current native/asset references.
- Added skeleton-free `TABP_EnemyBase` and `ALI_EnemyBasePose`; graph is BasePose → optional SpecialPose → DefaultSlot → HitReactionSlot. Phantom, Medusa and VoidGazer now have skeleton-bound Main children and BasePose layers. Medusa retains its separate SpecialPose layer. Production BPs set these classes through inherited `AEnemyBase` properties.
- Shared `UEnemyAnimInstance` owns the main data contract and proxy. Removed `UVoidGazerAnimInstance`, `UHumanoidEnemyAnimInstance` and `UMedusaAnimInstance` after migrating serialized ABP references. Void clip clock is `UVoidAnimationComponent`; its procedural final-pose work is `FVoidGazerPoseProcessor`.
- Development Editor build 16 succeeded. `TheManTest.Enemy.Animation` passed all three cases after the production BP switch. `TheManTest.Enemy.Phantom` passed seven cases, including PIE patrol and tactical movement. Void gameplay suite is still running as this note is written.
- Current edits remain local / in pending P4 CL 120484 and pre-existing pending CLs. No P4 submit or Git push was performed. No retargeting was performed in the destination project.

## Final migration and findings

- Phantom and Medusa cloned pose-layer children initially lost their `ParentAssetOverrides`: copying the old array verbatim left GUIDs pointing at old template nodes. `CopyPoseLayerAssetOverrides` now maps each old parent node to the cloned parent by unique node class and editor position, then writes the new GUID. Phantom's 16 overrides and Medusa's directional BlendSpace override are saved on their new BasePose children. The Phantom `AnimationOverrides` test and Medusa `TemplateInheritance` check pass after cold reload.
- The first Medusa BasePose template still implemented `EnemySpecialPose` inside its inherited movement graph. Linking Medusa's separate special layer displaced the base layer. Removed that duplicate linked node/interface from the cloned ground BasePose template and applied the same clean-up to the Humanoid BasePose template. `FormalBossIntegration` now asserts all three Enemy Main classes and BasePose instances, plus Medusa's separate SpecialPose instance.
- Changed Mesh component default AnimClass and inherited shared animation class settings on `BP_Phantom`, `BP_Phantom_WeaponTest`, `BP_Medusa`, and `BP_VoidGazer`; the test BP did not inherit the formal Phantom's class configuration and was set explicitly. Deleted unreferenced legacy `ABP_Phantom_Native`, `ABP_HumanoidEnemy`, `ABP_Medusa_Locomotion`, `TABP_Enemy_Locomotion`, and `ABP_VoidGazer`. The last cold asset audit confirms all four BPs point to their new Main/BasePose classes and all five old paths are absent (`D:/UnrealWork/EnemyAnimUnify-finalaudit.json`).
- Final Development Editor build: `D:/UnrealWork/EnemyAnimUnify-build28.log` succeeded. Cold UE automation: `TheManTest.Enemy.Animation` 3/3, Phantom suite 7/7 before final asset switch, Phantom `AnimationOverrides` and `PatrolWeaponDummy` after the switch, VoidGazer suite 6/6, HitMontage 2/2. No `-nullrhi` was used. Reports/logs are under `D:/UnrealWork/EnemyAnimUnify-*`. These assert graph inheritance, linked instances, asset overrides, Montage playback, evaluated poses and gameplay regressions; subjective animation appearance has not been separately signed off.
- Source is prepared in pending P4 CL 120484 where possible. Some directly edited pre-existing pending files remain in their original CLs (notably 120332 and 120351); do not submit 120484 alone without reviewing those dependencies. `.agents/harness` is excluded from the P4 client view. Scoped local Git checkpoints preserve pre-existing edits; no final Git commit, P4 submit or push was performed.

## 2026-10-17 main template naming follow-up

- User approved renaming the shared skeleton-free main template from `TABP_EnemyBase` to `TABP_Enemy_Main` so its relationship to `ABP_Phantom_Main`, `ABP_Medusa_Main` and `ABP_VoidGazer_Main` is clear. Renamed through Unreal `EditorAssetLibrary`; all three children were compiled and saved. No pose graph, animation clip, skeleton or gameplay behavior was changed.
- Updated the two C++ automation asset paths and current architecture documentation. `HitMontage.GraphAndDefaults` still referenced two main ABPs removed during FEAT-296; removed those obsolete assertions so the test checks the current shared main graph. This was a test migration gap discovered during verification, not a gameplay failure.
- Development Editor / Win64 build succeeded. Cold `TheManTest.Enemy.Animation` 3/3 and `TheManTest.Enemy.HitMontage` 2/2 passed. Direct old-path scan found no Enemy asset references; the old package file and redirector are absent. Evidence: `D:/UnrealWork/EnemyMainRename-parents.json`, `D:/UnrealWork/EnemyMainRenameTests/index.json`, `D:/UnrealWork/EnemyMainRenameHitRetest/index.json`.
- Scoped pre-write Git WIP checkpoint `51b5b813`. Result changes remain local and unsubmitted. Project P4 login reported an expired session, so no Perforce checkout, changelist or submit was attempted in this follow-up. Visual review status remains pending.

## 2026-10-17 reusable ground locomotion follow-up

- User approved extracting common ground BlendSpace evaluation into a Linked Anim Graph. Scoped pre-write Git WIP checkpoint `5d0562ea` preserves relevant local work. Added skeleton-free `TABP_EnemyGroundLocomotion` with `UEnemyGroundLocomotionAnimInstance` inputs `GroundBlendSpace`, `GroundX`, `GroundY`, `GroundPlayRate`.
- `TABP_Humanoid_BasePose` now calls the ground graph from patrol and aim states; its Phantom child supplies the existing patrol and aim BlendSpaces. `TABP_EnemyGround_BasePose` calls the same graph for Medusa, whose child supplies its existing directional BlendSpace. Old BlendSpace player nodes, disconnected getters and obsolete child BlendSpace overrides were removed. Species turn states, IK and Medusa special layer remain in their respective graphs.
- Created `BS_VoidGazer_Ground` from five existing final same-skeleton sequences, with idle at zero, forward/back/left/right at speed 140 and direction axes. Void BasePose selects the shared ground graph during ordinary ground locomotion and retains its SequenceEvaluator for wall/leap/look/melee/tentacle paths. `UVoidAnimationComponent` exposes the branch flag and ground play rate to `UEnemyAnimInstance`; the existing Void post-pose processor remains in place. No retargeting occurred in TheManTest.
- UE commandlet compiled/saved the graph assets and reported no Python errors. Development Editor builds succeeded during implementation.
- The first Void BlendSpace build had six valid serialized samples but lacked runtime triangulation. `TemplateInheritance` caught this by querying cardinal-direction runtime sample resolution. The editor helper now calls `ValidateSampleData()` and `ResampleData()` before saving; the repaired asset passes all five queries after cold reload. `GroundLocomotionPlayRate` is connected to Void's linked graph, preserving its idle and walking rate settings.
- Final Development Editor build succeeded (2026-10-17 16:28 local). Cold D3D12 editor run `D:/UnrealWork/EnemyGroundModule-resampletests` passed 6/6: `Enemy.Animation` 3/3, Phantom `PIESmoke` and `PIETacticalRetreat` 2/2, Void `GroundCombat` 1/1. Earlier full run `D:/UnrealWork/EnemyGroundModule-tests` passed 16/18; its two Phantom 1.1-second distance checks reached 75.8 and 70.1 cm against an 80 cm threshold while reporting valid strafe speeds. Both passed final retest at 83.7 and 82.4 cm. The full run also passed the other Phantom, Void and HitMontage cases, including Void wall leap, tentacle grab and live BT ground hop. Rendered RHI was D3D12, not NullRHI. UE graph compiles and asset saves succeeded; a separate subjective appearance review remains pending.
- Added a final assertion that Void BasePose actually binds `BS_VoidGazer_Ground`, beyond merely storing valid samples. Development Editor rebuilt and isolated cold `TemplateInheritance` passed 1/1 (`D:/UnrealWork/EnemyGroundModule-bindingtest`); `git diff --check` and `feature_list.json` parsing passed.
- Changes remain local and unsubmitted. No destination-project retargeting, P4 submit or Git push.

## 2026-10-17 reader-facing handoff documentation sync

- User identified the paired Enemy handoff documents omitted from the previous architecture update. Updated `Documentation/Enemy/EnemyDevelopmentStandard.md` and its offline HTML counterpart together: Phantom now points to `ABP_Phantom_Main` and `ABP_Phantom_BasePose` instead of deleted `ABP_Phantom_Native`; the hypothetical new Enemy creates Main and BasePose assets and configures both classes; the guide explains shared `TABP_Enemy_Main`, optional SpecialPose and `TABP_EnemyGroundLocomotion` responsibilities.
- Replaced the migration-era instructions still present in `Documentation/Enemy/EnemyAnimationTemplate.md` with current assembly, ground linked graph inputs, species examples and new Enemy steps. This documentation work did not change game code or assets. Scoped pre-write Git WIP checkpoint `0faa0907` preserved that file's prior local edits; the resulting docs remain local and unsubmitted.
- Verified all named existing animation asset paths, parsed the HTML, checked key architecture terms in both handoff formats, scanned all three docs for removed classes/assets, and ran `git diff --check`. FEAT-287's standard remains a review draft; this sync only corrects its animation content.

## 2026-10-17 Perforce sync and submission

- Synced the full project to Perforce head (1,227 incoming files, including the BlockoutTools plugin and project descriptor); a second sync preview reported up to date. Existing local pending work was preserved.
- Consolidated the FEAT-296 runtime assets, shared graph/code, relevant automation tests and the paired reader handoff documents into pending CL 120484. Replaced the obsolete pending `TABP_EnemyBase` add with `TABP_Enemy_Main`. The final changelist contained exactly 56 files; unrelated pending changelists remain open.
- Verified all opened versioned files matched head and no resolves were pending. The Development Editor build after sync succeeded (`D:/UnrealWork/FEAT296-postsync-build.log`); prior cold D3D12 gameplay and graph tests remain the relevant runtime evidence.
- Perforce renumbered and submitted the change as **CL 120796** on 2026-10-17. `p4 describe` confirms the submission, and a subsequent sync preview reports up to date. Subjective appearance review remains pending; no retargeting was performed in this destination project.
