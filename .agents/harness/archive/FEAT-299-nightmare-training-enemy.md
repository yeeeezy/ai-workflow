# FEAT-299 Nightmare training enemy scaffold

Status: in_progress. Started: 2026-10-18. FEAT-298 window vault was paused at the user's request.

## Scope

Create two isolated Nightmare-family test enemies for two incoming engineers with little Unreal experience. They share one TrainingEnemy directory and the same future scoped Perforce permissions, while each engineer owns one monster's BP and skill pool. Reuse the existing Enemy lifecycle, engagement, distance-based GA selection and ground movement. Leave mesh, animation and trainee GA implementations empty. Keep TestMap as the sole test map. The user chose direct `write` on this training folder for both engineers, so shelving is optional rather than a submission gate.

## Implementation

- User approved a local Git WIP checkpoint of the pre-existing abnormal dirty state: `02ad7d56`, covering tracked changes only. No Perforce submit or Git push.
- Added owner-specific `ATrainingEnemyAIController`, which starts the shared lifecycle tree once its pawn and configured Combat subtree are ready. Added an editor-only `UTrainingEnemyAssetLibrary` under the same Nightmare owner to build its saved Combat and Ambient BT graphs from shared task classes.
- Created six `/Game/Enemy/Nightmare/TrainingEnemy` assets: `BP_TrainingEnemy`, `DA_TrainingEnemy_Engagement`, `BP_TrainingEnemyAIController`, `BB_TrainingEnemy`, `BT_TrainingEnemy_Combat`, and `BT_TrainingEnemy_Ambient`. The combat tree uses auto Near/Mid/Far selection and parallel KeepPlayerDistance; ambient uses RandomPoint. The profile uses proximity/hit activation. One phase has empty GA arrays. Mesh and animation class remain unset.
- Team-facing instructions and proposed Perforce path boundary are in `Documentation/Enemy/TrainingEnemyOnboarding.md`.

## Verification and handoff

- Development Editor Win64 compiled successfully. Both Blueprints compiled and six assets saved. Cold commandlet confirmed profile, controller, Blackboard, BT graphs, no mesh/AnimBP, and one empty skill phase. Evidence: `D:/UnrealWork/TrainingEnemyOnboarding/author.json` and `cold.json`.
- Temporary TestMap PIE reached Engaged and moved 993 cm with the dedicated AI controller. A separate distant trial remained Dormant and moved 913 cm with ambient RandomPoint. Temporary actors were removed; TestMap was not saved. Evidence: `pie.json`, `pie-ambient.json`.
- No trainee GA was authored or activated, so distance choice is verified by saved BT configuration, not an actual ability cast. No Perforce submission or permission change has occurred.

## Two-enemy and Dev Panel follow-up (2026-10-18)

- User requested two independently assignable monsters within one shared-permission training folder and separate Dev Panel placement controls. Local scoped Git WIP checkpoint `973f261b` captured the preceding FEAT-299 state. Pending Perforce CL `121055` was created for this follow-up; no submit.
- A remains the initial `BP_TrainingEnemy` and its six assets. B adds six independent `*_TrainingEnemy_B*` assets (own BP, AI controller, Blackboard, Combat/Ambient BTs and engagement profile). Both have one empty GA phase, no Mesh/AnimBP, and use the same native controller class and shared runtime AI tasks.
- Added `DA_TrainingEnemy_A_Placement` and `DA_TrainingEnemy_B_Placement` under the owner Data folder. Dev Panel `WBP_DebugMenu` now has Training Enemy A/B buttons in TEST PLACEMENT, using its existing `UTestPlacementSubsystem` and Undo Last/Clear Placed. Neither test monster is saved into TestMap.
- Development Editor Win64 build succeeded after widget C++ edits. UE commandlet saved/compiled B, both presets and WBP; cold load verified distinct refs and empty presentation/skill slots (`dual_author.json`, `dual_cold.json`). Actual TestMap PIE invoked both panel placement callbacks: A and B each spawned once with their own AI controller and were fully removed by Undo; result `dual_pie.json` has `ok:true`, map unsaved. First PIE probe failures were Python inspection API limitations only; corrected the external test script and final trial passed.
- Team-facing `Documentation/Enemy/TrainingEnemyOnboarding.md` now states project-wide read plus shared TrainingEnemy-folder write for both users, direct submit, separate assignment ownership, and the common-asset collision risk. Pending CL 121055 was inspected before submission: exactly 24 FEAT-299 files (14 Enemy assets, five existing Dev Panel files, four native TrainingEnemy files and one team document); no unrelated opened files were included. No server permission change occurred.

## Perforce submission (2026-10-18)

- User explicitly requested Perforce submission. Rechecked pending CL 121055's exact 24-file scope and updated its description to reflect successful build/cold/PIE tests. `P4Local.cmd submit -c 121055` succeeded; Perforce renamed it to submitted CL **121062**. `p4 describe -s 121062` lists exactly those 24 files; target paths are no longer opened. Server documentation readback begins with the expected two-enemy handoff. Unrelated opened changelists were untouched. No Git push or server permission change. Earlier pending-CL references above are historical; use 121062 for administrator/team handoff.
