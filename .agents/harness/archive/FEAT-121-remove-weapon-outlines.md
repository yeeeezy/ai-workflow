# FEAT-121 Remove weapon outlines

Status: done
Closed: 2026-09-17

2026-09-17: User explicitly requests removal of outlines from every gun, including future use. This authorizes the required source/assets/build/editor follow-up after the manual acceptance slot. WIP checkpoint 4b8cf69 preserves FEAT120 and prior package state. No source/resource project edits.

Audit: only LaserRifle has native SourceOutline and dedicated M_LaserRifle_Outline / SM_LaserRifle_Outline; InfiltratorLaserRifle inherits it. Repair/Electric/Explosion outlines were removed earlier. Keep scan/highlight materials and UI border styling. Remove the native component and Blink special case, compile/save descendants, then delete zero-consumer outline assets through Unreal and verify all gun instances. Evidence: D:/UnrealWork/WeaponOutlineRemoval.

Implementation: removed native SourceOutline and Blink bOutline/bVisible special handling. Development Editor build.log succeeded. First NullRHI editor pass saved two BPs then crashed while spawning audit instances; no deletion had occurred. Normal D3D12 editor rerun completed assets.json ok=true, 10 gun/display configurations, zero outline references, deleted two dedicated assets through EditorAssetLibrary. Cold.json confirms both assets and references absent. Actual D3D12 PIE visual-final.json passed two Blink activations, mid-dash cloak, exact original material restoration, 350cm travel, pause/resume HUD; restored screenshot inspected. No secondary monitor was reported in this session, so all editor runs used RenderOffscreen; no visible editor was placed on the primary monitor. ADS/keyboard regression pending.

Final verification: regression.log BlinkKeyboard Success; combined run exited without emitting the final LaserRifle result, so only that test was repeated. aim-regression.log LaserRifleAim Success (physical RMB, sustained ADS, real firing/impact, reload, weapon cycling, pause, unpossess). Build succeeded with existing external-plugin StructUtils deprecation notices. No map/config/source-project changes; existing packaged executable is unchanged and predates this removal. Removed-outline policy recorded in AGENTS.md and current equipment architecture. Final changes remain uncommitted.
