# FEAT-280 CoreMorph GAS ownership

User approved component/GAS refactor. Transformation remains independent GA outside phase skill sets; manual preview. No new inputs or review maps.

Baseline exported from real BP: 7 components, native tail and missile tuning, tail3/missile4 material references, phase Near tail only, no Far missile, DefaultAbilities Flight/EncounterEntry/Reassemble. Evidence D:/UnrealWork/CoreMorphGASRefactor. Scoped pre-write checkpoint15d97d7a preserves3 known include-order fixes; other pending workspace changes untouched. Original CoreMorph source and four affected assets backed up externally. Perforce edit rejected expired credentials; local writable copies only, no submit/sync.

Implemented GA-owned attack state and tick-task lifecycle, Body-owned meshes and pose, locomotion-only motor. Removed old Combat/Effects component sources. Three Cue native bases now Actor classes under same reflected class names; saved BP loaded correctly, no recreate needed. First successful build build3.log. Boss/Cue BPs opened/compiled/saved; saved materials migrated exactly and behavior tree moved to motor. Runtime verification completed; see final verification below.

## Final verification and closeout
Status: done. Closed 2026-10-12 (local).

- Final Development Editor / Win64 build succeeded in 5.93s, no new compiler warnings (build-delivery.log).
- BP_CoreMorphBoss and three attack Cue BPs opened, compiled and saved in UE. Cold reload verifies Body/Flight/ScorpionMovement/Reassembly only, exact saved material references and native attack defaults, unchanged DefaultAbilities and phase skill sets, and Cue Asset Registry tags (cold.json).
- Station_Manta external actor retained its old enabled=true override on the new motor, with location/form/entry/reassembly/climb settings preserved. Only its external actor package F/4W/68VCV3P9SEAQZQQS860JTO was saved; SpaceStation map package was not saved.
- Final runtime.json: seven cases pass (tail completion and three cancellation stages, missile completion/cancellation, manually started transformation completion).
- Final edges.json: six cases pass (tail actual health100->75, tail death, invalid target, missile health75->55, missile death, transformation cancellation). Attack tags, Cue pools and transformation state clean up; body owns no attack effect pools.
- Existing TheManTest.Enemy.Movement.ScorpionExecution automation passed at 2026-10-13 05:59:00 UTC. Motor yields locomotion while an attack is active. Final editor-delivery.log has no engine Error/Ensure entries. Scoped git diff --check clean; no removed Combat/Effects component names remain under Source.
- Earlier external validation scripts failed due unavailable Python API, out-of-range fixtures and calling ability APIs on CDOs; these harness errors were corrected before final runs. No claim that earlier failed runs passed. An explicit-player-target fixture experiment was abandoned; final damage checks use the real player in the blast and a stable TargetPoint for targeting.
- Transformation stays registered independently, outside PhaseSkillSets; StartReassembly manual preview retained. Existing explicitly enabled SpaceStation encounter-entry transformation preserved. Missile remains unequipped in production, as before.
- External fixtures cleaned (edges-clean.txt); no temporary test map objects saved. No new production test inputs/maps. Architecture docs07/10/11/15 updated. No claim of user visual approval or performance improvement.
- Final implementation remains uncommitted/unsubmitted; pre-write checkpoint15d97d7a and external baselines retained. Perforce credentials remain expired. FEAT-278 stays pending visual review.
