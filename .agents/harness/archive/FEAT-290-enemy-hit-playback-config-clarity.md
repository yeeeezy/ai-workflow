# FEAT-290 Enemy hit playback configuration clarity

## Authorization and scope

2026-10-14 user agreed that Post Process hit playback should retain explicit component-applied displacement controls, while Montage should follow its authored animation and AnimBP Root Motion settings. User asked for the explanation to be added and for other necessary related corrections. This follow-up changes configuration help text and project documentation; it does not migrate production enemies to a new AnimBP or change hit playback behavior.

## Initial audit

- `FEnemyHitReactionRequest::bInPlace` and `FEnemyHitReactionRule::bApplyRootMotion` affect only the Post Process Animation branch. Montage ignores those fields and uses the asset plus AnimBP Root Motion settings.
- Rule `MaxDuration` and `BlendIn` / `BlendOut` affect Post Process Animation or ControlRig. Montage plays its authored length and blend settings, modified by configured PlayRate.
- `Strength` controls Post Process blend intensity. Montage accepts only positive strength but does not scale pose intensity.
- `Documentation/Enemy/EnemyAnimationTemplate.md` still describes the old `DefaultSlot → Output Pose` chain and omits the saved `HitReactionSlot`.

## Implementation and verification

- Added DA Details tooltips for `Strength`, `bInPlace`, both `bApplyRootMotion` fields, rule/global `BlendIn` / `BlendOut`, and rule `MaxDuration`. These state which fields apply to Post Process Animation, Control Rig and Montage. Playback code and asset values were not changed.
- Added a motion/timing ownership table and electrical `bInPlace` example to `Documentation/Enemy/EnemyHitReactionModes.md`. Corrected `Documentation/Enemy/EnemyAnimationTemplate.md` to show `DefaultSlot → HitReactionSlot → Output Pose`, slot setup and the existing graph test.
- UE 5.7 Development Editor / Win64 build passed after UHT processed the metadata, 2026-10-14 21:24 local. `git diff --check` passed for scoped files. No Blueprint asset changes were made.
- Source header and hit reaction document remain in existing pending Perforce CL 120341; template document remains in existing pending CL 120332. Neither change was submitted. WIP checkpoint before edits: `37d077631bb8d1501466510e0c39d7b6cee0f719`.

The broad AnimBP migrations suggested during design review were left out of this configuration clarification. Medusa's formal AnimBP has separate turning logic, while Void's native pose proxy has wall and tentacle behavior; both require their own behavior-preserving migration and visual verification. This follow-up makes no claim that those migrations are complete.
