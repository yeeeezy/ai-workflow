# FEAT-106 RepairGun persistent walkable foam

Status: done (2026-09-17)

User approved: environment foam remains until the current world ends and supports standing/jumping/stacking. Enemy hits retain slow and immediate destruction. No cross-session persistence requested.

- Startup feature_list.json parsed; active_feature was null. Previous FEAT-105 work preserved in local checkpoint 25bdbf2; no push.
- Initial source audit: expansion schedules destruction after LifetimeAfterExpansion (native default 5 seconds). Root sphere is QueryOnly and blocks Pawn; visual mesh has no collision. Actual Blueprint dimensions and walkability require audit.
- Evidence/scripts: D:/UnrealWork/RepairFoam. Only primary display available; use hidden/headless editor verification.

Completion: Development Editor build; production Blueprint audit/compile; actual PIE projectile impact, persistence past old timeout, landing/standing/jumping and stacked foam; retain enemy slow behavior. No saved test fixtures/maps.

## Implementation and verification

- Actual Blueprint audit: lifetime 5s, expansion 0.5s, scale5, root radius15cm, mesh local scale2.5 and ~5cm asset half-extents. Pawn and sphere already block each other and allow step-up, but the sphere is larger than the visible foam. Removed timed destruction; clear Actor lifespan on environment impact. Enemy slow/destruction unchanged.
- Solid foam switches Pawn/projectile blocking to BulletMesh with one generated 26-DOP convex hull in SM_RepairGun_Bullet. Root sphere retains hazard overlaps. Repeated hits do not restart expansion. Solid foam disables Actor Tick and projectile movement. No cross-world persistence.
- build-final.log Development Editor Win64 Succeeded. Earlier test-only protected-access compile error corrected using existing public slow multiplier getter.
- Blueprint compiled/saved in headless editor (author.log). Native field removal needed no binary Blueprint delta. Initial author.json incorrectly counted primitives only (UE GetSimpleCollisionCount excludes convex shapes); cold audit uses GetConvexCollisionCount. No assumption that the initial zero count meant missing hull.
- pie-final.log: TheManTest.Player.Weapons.RepairFoam Success. Production projectile physically hits temporary floor; real player lands/jumps/lands again; second projectile physically hits first foam; stacked foam supports character for another7 seconds, beyond prior lifetime. Foot-to-visible-top difference3.4cm, movement base is BulletMesh. Repeated-hit and enemy slow/destruction assertions pass.
- First PIE run exposed a test fixture initialization mistake (Static mesh component before SetStaticMesh); changed test floor to Movable before assigning mesh. Final test passes without this warning. All test objects transient; no map saved.
- Existing unrelated WBP_PauseMenu stale PausedWorldBlur GUID ensure appears at editor startup before test execution; not introduced or modified here. Final automation result Success. Only hidden NullRHI verification because secondary display unavailable; no rendered visual acceptance claimed.
- Final work remains uncommitted/unpushed. Prior work preserved in25bdbf2.
- Final independent editor cold read: cold.json ok=true, convex_count1, SimpleAndComplex, InitialLifeSpan0. Commandlet lacks StaticMeshEditorSubsystem, so final read used full hidden editor instead. All background editors exited. Git confirms only RepairGun mesh/code/tests and harness changes; no maps or other weapon assets changed.
