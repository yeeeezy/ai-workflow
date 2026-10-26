# FEAT-289 Configurable Enemy hit reaction Montage presentation

## Scope and authorization

2026-10-14 user approved adding a DA choice between the existing Post Process hit-reaction path and direct Montage playback. This follows the reviewed plan to use a dedicated hit slot and define behavior when a skill Montage is active. Existing Phantom and Medusa Post Process configuration must remain the default until explicitly changed. No animation retargeting inside TheManTest.

## Initial audit

- `UEnemyHitReactionConfig` currently supports Disabled, Animation and ControlRig; `FEnemyDirectionalReaction` and rule pools accept only `UAnimSequence`.
- `UEnemyHitReactionComponent::RequestReaction` selects an AnimSequence, activates a Post Process class, and samples time/alpha in `UEnemyHitReactionAnimInstance`.
- Medusa skills and death play Montage on the main AnimInstance. The current main AnimGraphs use `DefaultSlot`; the new Enemy locomotion template also has `DefaultSlot`.
- Broad pre-existing Git working tree: 876 status entries; pending Perforce change 120332 contains FEAT-288 files. Local WIP checkpoint before edits: `e8379b38d1e4b21ae98e8adc9214d979c5ce6bc2`.

## Implementation

- `UEnemyHitReactionConfig` adds Montage as an opt-in mode, directional Montage sets, per-rule Montage pools, and validation of skeleton and `HitReactionSlot`. The prior enum values are retained so saved DA values do not shift.
- `UEnemyHitReactionComponent` plays Montage on the main AnimInstance; it does not feed the Post Process sequence. Existing skill/death Montage has priority when a hit arrives. If a skill begins during a hit, the hit stops on the next component tick. Existing hit no-restart/minimum interval and death cleanup remain.
- Saved `HitReactionSlot` after the existing `DefaultSlot` in `TABP_Enemy_Locomotion`, `ABP_HumanoidEnemy`, and `ABP_Medusa`. Registered `HitReactionSlot` in `HitReactionGroup` on concrete Medusa and two Phantom skeletons. Main and child AnimBPs were opened, compiled and saved. No retargeting was performed.
- Added `Documentation/Enemy/EnemyHitReactionModes.md` with DA setup and playback policy. Phantom and Medusa production DAs keep Post Process Animation mode.

## Verification and handoff

- UE 5.7 Development Editor build succeeded after the user closed the editor that had locked the DLL. Final build after skill-priority fix succeeded 2026-10-14 21:09 local.
- Asset authoring script wrote all three main ABPs, three child ABPs and three skeletons; evidence `D:/UnrealWork/EnemyHitMontage/author.json`. Editor crashed while shutting down open asset windows after the save marker. Separate cold editor runs loaded the saved assets and passed `TheManTest.Enemy.HitMontage.GraphAndDefaults` and `PlaybackPIE`, including the final skeleton slot-group checks, 2026-10-14 21:10 local. Existing `TheManTest.Enemy.EightReactionAssets` and `HitReactionRules` also passed 21:08 local. Final report: `D:/UnrealWork/EnemyHitMontage/VerifiedReport`.
- Files are opened in pending Perforce changelist 120341, not submitted. FEAT-288's new `TABP_Enemy_Locomotion` remains in pending 120332 and includes this slot; both changes are needed when submitted. No other pending changes were submitted.

Feature complete for the approved selectable path. Existing production DAs remain on Post Process Animation; Montage selection requires assigning destination-skeleton Montages as described in the project document.
