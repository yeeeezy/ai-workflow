# FEAT-146 Infiltrator native two-weapon animations

Status: done — native gameplay integration completed2026-09-23.

## User-reported stray hands fixed — 2026-09-23

User correctly noticed extra third-person-looking hands in the first-person view. Root cause was the geometry split, not body visibility: original rest-pose hands lie below the1.02m lower-body height cutoff, so the initial LegsMesh accidentally included1523 arm/finger-weighted vertices. The prior pose/follower checks did not verify that lower-body geometry contained no hand vertices; earlier look-down captures showed the stray fragments and were incorrectly accepted.

Checkpoint cea9354 protects the initial lower-body asset and preceding handoff. External fix removes all vertices with any clavicle/arm/hand/finger influence (>1e-5), retaining waist/legs/coat geometry and native weights/rest bones/materials. Final source lower body is15602 vertices, down from17125. split_native.py now excludes these weights during initial generation too. Only SKM_Infiltrator_LowerBody reimported in UE; character visibility, fullbody source, first-person arms, weapon graphs and animation clips unchanged.

Evidence: lower-mesh-audit.json1523stray arm vertices; lower-mesh-fixed.json0remaining; lower-import-fixed.json saved; independent UE FBX export and Blender reimport lower-saved-verified.json reports16586split render vertices and0arm-weighted vertices. lower-runtime-fixed.json checks body visible=false, OwnerNoSee=true, CastHiddenShadow=true, isolates LegsMesh with arms/gun hidden and reviews both guns. Fixed-LegsOnly.png shows no stray hand fragments while the original fullbody shadow remains; Fixed-Laser-LookDown.png and Fixed-Scoped-LookDown.png visually reviewed. cold-gameplay.json passes again. No C++ changes/build needed. PIE ended after checks; editor remains open. Result uncommitted; no push/package.

## Gameplay integration completed — 2026-09-23

User explicitly approved switching the playable character. Checkpoint dabd90a protects prior16clips/docs and existing BP_Infiltrator state;6d532f9 protects preceding completion index;2841d09 protects the preexisting equipment architecture notes before updating the native attachment contract.

BP_Infiltrator now uses existing original SKM_Infiltrator_Lobby/SKEL_Infiltrator_Lobby for CharacterMesh0, new SKM_Infiltrator_FirstPersonArms(9545 vertices) and SKM_Infiltrator_LowerBody(17125 vertices). The split meshes retain native rest bones, original vertex positions/weights/UV/material slots; source lobby assets unchanged. Arms have four dedicated original-material copies with16cm/8cm camera-distance DitherTemporalAA mask, hiding the close shoulder seam. Body/legs do not share these clipped materials. Body remains the original hidden shadow caster; legs follow its pose.

TMIIR alone retargeted20 additional existing body locomotion clips needed to replace the old skeleton without losing walk/jog eight directions and jump/fall/land. Only final FBXs imported under Infiltrator/Animations/Body/Locomotion. Native first-person16clips from prior batch remain byte-identical.3native BlendSpaces,4native AnimBPs and4native Fire/Equip montages use the existing gameplay graph semantics and new compatible assets. Gameplay uses no live retargeting.

Editor-only UInfiltratorAnimationAssetLibrary duplicates the existing concrete graph ancestry, resolves inherited asset overrides to finalized native animations, remaps bone references and removes independent support-hand ModifyBone sway. It does not modify shared templates or perform retargeting. Two existing weapon Blueprints now use native layers/montages, Bip01-R-Hand attachment and measured per-gun hand-to-gun offsets. Aim/muzzle transforms and gameplay settings preserved. Silver camera framing matches the previous GunAxisFix authored target; native scoped hip pose uses the same arms transform, ADS continues to align to its own optic. Saved default loadout remains silver only. Scoped addition during testing was PIE-only.

