# FEAT-288 Skeleton-free Enemy animation template and overrides

## Authorization and scope

2026-10-14 user approved implementing the proposed boss/enemy animation architecture, emphasizing a skeleton-free AnimBP template and child overrides so most new enemies only fill movement animation assets. Shared movement state and a slot for GA-driven skill Montages belong in the template; skeleton-specific presentation remains per enemy. Runtime animation/retarget authoring boundary remains: do not retarget inside TheManTest.

## Initial audit

- Current humanoid uses `ABP_HumanoidEnemy` plus Phantom native child. Medusa owns `ABP_Medusa` with movement and turning BlendSpaces. Void uses `UVoidGazerAnimInstance : UAnimSingleNodeInstance` and a custom pose proxy; `AVoidGazer::BeginPlay` assigns that native class. No Void AnimBP asset exists.
- Existing player animation uses skeleton-free `TABP_CharacterBase_*` templates, so Unreal template inheritance and editor authoring support already exist in this project.
- Runtime animation files and Enemy asset directory rules read. Broad Git dirty state is pre-existing; Perforce opened empty. Local WIP checkpoint before edits: `5e08df8aa60999f1c476c440d47ff1f7d64a6192`.

## Implementation

- Added `UEnemyAnimationAssetLibrary` editor helpers to author the skeleton-free template, one movement BlendSpace Parent Asset Override, `ALI_EnemySpecialPose`, and the optional linked layer. Shared graph: `BlendSpace(Direction, Speed) -> EnemySpecialPose(InPose) -> DefaultSlot -> Output`. The interface default implementation passes `InPose` through.
- Created `/Game/Enemy/_Shared/Animations/Logic/TABP_Enemy_Locomotion` and `/Game/Enemy/_Shared/Animations/Interfaces/ALI_EnemySpecialPose`. Created `/Game/Enemy/Nightmare/Medusa/Animations/Logic/ABP_Medusa_EnemyTemplateExample`, bound to Medusa's original Skeleton with `BS_Medusa_Directional` as serialized Parent Asset Override.
- Existing `ABP_Medusa` remains assigned because its native animation instance supplies smoothing and turning. Void's native single-node proxy remains assigned because wall/ground and tentacle poses have not been reproduced. No retargeting performed in TheManTest.
- Project-facing instructions: `Documentation/Enemy/EnemyAnimationTemplate.md`. Local architecture index updated in `arch/06-animation.md` and `arch/12-anim-blueprint.md`.

## Verification

- Development Editor / Win64 build succeeded after C++ edits.
- UE editor cold-opened all three animation assets, compiled and saved each (`D:/UnrealWork/EnemyAnimTemplate/editor_verified.json`; `Saved/Logs/TheManTest.log` at 2026-10-14 20:39 local).
- Cold-load automation `TheManTest.Enemy.Animation.TemplateInheritance` passed: template has no skeleton, child inherits it and persists the Medusa BlendSpace override, interface is pass-through, and the shared slot graph is connected.
- PIE automation `TheManTest.Enemy.Animation.TemplatePlaybackPIE` passed in TestMap: a temporary BP_Medusa using the child AnimBP evaluated its skeletal pose, received nonzero locomotion speed, and played the existing `AM_Medusa_Melee` Montage through the shared slot. Both tests succeeded, automation exit code 0 (`Saved/Logs/TheManTest.log` 2026-10-14 20:43 local). Existing TestMap AI/navigation warnings did not fail the tests.
- UE auto-opened the three new assets in the default Perforce changelist. They were moved, together with the three source files and project-facing documentation, into isolated pending changelist **120332** (seven adds). No Perforce submission requested or performed.
