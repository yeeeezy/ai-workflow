# FEAT-105 Phantom native retarget and patrol

Status: done (implementation and machine verification; user visual acceptance pending)

User authorized native animation retarget, both-hand rifle correction and saved patrol placement in current SpaceStation. External authoring: D:/Blender Projects/PhantomRetarget and TMIIR only. Target receives finalized assets. Checkpoint a2977de. Existing BP is stationary test configuration; restore shared AI and speeds. No final commit/push authorized.

Initial audit: FEAT-084 adapted original mesh to old mannequin proportions; no retarget. Original recovered rig and weights available in A0100_RigRecovery. ExecutiveLobby contains proven UE5.7 retarget pipeline.

## Authoring milestone

- Native CAT rig exported without mesh deformation (surface delta below 0.00001 m), semantic bone names preserve original weights and four-spine hierarchy. Dedicated hand_r_wep follows fitted rifle mount.
- 31 active sequences retargeted in TMIIR only. External Blender correction maintains left support contact, adjusts reachable two-arm stance and preserves release/reload motion. Root displacement is carried by actual top imported root for four reactions. Final FBX reimported and validated in TMIIR; source_final_validation.json ok=true.
- Both sides of rifle grip and relaxed/run/reload previews inspected. Final destination integration underway.
- Existing editor asset override helper now also updates child override entries; Development Editor build succeeded. No gameplay C++ change.
- User requested shutdown after completion; save and finish verification before shutting down.

## PIE findings and corrections

- Saved SpaceStation real Phantom_Patrol plus four PatrolPoint actors in Gameplay/PhantomPatrol. Route around (1800,300)-(2400,900), about 21 m from existing PlayerStart. Added actual 50x50m navigation volume, four links validated. 65-second PIE reached every point within 100 cm acceptance; all samples Patrol, moving/turning/stop states and socket attachment passed.
- Native MCP volume factory produced an empty brush; replaced via engine CubeBuilder on builder brush + BRUSH ADDVOLUME and notified navigation bounds. Empty failed volume removed; only valid navigation saved.
- Runtime smoke caught mirrored fitted weapon mount (negative determinant) and reaction top-root FBX conversion error. Corrected right-handed fit basis and baked armature root movement directly from external retarget source; re-authoring all 31 clips in external workspace. Previous visual smoke was NOT accepted.
- Original rifle had no Muzzle socket, preventing existing AimIK initialization. Added measured muzzle socket to actual Phantom rifle and native AimSocket; native AimAlpha reaches 1.0. Final grip orientation and four-direction root movement still require final PIE recheck.

- Final corrected import completed. Full rendered combat smoke passes: native AimAlpha1, four clips attached exactly, four reaction capsule displacements101..147cm in expected directions, cloak material restore, and native pelvis ragdoll. Final patrol and cold audit underway.

- Opposite-side close-up exposed remaining right-palm gap. Refit actual rifle 2.5cm rearward/1cm downward in external authoring, preserving original arm lengths and baked support contact; regenerating31 final clips. Prior near-side-only visual acceptance superseded.

## Final acceptance

- Final right-grip correction migrated after external31-clip rebuild and TMIIR revalidation. Both actual UE aim views inspected; support hand and right grip contact corrected. Native proportions and weights preserved.
- Development Editor Win64 build.log Succeeded. All changed Blueprints compiled; cold_audit.json ok=true confirms31 native sequences, native ABP dependencies, no owner redirectors/IK rigs/retargeters, four saved points, no temporary validation actors.
- combat_pie.json ok=true: AimAlpha1,62 bones, capsule half88; relaxed/run/reload/fire clips attachmenterror0; four capsule root displacements101..147cm in expected directions; cloak material restoration and native pelvis physics passed. This smoke exercises animation/reaction compatibility, not a full combat playthrough.
- Final cold patrol records actual arrival counter, current point and wait timer, confirming complete four-point loop and continued circulation. Engine arrival handling includes navigation projection/reach tolerances; raw point-center distance is not the acceptance assertion.
- Acceptance instructions and screenshots: D:/Blender Projects/PhantomRetarget/Acceptance.md, UE_Aim_Native.png, UE_Aim_Right_Final.png and UE_Patrol_*.png. Only real patrol/nav actors saved; test camera/light removed. Existing PlayerStart unchanged. No package regenerated, no final commit/push.
- User-authorized shutdown follows final save and editor exit.

- Final cold patrol:12 actual arrivals, all4 indices visited. All six final evidence JSONs ok=true. Harness audit found pre-existing FEAT-080 duplicated across active/archive indexes; left historical unrelated entries unchanged. FEAT-105 exists exactly once.