Verification: Development Editor Win64 build-native-cleanup.log Succeeded (no version/package increment); all four graphs opened, compiled and saved in UE. runtime-acceptance.json passes9checks and938contact samples across both guns, ADS/real ammunition-consuming fire, movement/jump and ADS-cancel-on-switch. Max left wrist positional drift relative to its authored gun contact: silver0.4599cm, scoped1.0405cm (blending/compression included). This is a wrist-contact measure, not a zero-finger-intersection guarantee. locomotion-runtime.json passes271forward/side/back samples, both feet move >5cm and lower-body follower agrees within.02cm. visual-final.json passes final material/fullbody inspection; Final-LaserRifle-Hip.png, Final-ScopedRifle-Hip.png, Final-NativeBody.png and Game-*-ADS.png. Fullbody inspection temporarily makes hidden body visible and moves camera within PIE; it is not a new gameplay camera.

Cold-gameplay.json loads47 animation/graph assets, verifies native skeletons, layers, montages, mesh materials, one-gun loadout and zero animation redirectors/source-work dependencies.140original-package hashes differ only for intended BP_Infiltrator and the two weapon BPs; original lobby mesh/skeleton/materials and weapon meshes/textures unchanged. Existing unrelated working-tree edits/deletions preserved. Original source16clips unchanged.

Resolved authoring/test issues: external source skeleton had not been persisted with its mesh, so it was reimported/saved in TMIIR before body retarget. Editor DLL lock required clean close/rebuild. New graph cleanup removes obsolete left-hand sway/bone references. Several Python verification calls used nonexposed UE methods and were corrected; final reports above supersede earlier failed probes. An incomplete unused material attempt was never saved; discarded at controlled editor restart, cold registry confirms absent. Initial fullbody capture retained the preexisting visible=false flag; final inspection explicitly enables it transiently. No test maps or scene actors saved.

Final editor reopened on secondary (fresh detection and gameplay-gui-placement.json), with clean native Infiltrator PIE for user review; actual ready state recorded in ready-for-user.json. No test callbacks or extra test weapon left in that run. DreamCatchers UE5.6 left untouched. Final source/assets/docs remain uncommitted; no push/package.

## Native animation batch completed — 2026-09-23

User-authorized replacement of bulky skinning preview. TMIIR performs all IK Rig/IK Retargeter/batch operations under /Game/InfiltratorNativeWeaponWork. Native retarget outputs and source assets remain external. Blender refines grip on ORIGINAL Infiltrator_Prepared mesh, weights and rest bones, without stretching arms. Actual current gun geometry matches prior grip references. Both wrists follow the rigid gun; finger rotations corrected after underside/side review, particularly thumb and little finger.

Only16 finalized AnimSequence assets imported to /Game/Characters/Infiltrator/Animations/FirstPerson/{LaserRifle,ScopedRifle}. Each has Still, Idle, Run, JumpStart, JumpLoop, JumpEnd, Fire, FireBase. Existing native SKEL_Infiltrator_Lobby is the target. Fire remains mesh-space additive with its own scaled FireBase. No source skeleton/mesh, IK Rig/Retargeter, retarget work folder, Blueprint, map or config introduced/edited in the destination. Existing unrelated BP_Infiltrator working-tree change predates this batch.

External evidence directory: D:/Blender Projects/InfiltratorNativeWeapons. retarget.json, bake-validation.json, final-evidence.json, final-TMIIR.json and final-TheManTest.json.594 frames and5940 bone checks per project. Cold compressed-pose verification passed in BOTH projects, max position difference 0.009750cm. Initial compressed check falsely sampled unfinished async data; fix is explicit Editor.AsyncAssetCompilationFinishAll and actual native OptionalSkeletalMesh before evaluation. No authored animation corruption was found. preserve.py after confirmed140 original packages unchanged.

Review/Review.html presents both motion montages and Right/Left/Under/Full closeups; video uses32 inspection samples per gun at10fps, not real-time source playback. Gun materials are neutral inspection shaders on real geometry. Updated left/underside and jump/fire samples visually inspected. Finger nearest-surface heuristic is not a closed-mesh collision proof; small residual thumb-surface intersections may remain. Native coat/sleeve appearance is retained. Rendered contact and pose tests do not constitute runtime camera/attachment/PIE acceptance. Gun attachment transform is recorded in grips.json for future integration; do not assume existing player GripPoint socket aligns with this native skeleton.

Fresh monitor detection and review-placement.json verify browser review on secondary [2580,20,4460,988]. DreamCatchers unrelated editor untouched. No C++ or Blueprint changes, so no build/compile required. Final assets/docs uncommitted; checkpoint318b4f1 protects prior handoff. Next: user visual review; gameplay integration is separate from this requested two-gun animation batch. Previous skinning notes below are historical and superseded.

