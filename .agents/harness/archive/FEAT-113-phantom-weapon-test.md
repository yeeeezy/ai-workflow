# FEAT-113 Patrolling Phantom weapon test variant

Status: in_progress

User authorized this scoped follow-up and continued parallel supervisor management. Prechange WIP checkpoint 660450e preserves all prior batch plus user GameMode default selection. Root alone harness; no finalcommit/push.

Criteria:
- Keep production Phantom unchanged
- Reusable variant patrols but cannot activatecombat from perception/damage/alerts; stays hittable
- Actual projectile damage/slow/death and patrol verified, no permanent map fixtures

2026-09-17 final audit: combined-cold.json passes clone flags/plain AIController/empty offensive grants and original visual identity. First PatrolWeaponDummy PIE failed movement/arrival because its distant transient floor lacked NavMesh; normal Humanoid patrol requires pathfinding and has no old documented direct-move fallback. Damage/no-combat/phase/no-cloak/slow/expiry/death assertions did not fail. Agent correcting unsaved TestMap navigation fixture; do not weaken patrol assertions or alter production behavior to pass. User must assign PatrolPoints on reachable navigation when placing the reusable clone.

Final113 test-fixture updates compile (final-build.log Succeeded5.02s). Corrected public navigation API use and duplicate NavData; real navigation endpoint projection now passes. FINAL remaining failure: actual patrol translation and arrival still fail in final-pie.log, despite reachable endpoint navigation. Damage/no-combat/phase/no-cloak/slow/expiry/death pass.113 NOT complete. User acceptance released for verified other features; all UE processes exited, no further editor/build/target writes during manual acceptance. Read-only diagnosis next.
