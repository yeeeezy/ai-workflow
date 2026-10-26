# Boss landing and scan follow-up audit ? 2026-09-17

User request: landing must depend on player position; ascent is too high; inspect and record why the original red scan is missing in SpaceStation. This turn inspected and recorded findings, without implementing runtime fixes.

## Requested encounter changes
- Current ascent is about460m above flyby and visually too high. Reduce it and expose a configurable height;80?120m was proposed, not yet implemented or numerically approved.
- User explicitly requires player-relative landing instead of the fixed Station_BossLanding marker.
- Current Reassembly.GetLandingPoint uses LandingReference or a4000cm forward fallback, then probes WorldStatic; Entry GA and Reassembly.Start call it separately. The correction must use one shared captured ground destination for dive and assembly, otherwise they can disagree if the player moves.
- Recommended timing to confirm/design: capture the player's current position immediately before dive, project to usable ground, then lock until landing. Do not silently assume the earlier trigger-time flyby snapshot is also the desired landing snapshot. Ground validity/clearance and no-ground behavior need definition. No change applied yet.

## Scan findings (cold editor read)
- SpaceStation has an enabled/unbound PostProcessVolume, BlendWeight1, but WeightedBlendables is empty.
- TestMap has an enabled/unbound PostProcessVolume, weight1, with Mat_Outline_library and M_Highlight (both under /Game/Characters/Infiltrator/Materials), each weight1.
- Mat_Outline_library is a PostProcess material. Its outline color=(1,.032094,0,1), flow color=(1,.048886,0,1): still red/orange, not changed to gold. Asset Registry currently references the correct plural Materials/MPC_ScanEffect.
- Both Infiltrator and MaintenanceWorker CDO ScanEffect reference the correct MPC and M_InfiltratorScanTerrainAdaptive. Their HeadCamera weighted blendables are empty. ScanEffect itself drives MPC/decal but does not install a post-process material.
- FEAT-056 records a separate gold/white adaptive terrain decal intentionally coexisting with the original red post-process. Missing SpaceStation blendables explain loss of that original red layer; gold terrain scan is a different effect.
- Loading TestMap also logs an obsolete singular Material/MPC_ScanEffect missing-package reference. Exact referring object is not established; do not misreport it as a confirmed current Mat_Outline_library dependency (registry shows plural path). Audit stale reference during repair.
- Proposed repair: install the existing scan/highlight postprocess through a consistent player-owned or map-configured path, preserve existing station grading, remove obsolete reference if identified, and validate actual scan-on/off and red rendering. Not applied in this audit.

## Evidence and workspace
D:/UnrealWork/ScanAudit/audit.json compares maps/CDOs; material.json records material domain/colors/dependencies; editor logs retain old-path warning. No Blueprint compile/save or map save invoked by audit scripts. A separate user UnrealEditor process (PID19608, started00:06:50) remained open; SpaceStation.umap changed on disk00:14:35 while audit logs contain no SavePackage. That map modification was not overwritten, reverted or included in our checkpoint; ownership/content not inferred.

Follow-up FEAT-099: SpaceStation scan/highlight blendables restored and rendered on/off verified while implementing scan-owned highlight lifetime. Missing postprocess is resolved; lower boss climb/player-relative landing and obsolete MPC warning remain separate follow-ups.
