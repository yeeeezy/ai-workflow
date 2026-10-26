# FEAT226 Portal border refinement

User approved reference-driven refinement2026-09-30. Reference BV1TZjw6FEyY inspected from public video. Keep horizontal shape/live capture, add thin core, flowing violet texture and soft frayed outer wisps; reduce clipping. Scope two existing horizontal border/view materials. Checkpoint cc4ec024 preserves FEAT225. Work/evidence D:/UnrealWork/PortalBorder.

Status: done (closed2026-09-30)

Implemented first material pass in two existing packages: exact28vertex silhouette now uses signed segment distance for uniform world-plane rim width, three moving samples of existing engine TilingNoise05, shared noisy aperture/core/body/wisps, controlled violet HDR and derivative AA. Same runtime texture/foreground/OpenAmount parameters. No native or Niagara changes. Live compile/save completed; actual PIE validation running. First scripting attempt corrected unnamed ComponentMask input pins before successful authoring.

## Final result and verification
Final parameters BorderWidth.006/CoreGlow1.15/FlowSpeed1, three noise frequencies3.8/15/37 and signed contour offsets.016/.006, violet core/body/wisps/halo,10cm intersection fade. Scene alpha and rim share perturbed28point contour; runtime foreground gate and OpenAmount retained. Only two existing material packages changed; no C++, BP, Niagara, texture, map, source project or retarget edits.

Live material compile/save passed; final cleaned graph actual TestMap39checks pass (includes repeated frame deadline checks): opening,10m exit, stationary opening/timeout, arm materials preserved, live scene, near/far screenshots, walking translation/velocity/view/audio/close/capture cleanup. FinalNear00001.png and FinalFarB00001.png visually inspected. SpaceStation12/12directions open and actual walk-through passes, cameraerror2.13e-14; StationFixed_0000000.png inspected. Sequential far captures show evolving fine contour. No performance benchmark claimed.

Independent final commandlet cold load0errors/0warnings checks one custom shader per material, exact shared shader code and three parameter defaults, emissive/opacity hookups, runtime texture parameters/foreground gate, BGA references, Niagara fallback/user binding/warmup and10m/5s defaults. Cold validation initially found obsolete nodes because engine DeleteAllMaterialExpressions iterates while removing expressions; fixed authoring to delete an explicit snapshot via DeleteMaterialExpression. Final cold verifies only one shader per graph and final PIE rerun passes. No engine modification.

Editor34592 restored SpaceStation on detected sole primary monitor, noPIE and dirtycontent/maps0. Checkpointcc4ec024 preservesFEAT225. Final results saved/uncommitted; user config/rifles/pycache untouched. No package/version/push. Evidence/scripts D:/UnrealWork/PortalBorder; reference download D:/UnrealWork/PortalBorderReference. Subjective visual acceptance remains user review.