## User correction — 2026-09-23

User found the skinning candidate too bulky and requested native proportions with only the two specific gun animation sets retargeted; pay special attention to hand positions during review. Scope: InfiltratorLaserRifle and InfiltratorScopedRifle,8 sequences each. Prior external skinning candidate remains an unused preview. Current work: D:/Blender Projects/InfiltratorNativeWeapons; all IK Rig/Retargeter/batch operations restricted to TMIIR /Game/InfiltratorNativeWeaponWork. Current source animation/mesh exports are read-only from TheManTest. Checkpoint318b4f1 protects preceding handoff. Existing native grip reference gun geometry bounds and vertex counts exactly match current exported guns. Original character mesh, weights and rest skeleton must remain unchanged.

## Authorization and scope — 2026-09-23

User chose full skinning and requested Infiltrator first to inspect the result. Work lives in D:/Blender Projects/InfiltratorPlayerSkin. No TheManTest assets, Blueprints, code, maps or configuration edited. No animation retargeting performed. Source/lobby files read only. Documentation checkpoint7582a1a preserves previous scene-fade handoff; unrelated changes/deletions not staged.

## Source and binding

Read-only UE export current.json confirms BP_Infiltrator CharacterMesh0 uses SKM_MaintenanceWorker_ReferenceBody, ArmsViewMesh uses SKM_MaintenanceWorker_FirstPersonArms, LegsMesh uses SKM_MaintenanceWorker_LowerBody. Body and arms share67 Blender bones plus FBX armature root, same hierarchy/rest positions within1.04e-6m. Initial obsolete SKM_MaintenanceWorker_Body export had88 bones and was replaced by actual ReferenceBody before binding. Do not use the obsolete skeleton.

Original mesh is InfiltratorLobbyO7665/Infiltrator_Prepared.blend, checked against current lobby FBX. Original textures loaded via source-final.json and packed; source UV/material assignments retained. Existing source weights semantically remapped and bind shape fitted to player reference joint anchors. Finger segments mapped individually. Source coat chains replaced by pelvis/nearest-thigh weights. Boots fitted to target ankles while preserving sole height and adding toe weights. Target skeleton unchanged.

## Files

- Infiltrator_PlayerSkin.blend: editable rest-pose full body, actual player reference skeleton, packed original textures and studio lighting.
- SKM_Infiltrator_PlayerBody.fbx:44927vertices/85782triangles/51weighted groups.
- SKM_Infiltrator_FirstPersonArms.fbx:9545vertices, shoulder cut based on arm weights; same skeleton.
- Infiltrator_MotionPreview.blend:90frames of existing same-skeleton BodyJog; separate hidden arm candidate.
- Preview.png: idle and two jog poses. Individual01_Rest.png,02_Idle.png,03_Jog.png,04_Grip.png,05_Hands.png renders. Grip image copies existing first-person local upper-body pose without weapon geometry/contact validation.
- Infiltrator_MotionPreview.mp4:30frames at15fps, two-second jog excerpt. Scripts and evidence live beside outputs.

## Evidence and limits

binding.json: zero unweighted vertices, normalized weights. Independent FBX reimport fbx-validation.json: names/parents exact, maximum rest-head error5.81145e-7m, UV retained, zero unweighted/unnormalized vertices for both outputs. motion-validation.json:30 evaluated/rendered samples finite and under3m extent. Idle/jog/upper-body/close renders visually reviewed.

Initial NullRHI export failed with MeshObject assertion; render-enabled commandlet export succeeded. Three warnings include historical missing preview-mesh references; no assets saved. No C++ build required. No UE import, Blueprint compile, PIE, actual camera/framing, gun contact, shadow, capsule or collision acceptance. Coat follows legs without cloth and may intersect in additional poses. Shoulder cut and finger/elbow deformation still require user review and game-camera checks. Jump assets exported but not visually accepted.

Fresh monitor detection: primary2560x1600, secondary1920x1080. First image viewer and later editable Blender preview placed and verified on secondary. Original DreamCatchers UE5.6 process untouched; no TheManTest GUI was running. No final commit/push/package.
