# FEAT-125 Infiltrator rifle surface and clear reflex sight

Status: done
Closed: 2026-09-18
User approved the presented plan on 2026-09-18: clear sight window, dark metal with restrained orange paint and material differentiation; validate ADS/fire/transitions. All visible windows on secondary monitor. This scoped task supersedes the previous acceptance freeze for this work only; FEAT120/113 stay pending.

Checkpoint 3663732 preserves known FEAT124 cleanup, excluding the unrelated user MaintenanceWorker Blueprint. Evidence/scripts: D:/UnrealWork/InfiltratorSightRevision. Blender sources: D:/Blender Projects/InfiltratorSightRevision.

Audit: original authoring uses a single opaque skin with fixed metallic .55 and roughness .46. Source geometry has an enclosed long optic, so surface changes alone cannot produce the reference's open reflex window. Reuse original body geometry and author owner-local optic and material slots; preserve animations and original Worker LaserRifle.

Implementation: owner-local SM_InfiltratorLaserRifle preserves source body and replaces two optic geometry islands with an open reflex frame, one transparent window and tiny centre reticle. Six sections cover body, optic metal, paint, rubber, glass and emitter. Original UV texture retained; body shader separates paint/metal roughness and metallic response with subtle machining normal detail. No source skeletons, animations or retargeting assets involved.

First rendered ADS showed a white lens despite correct slot/material/section mappings. Exported polygon groups and cold slot audit confirmed the lens was bound to Glass. Switched lens to low-opacity unlit tint to remove the excessive lit translucent reflection. Second actual PIE screenshot Rifle-ADS.png confirms visible terrain throughout the aperture and centre-aligned reticle. Initial aim/fire/ammo/release/re-entry/single-slot-cycle checks passed. Final colour polish and cold/Blink validation pending.


Final verification: Final final.log editor exit0; cold.json ok=true: 10 owner assets, 6 material slots, no redirectors, original Laser mesh independent, Infiltrator Lobby child consumer verified; weapon/Lobby Blueprints compiled. visual.json ok=true: actual ADS, aimed fire/ammo, release/re-entry, single-slot cycle and all6 materials restored after Blink. Final Hip/ADS/Blink screenshots inspected. No C++ changes; no native rebuild needed. All visible UE windows verified on secondary monitor. Existing unrelated Worker reference-material/legacy scan MPC warnings persist. No animation or map writes.
Final lens is a low-opacity tint, not a magnifying scope or scene capture. User visual acceptance remains available in the secondary-monitor review window. Final material/mesh changes are uncommitted.
