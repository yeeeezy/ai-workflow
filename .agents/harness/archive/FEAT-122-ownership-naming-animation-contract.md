# FEAT-122 Ownership, naming and animation contract cleanup

Status: done
Closed: 2026-09-17

2026-09-17: User authorized continuing the remaining audit findings. FEAT121 outline removal already completed. Checkpoint61e63a4 preserves that change; newly observed user BP_MaintenanceWorker edit deliberately excluded and its hash saved externally.

Plan: move Executive sniper mesh/materials/textures to its weapon owner using AssetTools, preserve separate Lobby hologram override; replace native runtime animation string reflection with typed FireAnimationWeight and migrate graph references/defaults; semantic rename for imported Laser cloud dissolve texture; update Phantom tests to current native animation and GAS paths. No retargeting or source-resource edits. Align documented Blueprint ability prefix with existing project BGA_ usage rather than mass-renaming unrelated abilities. Evidence/scripts: D:/UnrealWork/OwnershipNamingCleanup.

Implementation:
- AssetTools moved/renamed 11 Executive mesh/material/texture packages from Lobby to Weapons/ExecutiveSlowRifle and one Laser dissolve texture to T_LaserRifle_CloudDissolve_BaseColor. Saved all referencers and fixed redirectors. Gameplay hidden hologram and Lobby visible hologram overrides remain distinct.
- UCharacterBaseAnimInstance now reads/writes native FireAnimationWeight. ADS weapons supply hip/aim values; other weapons restore each animation class default. Migrated graph references and defaults in ten AnimBPs. Removed the worker's unused old-name _0 duplicate only after a reference check. Editor migration helper belongs to the existing Core/Editor asset library; no runtime Blueprint-name reflection remains.
- Updated Phantom native animation, BlendSpace, GAS ability and shared cover test paths; ExecutiveEquipped now checks the owner-local mesh. Corrected the stale RepairGun muzzle assertion to the FEAT072 SniperScout system already configured in its Blueprint. LaserRifleAim additionally asserts native arms/layer weights and restoration on non-ADS equipment.
- BGA_ documented as the existing Blueprint ability convention; native UGA_ naming unchanged. Gun outline prohibition retained. Architecture documents updated.

Verification milestones:
- build-final.log: Development Editor Win64 Succeeded, no compiler errors or new compiler warnings.
- assets.json and finish-asset.json: authoring/compile/save succeeded. cold.json: 12 new paths load, old packages and referencers absent, no Executive weapon dependency on Lobby, no scoped redirectors, both material overrides intact, all ten saved animation defaults preserved. Exported graphs contain no old weight variable name.
- verify-final.log: Phantom.AnimationOverrides, Phantom.ReusableCombatModules and ExecutiveEquipped all Success.
- First regression run exposed two fixture assumptions: obsolete RepairGun muzzle expectation (corrected against documented live configuration), and LaserRifleAim unable to find the gun. Readback confirms the user's saved worker loadout contains ExplosionGun/RepairGun/ElectricGun. Final aim run uses a fresh editor and transient RepairGun/LaserRifle loadout; it never saves character or map changes. The saved loadout was preserved.
- filesystem-final.json: every old package absent, every new package present, no empty old parent directories, user BP_MaintenanceWorker SHA256 unchanged. Existing packaged executable is not rebuilt in this cleanup.
- aim.log: LaserRifleAim Success, including actual RMB ADS/fire/impact/reload/switch, native hip/ADS weights in arms and linked layer, and non-ADS default restoration. All four targeted tests passed in final runs. All editor processes exited. git diff --check passed. Changes remain uncommitted; checkpoint61e63a4 is the pre-cleanup recovery point. FEAT120/113 acceptance remains separate and incomplete.
- Harness index check: both JSON files parse and FEAT122 exists exactly once in the completed index. Global uniqueness check detected a pre-existing FEAT080 duplicate across the active/archive indexes, confirmed in HEAD; unrelated historical records were preserved.
