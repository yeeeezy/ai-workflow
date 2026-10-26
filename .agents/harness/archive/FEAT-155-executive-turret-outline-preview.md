# FEAT-155 Executive turret outline preview

Status: done. SciFi HUD edge refinement completed2026-09-25. User approved proposal 2026-09-25.

Selective checkpoint 9c541c3 preserves known preexisting touched docs and FEAT154 Drone changes. Broad unrelated work/deletions remain untouched.

Hold E uses transient skeletal component in idle pose, custom-depth stencil253 only, no main/depth pass, collision, shadows, AI or GAS. Owner-local camera postprocess draws red external silhouette with transparent interior. Invalid target hides; cancel/confirm/EndPlay destroys component and removes exact blendable. Preview and actual spawn share GetTurretPlacementTransform, captured on confirmation with player aim yaw; existing terrain validation retained. One owner-local postprocess material; enable custom-depth stencil mode3. Build and rendered PIE complete. Evidence D:/UnrealWork/ExecutiveOutline.


## Verification and integration

Development Editor Win64 build succeeded with no compiler warnings/errors; material compile and cold PIE passed. TestMap real Enhanced Input E/LMB:24 assertions cover valid/sky/release/pause/reentry/confirm, no preview collision/shadow/combat spawn, exact actual position/scale/rotation despite post-confirm turn, full travel/sink/turret and owner cleanup. Additional rendered stencil1 on/off cube comparison preserves existing scan highlight while stencil253 preview stays pure red; active-preview unpossess cleans immediately. Evidence D:/UnrealWork/ExecutiveOutline.

First rendered check exposed existing M_Infiltrator_Highlight outlining every CustomDepth mesh, adding unwanted orange interior edges. Its original graph now gates only reserved stencil253 out of the existing outline alpha (including adjacent pixels); all other highlight behavior retained. Final Outline.png and ConfirmedPreview.png visually inspected: transparent interior and red silhouette. Deployed.png confirms the actual turret; zoom returns after confirmation, so screenshot pixel sizes differ, while measured mesh world transforms match. ScanCompatibilityOn/Off.png prove normal stencil1 cube highlighting still works alongside the red preview. No persistent test cube or map save.

External verification scripts corrected Python API naming/reflection and screenshot callback reentrancy; ownership test waits for the existing0.1s drone cleanup tick. These were test-harness issues, not new gameplay failures. Final verified.json has24 passing checks; compatibility.json succeeds. Native code was unchanged after the successful build. New material84 instructions; filtered highlight102 instructions. Editor placed/verified on detected secondary monitor; final editor exited.

Selective checkpoint d75ba5f preserves preexisting FEAT154 archive-index entry before completion update. Four source files, DefaultEngine custom-depth mode3, one new Executive material and the scoped scan-highlight exclusion comprise this task. No Blueprint, animation, map, package/version change, final commit or push. Unrelated preexisting edits/deletions remain.


## User correction: hologram, not outline

User rejected the outline-only visual and authorized a full red translucent holographic projection. Prior screenshots are superseded, not accepted final appearance. Checkpoint d87d1b5 preserves previous scoped implementation. New owner-local translucent unlit two-sided skeletal material adds readable volume, Fresnel rim, animated horizontal scan rows and broad moving scan band. Native preview uses this surface material, no CustomDepth/camera blendable. Undo previous-only scan-highlight exclusion and renderer override; remove obsolete outline material via editor after native rebuild. Shared final spawn transform and placement rules remain. Evidence D:/UnrealWork/ExecutiveHologram; build and rendered PIE passed.


### Final hologram verification

User-corrected full red hologram implemented. Unlit translucent two-sided material204 instructions compiles/cold-loads; Development Editor Win64 succeeds. Actual TestMap PIE24 checks pass through E/LMB, transparent surface without custom depth, invalid sky/cancel/pause/reentry, unchanged camera blendables, full deploy and exact mesh position/scale/yaw, owner cleanup. Hologram.png visually reviewed: full semitransparent volume, internal shape, red glow and scan rows. Prior outline-only material removed via Unreal after zero referencers; original highlight graph/config restored. Evidence D:/UnrealWork/ExecutiveHologram.

PreviewHologramMaterial now assigns only the transient preview mesh. No camera postprocess, stencil reservation or global renderer dependency. Defaults: HologramColor(1,.004,.012), GlowIntensity3, Opacity.32; fine3.4cm horizontal rows,135cm broad scan cycle, Fresnel rim and faint pulse. Actual turret still uses original materials. Existing full-size preview/final-transform sharing retained. Original M_Infiltrator_Highlight alpha connection restored and previous three added nodes removed through MaterialEditingLibrary; obsolete M_TheExecutive_TurretPlacementOutline deleted after zero referencers, verified absent on disk. No material imported from external pack, no animation/retargeting/Blueprint/map changes.

Final captures Hologram.png/HologramLater.png and Deployed.png are actual PIE; screenshot size1600x900. Visual confirmed full surface, transparency and scanlines. Editor placed/verified on freshly detected secondary monitor and exited. Native build21.45s, no compiler errors/warnings. No packaging/version increment, final commit or push; no saved temporary test fixtures. Prior outline section is historical and explicitly superseded.


## TMIIR SciFi HUD edge refinement

User accepts hologram direction but requests Scifi_HUD_Effects HUD edge quality. Read actual source material graphs/instances and rendered source HUD without saving TMIIR. M_uiFrame: translucent two-sided, cosine edge masks with Edge sharpness1000 in MI_uiFrame, Power20 and Color alpha0.1. M_uiHoloMesh: Fresnel exponent10, high edge power, masked single-sided, scrolling lines. SourceHUD.png reviewed: narrow nearly white luminous core/soft color halo and very faint transparent panel. Source data/captures in D:/UnrealWork/ExecutiveHologramHUD.

Checkpoint6ad3022 preserves current touched material/docs. Adapt these principles to red 3D turret: sharpen core with EdgeSharpness10, separate EdgePower10, much fainter body, antialiased sparse scan rows, single-sided surface to reduce doubled interior haze. Material-only change; no source assets migrated, no native logic/input changes. Compile/rendered PIE passed.


### HUD-reference final verification

SciFi HUD reference refinement complete. Actual TMIIR Scifi_HUD_Effects material graphs/instances inspected and SourceHUD.png rendered/read without saving source assets. Existing hologram now separates sharp luminous core (EdgeSharpness10/EdgePower10), soft red halo and faint surface; single-sided rendering reduces doubled haze and antialiased scan rows preserve full volume. Material211 instructions compiled, cold-loaded and actual TestMap E/LMB PIE24 checks passed. Final Hologram.png visually compared with prior version and source HUD. Material-only revision; no new C++ build required. Evidence D:/UnrealWork/ExecutiveHologramHUD.

The prior uniformly bright coral fill is replaced by a much fainter red surface and stronger narrow illuminated contours over the actual 3D model. This remains a complete hologram, not the superseded silhouette-only postprocess. Main/secondary prototype parameters were not changed. Shader core uses exponent10 and independently tunable power10, broader rim exponent3, body alpha0.065 before scan/edge contributions and Opacity scaling. Front faces render only, matching the source mesh hologram choice. Actual source HUD and final turret screenshot inspected. C++ unchanged since previous successful21.45s build; this material revision compiles211 instructions and24 end-to-end runtime assertions pass. Both editors exited; source/reference and target windows verified on detected secondary. No map/Blueprint saves, imported assets, animation/retargeting, package/version change, final commit or push.
