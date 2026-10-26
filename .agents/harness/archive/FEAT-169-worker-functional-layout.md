# FEAT-169 MaintenanceWorker functional asset layout

Status: done. Closed2026-09-26. User explicitly authorized extending the approved Gameplay/Lobby/_Shared scheme to MaintenanceWorker. Preserve current appearance and3weapon loadout; no retargeting. Retain uncertain unused/legacy assets for later user decision.

Audit226 packages;205 AssetTools moves planned,29 shared body dependencies proven by Lobby/gameplay reachability,54 external consumers. Existing shared/global weapon assets remain with their owners. Native/legacy animation variants kept distinct. Native editor helper has one explicit old locomotion path to update. Evidence and mapping: D:/UnrealWork/WorkerLayout. Checkpoint2d32aa9 protects scoped dirty assets/external consumers/harness; unrelated edits untouched. Editor40472 launched on freshly detected secondary monitor, bounds2572,12,4468,985; arrival has no dirty packages.

User reaffirmed role-owned weapon variants after discussing global Weapons ownership. Keep Gameplay/Weapons and per-role FirstPerson/Animations; do not migrate to global CharacterVariants.

Milestone:205 moves saved,54 external consumers resaved, all captured scalar/vector values identical. Native editor fallback path updated; Development Editor Win64 build succeeded11.31s after owned editor closed (initial attempt correctly rejected by Live Coding while a failed cleanup assertion had kept editor open). LobbyMap was resaved only to replace its existing Worker display-class reference.

Seven transient rename mesh copies consolidated through UE. Engine delete calls removed in-memory redirectors but left3 tiny disk packages with no referencers (two verified ObjectRedirectors, one no-asset old Lobby BP package); after editor exit, backed them up externally and removed exact validated paths.39 empty directories removed under Worker root. No actual Worker content assets deleted. Fresh cold verification and actual runtime tests next.

## Completed verification

D:/UnrealWork/WorkerLayout: build.log Development Editor Win64 Succeeded11.31s; cold.json1009 checks/205 moves/exact scalar-vector values; runtime.json24 actual SpaceStation checks (three weapon classes/layers/fire montages, movement/body/arms); lobby.json13 checks (shared body, three ready/relaxed poses); final-cleanup.json zero dirty maps/content.226 content assets retained,17 unused candidates retained for user decision,39 empty directories removed. No final commit/push/package/version increment.

The only cold dependency difference is removal of the preexisting nonexistent BP_MaintenanceWorker_Old during compilation. Final startup has no missing-package/object or Blueprint compile errors. SpaceStation movement689.7cm and foot motion58.07cm; all3native linked animation layers and fire montages play. Actual screenshots inspected. LobbyMap resaved for class-path reference only; PIE fixtures were not saved. No asset names, palettes, gameplay behavior or retargeting changed. Full obsolete audit list: FEAT-169-retained-assets.md. FEAT167 remains paused.
