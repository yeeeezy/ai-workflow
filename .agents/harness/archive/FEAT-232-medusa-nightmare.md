# FEAT-232 Medusa nightmare

Status: done
Closed: 2026-09-30

User approved runtime scene darkening plus emissive materials as in the browser reference, explicitly rejecting silhouette postprocess masking. Nightmare enemy Medusa temporarily uses Phantom mesh. During nightmare Medusa is invisible until a player projectile hits; use existing Infiltrator scan gold reveal, hold/fade about0.8s, repeat hits refresh. No attacks or attack tell in this batch. Existing blink accepted as-is.

Checkpoint19749ff7 preserves completed FEAT231 changes; user config/two rifle packages excluded. External evidence/scripts D:/UnrealWork/Medusa.

Read-only findings: EnemyBase.ReactToProjectileHit is already called for valid non-enemy projectile instigators before damage; can override in Medusa. Current scan is MI_Infiltrator_ScanGoldXRay surface effect (FEAT196/197), not obsolete smoke/Flame. Need suppress independent scan reveal during nightmare and restore exact materials/lights after skill.

Milestone: native Medusa + GAS timed skill + world-owned darkness restoration implemented. Blueprint and30 original-preserving player/weapon emissive mappings saved. Development Editor build4.65s succeeded. First full TestMap run passed actual projectile hit/fade/repeat, scan exclusion/preexisting scan cleanup, overlapping sources, cancellation, timeout and death. Cold103checks passed; equipment/possession and SpaceStation visual follow-ups ongoing.
Bug found in equipment lifecycle test: swapping back copies visible emissive MID to ShadowStaticMesh. Restore now remaps exact owned MID identities across all affected weapon components after ending equip wrappers. No changes to original weapon assets. Smoke eligibility also observes concealment when legacy smoke is manually enabled.

## Final verification and handoff
Development Editor Win64 build5.25s succeeded without new compiler warnings. BP_Medusa and BGA_Medusa_Nightmare opened/compiled/saved. TestMap185 checks passed (real projectile hit, refresh/fade, scan exclusion/preexisting ownership cleanup, overlap, timeout/cancel/death); final lifecycle38 checks passed including proximity, actual equipment swaps, swap-time cancellation/shadow material restoration, unpossess/repossess and destruction. SpaceStation2337 checks passed including actual hit/reveal and all captured scene surfaces restored; delayed hit screenshot visually inspected. Native Scan.GoldHighlightLifecycle Success. Final cold103 checks passed,42 scoped assets/30 emissive mappings, no redirectors/retarget assets. Final editor39632 SpaceStation, noPIE, dirty content/maps0 on sole primary.

Production Blueprint: /Game/Enemy/Nightmare/Medusa/Blueprint/BP_Medusa. Drag into an existing level to use. Auto activation within25m after1s spawn grace;8s duration,12s cooldown; hit hold0.1s then fade to0 by0.8s, repeat hits refresh. All values configurable on Medusa. No attack logic. No saved map placements; test fixtures transient. UI remains English.

Implementation uses world scene state in this single-player project, not a silhouette screen mask. It temporarily suppresses scene lighting/atmosphere, neutralizes static emissive surfaces and uses source-preserving unlit emission variants for player meshes/weapons; normal source packages are not overwritten. Future new player/weapon materials need a matching variant in the Medusa Blueprint map. Existing emissive VFX remain independent. No performance benchmark or multiplayer support claimed.

Fixes: weapon shadow components copy runtime materials during equip; cleanup now restores owned MID identities across copied references. Dynamic original material parameters are copied into emissive variants while active. Existing scan regression passes after adding virtual reveal eligibility, including optional legacy smoke exclusion.

Evidence: D:/UnrealWork/Medusa/{build-final.log,author.json,review.json,lifecycle.json,station.json,cold.json,cold-final.log,handoff.json}. Visuals: Hidden.png,HitReveal.png,StationHidden.png,StationHit.png,StationRestored.png. Initial station screenshot captured before the mesh reached the render frame; final StationHit waits0.1game seconds and shows actual gold reveal. Initial scripting/API issues are superseded by successful final reports.

Checkpoint19749ff7 preserved accepted blink. Final Medusa/source/harness edits uncommitted; no push/package/version changes. User DefaultGame.ini and two rifle Blueprint edits excluded and not overwritten. Existing arch/10 file has legacy encoding; only ASCII notes appended without rewriting its existing bytes.
