# [FEAT-014] 环境危险区系统（火焰）

**创建日期：** 2026-06-07
**状态：** done
**Archive 文件：** `archive/FEAT-014-environment-hazard.md`

---

## 功能概述

基于接口的可扩展环境危险区系统。
- 任何实现 `IHazardSuppressor` 的 Actor（当前：`ARepairGunBullet`）进入危险区时，关闭 Niagara 特效和伤害 tick。
- 压制者 Actor 销毁后（RepairGun 泡泡 LifetimeAfterExpansion 到期），危险区自动恢复。
- 支持多个同类 Actor 同时压制（计数清零才恢复）。

---

## 继承与接口关系

```
IEnvironmentHazard（伤害接口，留空）
  └── AEnvironmentHazardBase（Abstract，核心逻辑）
        └── AFlameHazard（具体火焰，蓝图配置 Niagara 资产）
              └── BP_FlameHazard（蓝图实例）
        └── APoisonGasHazard（待扩展）
        └── ...

IHazardSuppressor（标记接口）
  └── ARepairGunBullet（膨胀泡泡，压制危险区）
  └── 未来其他"修复/抑制"工具
```

---

## 碰撞设计（压制检测关键）

| 组件 | ObjectType | 对 WorldDynamic | 对 GameTraceChannel1 | GenerateOverlapEvents |
|---|---|---|---|---|
| `BulletBase::CollisionSphere` | GameTraceChannel1 | ECR_Overlap（新增）| ECR_Block | true（新增）|
| `EnvironmentHazardBase::HazardZone` | WorldDynamic | — | ECR_Overlap | true |

两端均 ECR_Overlap + GenerateOverlapEvents = true → `HazardZone::OnComponentBeginOverlap` 触发。
检测 `OtherActor->Implements<UHazardSuppressor>()` → `Suppress(OtherActor)`。
绑定 `OtherActor->OnDestroyed` → `Resume(OtherActor)`。

---

## 新增文件

| 文件 | 作用 |
|---|---|
| `Public/Environment/Hazards/EnvironmentHazardInterface.h` | IEnvironmentHazard 接口（InflictDamage 留空） |
| `Public/Environment/Hazards/HazardSuppressorInterface.h` | IHazardSuppressor 标记接口 |
| `Public/Environment/Hazards/EnvironmentHazardBase.h/.cpp` | 抽象基类：HazardZone + HazardEffect + 压制逻辑 |
| `Public/Environment/Flame/FlameHazard.h/.cpp` | 具体火焰类（蓝图配置 Niagara 资产） |

## 修改文件

| 文件 | 变更 |
|---|---|
| `TheManTest.Build.cs` | 添加 `Niagara` 模块依赖 |
| `BulletBase.cpp` | CollisionSphere 添加 ECR_Overlap for WorldDynamic + SetGenerateOverlapEvents(true) |
| `RepairGunBullet.h` | 继承 `IHazardSuppressor` |

---

## 蓝图配置步骤

1. 编译 C++
2. Content/Environment/Flame/ 下创建 Blueprint，父类 `AFlameHazard`，命名 `BP_FlameHazard`
3. 打开 BP_FlameHazard：
   - 选中 `HazardEffect` 组件 → Details → Niagara System Asset → 选择火焰 Niagara 资产
   - 选中 `HazardZone` 组件 → Details → Sphere Radius → 调整覆盖范围
4. 将 BP_FlameHazard 拖入关卡
5. PIE 测试：RepairGun 泡泡接触火焰区 → 特效停止；泡泡到期销毁 → 特效恢复

---

## 扩展新危险区类型（未来）

1. 新建 C++ 类继承 `AEnvironmentHazardBase`
2. 重写 `InflictDamage_Implementation` 实现对应伤害 GE
3. 在编辑器创建蓝图子类，配置对应 Niagara 资产
4. 无需修改 RepairGun 或 BulletBase 任何代码

---

## 实现日志

- 2026-06-07：C++ 层完成（接口、基类、FlameHazard、BulletBase 修改、RepairGunBullet 接口添加）
- 2026-06-07：编译报错 `FOverlapResult` 未定义 → DamageTick 改用 `UKismetSystemLibrary::SphereOverlapActors`，消除对 `WorldCollision.h` 的依赖
- 2026-06-07：编译通过，PIE 验证通过

---

**完成标准全部满足日期：** 2026-06-07
**功能关闭日期：** 2026-06-07

## FlameJet visual replacement (2026-10-21)

The owner requested the newly imported `PPP_FlameJet_01` as the current flame Hazard effect. The source is `D:/Unreal Projects/TMIIR/Content/PyroParticlePack/Particles/PPP_FlameJet_01.uasset`; it is a Cascade `ParticleSystem`, not a Niagara system. Eleven assets in its material/texture dependency closure were migrated and renamed into `Content/Environment/Hazards/Flame/{Particles/Fire/Cascade,Materials/Jet,Textures/Jet}`. The temporary `/Game/PyroParticlePack` source path and both redirector files were removed after a cold registry and disk check.

`AFlameHazard` now owns `FlameJetEffect` (`UParticleSystemComponent`) with `P_FlameHazard_Jet`; the old inherited Niagara component is inactive, has no asset on `BP_FlameHazard`, and is hidden at runtime. `AEnvironmentHazardBase` routes suppress/resume through a virtual effect activation method; the flame subclass toggles the Cascade component, retaining the existing RepairGun foam and damage timer behavior. The Blueprint's visual scale is 0.4 after a transient rendered SpaceStation PIE comparison: scale 1 filled a nearby player's view, while 0.4 showed two distinct flame jets. The source Hazard radius remains 150 cm and two placed SpaceStation instances retain their original transforms, including the second actor's 90-degree pitch.

Development Editor and Game builds succeeded (`D:/UnrealWork/FlameJetHazard/EditorBuildFinal.log`, `GameBuildFinal.log`). Blueprint compiled and saved in UE 5.7. Cold Blueprint readback confirmed the old Niagara asset was null, the new component template resolves to the migrated ParticleSystem, and the materials' parent references load. Actual SpaceStation PIE confirmed both placed hazards had active visible Cascade components and inactive old Niagara components; rendered screenshots are under `Saved/Screenshots/WindowsEditor/FlameJetHazard-PIE*.png`. Perforce edit attempt timed out connecting to the server, so the scoped source and Blueprint were made locally writable and remain unsubmitted. Checkpoint ref: `refs/wip/before-flamejet-hazard`; scoped originals: `D:/UnrealWork/FlameJetHazard/checkpoint/`.

Final cold UE 5.7 readback loaded SpaceStation and confirmed `BP_FlameHazard` and `BP_FlameHazard2` both use `P_FlameHazard_Jet` at scale 0.4, both have null old Niagara assets, and `/Game/PyroParticlePack` has zero registered assets. Evidence: `D:/UnrealWork/FlameJetHazard/final_cold_report.json` (`checks_ok=true`). The transient scale-0.4 gameplay screenshot was inspected at `Saved/Screenshots/WindowsEditor/FlameJetHazard-PIE-Scale04.png`. The editor was closed after the cold check; no level placement was saved or changed.

## Assignable flame switch child (2026-10-21)

`BP_FlameHazardSwitch` is a child of `BP_ToggleSwitch` at `/Game/Actors/Interactables/ToggleSwitch/Blueprint/`. It preserves the modeled `SM_UpDownSwitch` and adds `UFlameHazardSwitchComponent`. On each placed switch instance, select `FlameHazardSwitch` in Details, assign one or more placed `BP_FlameHazard` actors to `TargetFlames`, and set `bStartsOn`. E toggles all selected targets and updates the interaction prompt. The original `BP_ToggleSwitch` stays generic and actionless.

`AEnvironmentHazardBase::SetHazardEnabled` separates manual switch state from RepairGun suppression. The effective state drives both the damage timer and the virtual visual effect hook. Foam removal restores a hazard only if its switch remains on. `AInteractableBase` broadcasts `OnInteracted` from its normal E interaction path, allowing the specialized component to react without changing generic switch behavior.

Checkpoint `refs/wip/before-flame-hazard-switch` with scoped originals under `D:/UnrealWork/FlameHazardSwitch/checkpoint/`. Development Editor and Game builds succeeded (`EditorBuildFinal2.log`, `GameBuild.log`). `TheManTest.Environment.FlameHazardSwitch` PIE automation passed with zero errors (`ReportFinal/index.json`, Success with unrelated existing warnings), covering two assigned flames, off/on/off E interactions and effective hazard state. Cold commandlet loaded the child Blueprint, its parent reference and static mesh. No switch placement was saved because target placement was not specified. No Perforce submit or Git push.
## Switch-bound tagged UE lights (2026-10-21)

Owner clarified the lamp belongs to the switch power state, not to each Flame Hazard's effective fire/suppression state. `UFlameHazardSwitchComponent` now exposes instance-editable `LightActorTag`. Give the placed switch a unique tag value in its `FlameHazardSwitch` component, then add that same value to the Actor Tags of any Point Light, Spot Light, Rect Light, or light Blueprint actors controlled by this switch. On E, the component sets every tagged actor's `ULightComponentBase` visibility to the same on/off state as the selected flames' manual enable state. An untagged light is untouched. Foam can still suppress a powered flame; it does not cut switch power or its lamps.

The switch resolves matching light components once on the first state application and caches weak references; if none are found it retries on later state changes. Use distinct tags when switches must control distinct lamp groups. No level light or switch actor was placed or tagged in this change. Development Editor/Game builds succeeded (`D:/UnrealWork/FlameHazardSwitch/EditorBuildLights.log`, `GameBuildLights.log`). Focused TestMap PIE automation passed (`LightsReport/index.json`), verifying tagged light off/on/off alongside two Flame Hazards while an untagged Point Light remains lit. Scoped pre-write backup: `D:/UnrealWork/FlameHazardSwitch/light-checkpoint/`; local WIP ref `refs/wip/before-flame-switch-lights`. No Perforce submit or Git push.

## MainGame single-jet height (2026-10-21)

The owner clarified the floor-to-ceiling span in the latest system screenshot and requested the original single jet in `Lvl_MainGame`, shorter than that storey. The uncommitted triangular three-jet C++ prototype was restored byte-for-byte from `D:/UnrealWork/FlameTriangularJet/checkpoint/`; its temporary inspection test was removed. The Blueprint's 0.4 default scale and SpaceStation instances were left intact. The sole placed MainGame `BP_FlameHazard_C_1` is at `(4181.33, 992.80, 195.00)`. Visibility traces at that XY found the floor at Z170 and the ceiling underside at Z525, so the emitter has about 330 cm to the soffit. Development Editor build passed after UHT regeneration. An initial uniform scale `(0.16, 0.16, 0.16)` was saved and inspected in rendered MainGame PIE (`Saved/Screenshots/WindowsEditor/main-flame-016-pie*.png`), but it made the jet too thin. Level backup: `D:/UnrealWork/FlameTriangularJet/checkpoint/Lvl_MainGame-before-flame-height.umap`. No Perforce submit or Git push.

On 2026-10-22 the owner correctly found the uniform 0.16 scale too thin. The MainGame instance was then changed to `(0.40, 0.40, 0.25)` to retain its horizontal width. A front-view PIE screenshot was incorrectly judged to fit under the ceiling; the later side-view screenshot disproved that assessment. This component-only scale is superseded by the duplicate-asset solution below. Earlier rendered evidence: `Saved/Screenshots/WindowsEditor/main-flame-wide-025.png`.

The owner's next side-view screenshot demonstrated that the above component-only scaling still allowed many particles through the ceiling. The earlier visual verdict was wrong because it sampled a low frame from an angle where the ceiling was offscreen. An editor-only diagnostic (removed after use) inspected all three Cascade emitters: source life 1 s/upward speed 600 cm/s; tail life 0.3-0.5 s/upward speed 2000-3000 cm/s, spawn cylinder offset Z 0-100, size growing to 1.5x; sparks have downward acceleration -1600 cm/s². Component transform scaling alone did not constrain the source/tail movement.

MainGame-only asset solution: duplicate the original asset to `/Game/Environment/Hazards/Flame/Particles/Fire/Cascade/P_FlameHazard_Jet_LowCeiling`, set source upward velocity to 130 cm/s, tail upward range to 180-220 cm/s and tail spawn offset Z to zero at both LODs, rebuild the raw distributions, and save the duplicate. The placed MainGame `BP_FlameHazard_C_1` initially referenced this duplicate at XYZ scale `(0.33, 0.33, 0.40)`; see the later height adjustment below. The Blueprint default continues to reference the untouched original `P_FlameHazard_Jet` at `(0.4, 0.4, 0.4)`; original asset hash matched its pre-write backup. MainGame side-view rendered PIE frames `Saved/Screenshots/WindowsEditor/main-flame-final-candidate-{1,2,3}.png` show the broad single flame with a visible gap below the ceiling. The level was saved and cold reopened with one placed hazard and the expected template/scale. Final Development Editor build passed after removing the diagnostic test. Scoped backups: `D:/UnrealWork/FlameCeilingFix/checkpoint/`. No Perforce submit or Git push.

At owner request the MainGame instance's Z scale was raised from 0.40 to 0.60 (1.5x) while X/Y stayed 0.33. Rendered side-view PIE at `Saved/Screenshots/WindowsEditor/main-flame-1p5-height-{1..7}.png` was checked over seven frames; the flame remained visually below the ceiling in this sequence. The map was saved, switched to TestMap, reopened and read back with the low-ceiling template and `(0.33, 0.33, 0.60)`. The placed `BP_FlameHazardSwitch_C_1` has an empty `TargetFlames` array as of this readback; `TargetFlames` is the `EditInstanceOnly` array in the `Flame Switch` category of its `FlameHazardSwitch` component. `LightActorTag` selects lamps, and `bStartsOn` selects initial power state. Scoped pre-height map backup: `D:/UnrealWork/FlameCeilingFix/checkpoint/Lvl_MainGame-before-1p5-height.umap`. No P4 submit or Git push.

The owner could not find `Target Flames` in Details. A full editor screenshot showed the placed `BP_FlameHazardSwitch (Self)` selected with the `target` filter, which searches actor properties rather than its `FlameHazardSwitch` component. The component is in the Details component tree below the initial visible items and must be selected to show `Flame Switch > Target Flames`. Because this map has exactly one placed switch and one placed flame, the switch's instance `TargetFlames` array was assigned to `BP_FlameHazard_C_1`. The map was saved, switched away to TestMap, reopened, and the reference read back correctly. The prior empty-array observation is superseded. Scoped backup: `D:/UnrealWork/FlameCeilingFix/checkpoint/Lvl_MainGame-before-switch-bind.umap`.

## Flame gameplay damage (2026-10-22)

The owner reported that visible fire dealt no damage. `AFlameHazard::InflictDamage_Implementation` was an empty TODO despite the base class querying Pawn overlaps every `DamageInterval` (0.5 s). It now applies the native instant `UGE_FlameDamage` through the target's AbilitySystemComponent. The GE subtracts from `UTheManAttributeSetBase::Health` using `Data.Damage` as a SetByCaller magnitude, so existing health clamping, HUD/death and debug invulnerability remain in the standard GAS path. Default `DamagePerTick` is **50** and is editable per placed flame; zero disables damage. Only authoritative instances apply damage. `FireDamageEffectClass` is exposed for a future Blueprint GE subclass and Gameplay Cue configuration; no cue is bound yet.

Development Editor build succeeded. `TheManTest.Environment.FlameDamage` PIE automation passed with zero errors: the native default removed 50 health from an Infiltrator, and a per-instance 7-point override removed 7 more. Report: `D:/UnrealWork/FlameDamageFix/report-final/index.json` (seven unrelated existing test-map warnings). Scoped pre-write backup: `D:/UnrealWork/FlameDamageFix/checkpoint/`. No Perforce submit or Git push.

## Main power switch and GameState flag (2026-10-22)

The owner designated the existing `BP_FlameHazardSwitch` as the main power switch. Its `UFlameHazardSwitchComponent` still controls its instance-assigned `TargetFlames` and light actors selected by `LightActorTag`, while `SetSwitchOn` now writes the same on/off state to `ATheManGameStateBase::bIsPowered` after updating flames and lights. This applies at BeginPlay from `bStartsOn` and on each E interaction; turning off resets the flag. The GameState exposes Blueprint-readable `IsPowered` and callable `SetPowerOn`, and the switch component's editor category/display name now reads `Main Power` / `Main Power Switch`. The existing Blueprint asset path is retained.

Scoped backup: `D:/UnrealWork/MainPowerSwitch/checkpoint/`; local Git WIP ref `refs/wip/before-main-power-gamestate`. Development Editor build passed (`D:/UnrealWork/MainPowerSwitch/editor_build_final.log`). Focused `TheManTest.Environment.FlameHazardSwitch` PIE automation passed with Success, asserting GameState false/on/false alongside two flames and a tagged lamp (`TestReport2/index.json`). Cold commandlet loaded the Blueprint and component. After the owner-approved remote map plus local vent merge, `Lvl_MainGame` has 1858 actors and no placed `BP_FlameHazardSwitch` or `BP_FlameHazard`; the Blueprint component's `bStartsOn` default remains true, but it cannot affect this map until an instance is placed and configured. No map, Blueprint asset, Perforce submit or Git push in this change.

## Main power naming migration (2026-10-22)

The owner required the asset and component names to match their new role. The Blueprint moved in Unreal Editor from `BP_FlameHazardSwitch` to `/Game/Actors/Interactables/ToggleSwitch/Blueprint/BP_MainPowerSwitch`; its SCS component instance was renamed from `FlameHazardSwitch` to `MainPowerSwitch`. The native class and source files are now `UMainPowerSwitchComponent` / `MainPowerSwitchComponent.h/.cpp`; its reflected methods are `SetPowerOn` and `IsPowerOn`, and its private state is `bPowerOn`. The focused test moved to `Private/Actors/ToggleSwitch/Tests/MainPowerSwitchTests.cpp` with path `TheManTest.Interaction.MainPowerSwitch`. `DefaultEngine.ini` has class/function CoreRedirects for older native references. The old Blueprint asset path and source files are absent; the renamed Blueprint has one `MainPowerSwitch` component of the new native class and still inherits `BP_ToggleSwitch`.

Scoped original files and asset are under `D:/UnrealWork/MainPowerSwitch/rename_checkpoint/`, with WIP ref `refs/wip/before-main-power-switch-rename`. Development Editor build passed (`rename_build.log`); cold Unreal editor migration compiled/saved the Blueprint and a fresh PIE run passed the renamed automation test with zero errors (`RenameTestReport/index.json`, seven unrelated TestMap warnings). Asset registry found no incoming package referencers before rename. The current MainGame remains unchanged and has no placed switch/flame actor. No P4 submit or Git push.
## Main power initial state for powered doors (2026-10-22)

The new SpaceStation powered door must show `No power` until the main breaker is enabled. `UMainPowerSwitchComponent` now defaults `bStartsOn` and its private initial `bPowerOn` to false. The existing `BP_MainPowerSwitch` component inherits this value: a cold editor instance readback reported `starts_on=false`, `power_on=false`; level instances can override `Starts On`. The focused main-power PIE test now asserts off at spawn, then verifies its existing flame/light/GameState toggle behavior. The change is recorded with the new door in FEAT-302 archive and `Documentation/Interaction/PoweredDoor.md`.

## Apparent empty-space blocker near switch (2026-10-22)

The owner reported being stopped in apparently empty space near the main power switch. An isolated PIE instance of BP_MainPowerSwitch had no Pawn-channel blocking trace at its mesh center or 40, 80, 120, and 200 cm to the side; InteractionRange responds to Pawn with overlap, not block. The focused test now asserts these properties and passes. The human player capsule was enlarged to radius 46 cm on possession by an earlier near-wall camera fix; that can make a narrow opening look clear while its 92 cm collision diameter does not fit. The minimum possession radius is now 40 cm; BP defaults stay 34 cm. Saved MainGame currently has no placed switch, so no scene-specific obstruction could be traced around the owner's unsaved placement. See FEAT-304 archive for camera/arms interaction and validation. No P4 submit.

## MainGame spawn breaker, flame and light placement (2026-10-22)

After syncing depot `Lvl_MainGame.umap` #5 and retaining the owner's local RobotVent merge, placed `Spawn_MainPowerSwitch` near PlayerStart, three `Spawn_FlameHazard_01/02/03` actors along the corridor beyond the door, and `Spawn_PowerIndicatorLight`. The breaker component's `TargetFlames` array references all three flames. Its `LightActorTag` and the PointLight actor's `Actor Tags` are both `SpawnPowerCircuit`; `StartsOn` is false. Each flame uses `P_FlameHazard_Jet_LowCeiling` at relative scale (0.33, 0.33, 0.60), and DamagePerTick 50. The normal `Spawn_ToggleSwitch` has the existing switch mesh assigned on this level instance; it exposes E interaction but has no bound effect. See `Documentation/Interaction/MainGameSpawnInteractables.md` for actor labels and editor setup.

Cold map reload verified all references and defaults. Live MainGame PIE began with GameState power false, three flames off and tagged light off. Directly invoking the breaker's `Interact` turned GameState, all three flames and the light on; a second call turned them all off. Rendered PIE showed a fire column below the corridor ceiling. No new C++ or Blueprint asset changes were made for this placement. The map is opened for Perforce edit, the setup document for add; neither was submitted. Backup and logs: `D:/UnrealWork/MainGameSpawnInteractables/`.

## Imported toggle-switch collision blocked the MainGame spawn (2026-10-22)

The owner reported that the apparently empty area to the right of PlayerStart could not be crossed. The prior static map AABB inventory had incorrectly suggested only visible walls and props, because it excluded the static mesh asset's actual cooked simple-collision geometry. A real MainGame PIE 40 cm Pawn capsule sweep from `(510,630,93)` hit `Spawn_ToggleSwitch` at distance zero with `bStartPenetrating=true` in every one of 24 directions. The player pawn was displaced to `(510,-64.1,92)` on spawn. The two placed switches both use `SM_UpDownSwitch`. Its visible mesh is approximately 34 x 28.2 x 48 cm, while the imported single generated convex collider spans approximately 3400 x 2820 x 4800 cm in local coordinates, roughly a 100x scale mismatch.

The shared static mesh asset now has `BodySetup.CollisionTraceFlag = CTF_USE_COMPLEX_AS_SIMPLE`, so collision follows its actual visible triangles. The large generated convex hull remains stored but is no longer the active collision. The `InteractionRange` overlap Box is unchanged, so E interaction is preserved. The asset was backed up at `D:/UnrealWork/MainGameSpawnInteractables/checkpoint/SM_UpDownSwitch-before-collision-fix.uasset` (SHA256 `1404495C9350AC78A8486585484C69E214B3764429B1A105D34CEB71F535A1DA`). Cold MainGame PIE after the asset change spawned the pawn at `(510,630,90.15)`, found no early hit in 24 capsule sweep directions, and hit the intended visible door/walls/switch only at their actual distance. New `TheManTest.Interaction.MainGameSpawnClearance` regression test asserts the spawn and clearance, and it plus `TheManTest.Interaction.MainPowerSwitch` passed after a Development Editor build. Evidence: `D:/UnrealWork/MainGameSpawnInteractables/spawn_probe_pie.log`, `spawn_probe_pie_after.log`, `spawn_clearance_test.log`. Mesh asset and regression test are opened for Perforce add; no submit.

## MainGame flame height doubled (2026-10-22)

The owner asked to double the three placed flames' height. Before editing, a cold map inventory found 1966 actors versus the previous 1967; `Spawn_ToggleSwitch` was the only missing actor, so its owner-side removal was preserved. A scoped map checkpoint is at `D:/UnrealWork/MainGameSpawnInteractables/checkpoint/Lvl_MainGame-before-flame-height-double.umap` (SHA256 `8957A6E5FB0527373AE4796C5D857881979F85307A5B370E538763434A72556B`). Set only each `Spawn_FlameHazard_01/02/03` instance `FlameJetEffect` relative Z scale from 0.60 to 1.20. X/Y remain 0.33; the low-ceiling Cascade template, 50 damage per 0.5-second tick, hazard sphere, total power wiring and BP defaults are unchanged. The saved map cold-reloaded with 1966 actors and all three Z scales at 1.20.

In rendered MainGame PIE, activating `Spawn_MainPowerSwitch` enabled all three taller flames. The inspected first-person screenshot `Saved/Screenshots/WindowsEditor/flame-height-double-game.png` shows the nearest fire column reaching close to the corridor beam; it did not visibly extend through the ceiling in that frame. PIE was stopped, and the Editor remains on MainGame. `Documentation/Interaction/MainGameSpawnInteractables.md` now describes the current six-actor setup and height. No Perforce submit.

## Actors migration and intermittent spray (2026-10-23)

Owner approved moving the entire old Environment directory into Actors and removing the old directory after references are updated. The original `BP_FlameHazard` is retained, not replaced. The nine native hazard files moved from `Source/TheManTest/{Public,Private}/Environment` to `Actors/Hazards`, with concrete flame files under `Actors/Hazards/Flame`; both old native Environment directories are gone. All 54 flame asset packages moved via Unreal AssetTools from `/Game/Environment/Hazards/Flame` to `/Game/Actors/Hazards/Flame`, retaining the original Blueprint and class. Particle systems are under `Effects/Fire`; other assets remain grouped by `Blueprint`, `Materials`, and `Textures`.

`AFlameHazard` now has editable `SprayDuration` and `PauseDuration`, both defaulting to 2 seconds. When powered and unsuppressed, it starts spraying immediately, hides the Cascade jet and stops the damage loop during each pause, then resumes. Power off and foam suppression cancel and reset the cycle; `InflictDamage` also refuses direct calls during a pause. The old Niagara component remains disabled. Development Editor builds succeeded. Cold Blueprint compilation/save resolved the moved class, 2/2-second defaults, and moved Cascade template. Focused NullRHI PIE tests passed: `TheManTest.Environment.FlamePulse`, `FlameDamage`, and `TheManTest.Interaction.MainPowerSwitch`. Evidence: `D:/UnrealWork/FlameHazardReorg/`.

Cold map readback loaded MainGame #6 with three original flame instances using the moved low-ceiling jet, and SpaceStation with two original flame instances using the moved standard jet; all inherited 2/2-second defaults. The two SpaceStation external actor packages were resaved to update references. The `Lvl_MainGame.umap` #6 package is still exclusively checked out by `aniyang`, so its references cannot yet be saved. Two redirector files for `BP_FlameHazard` and the low-ceiling Cascade remain in `Content/Environment`; the old Content directory must be removed after the map lock is released and the map is resaved. Owner is contacting the lock holder. Scoped changes are in pending P4 CL 122048, not submitted; prior Spider Tank CL 122047 remains separate. Local safety refs: `refs/wip/before-environment-actors-flame-migration` and `refs/wip/before-flame-external-actor-resave`. No Git push.

## MainGame #7 intake and old directory completion (2026-10-23)

`aniyang` submitted MainGame revision #7 in P4 CL 122049 and released its exclusive checkout. Local #6 was clean by `p4 diff -se`; its SHA256 backup is `D:/UnrealWork/FlameHazardReorg/checkpoint/Lvl_MainGame-before-CL122049.umap`. Synced all six files in that CL: the map and five new observation-window/pustule meshes. Created another scoped safety ref `refs/wip/before-flame-maingame-resave-cl122049` and backed up the exact #7 map before saving. Resaved the #7 map through Unreal to point to the moved flame Blueprint and low-ceiling Cascade. The saved map cold-loads with 2956 actors and all three original `Spawn_FlameHazard_*` actors; SpaceStation cold-loads with its two original flame actors. All five teammate meshes remain referenced by the map: one frame, one glass, and four instances of each lobe mesh.

After a cold reference scan found zero referencers for either old redirector, Unreal removed the Blueprint redirector. The remaining 1505-byte Cascade redirector was still present on disk after Unreal deletion calls; its class was `ObjectRedirector`, it had zero package referencers, and its exact bytes were backed up before removing that orphan file. All old asset paths are marked for P4 delete, and the empty `Content/Environment` tree was removed. Final cold audit: 54 moved packages, zero redirectors in the new owner directory, zero assets in the old registry, no old disk directory, and both maps load their original flame instances from the new class. Evidence: `D:/UnrealWork/FlameHazardReorg/final-audit.json` and `teammate-mesh-report.json`. P4 CL 122048 remains pending (54 adds, 54 deletes, nine native moves, seven edits); no submit or Git push. Spider Tank CL 122047 remains separate.

## Perforce delivery (2026-10-23)

Owner requested submission. Submitted the scoped Environment migration and pulsing flame changes from pending CL 122048 as Perforce CL 122053 (133 files). Server `describe -s` confirms submission. The separate Spider Tank distance-recall CL 122047 remains open with its three files. No Git push.

## Actors category follow-up (2026-10-23)

After the owner pointed out that the prior migration only classified Hazards, approved an Actors-wide purpose hierarchy. Existing `Hazards` and `Interactables` assets remain; seven other packages moved through Unreal AssetTools: DestructibleCube and ExplosionGlassWindow to `Actors/Destructibles`, PatrolPoint and WindowVaultFrame to `Actors/LevelObjects`, and the editor GunFraming authoring Blueprint to `Actors/Tools`. Nine SpaceStation external actors, two TestMap external actors, and MainGame #8 were opened and resaved for the moved references. Twenty-three native files were moved with Perforce moves to the matching Actors categories. The test-only `WorldPersistenceTestDoor` native class moved to `Core/Persistence/Tests` and has no Content Blueprint. The separately pending Spider Tank test still includes `Actors/RobotVentCover.h`; a forwarding public header keeps that changelist independent until it can be updated. Team map: `Documentation/Actors/ActorDirectoryLayout.md`.

Development Editor build succeeded; six moved Blueprints compiled and saved. Cold audit found all seven new packages and direct dependencies in the original maps/external actor packages, zero old registry packages or disk directories, and loadable MainGame (2956 actors), SpaceStation (1845), and TestMap (13). Persistence (2), PoweredDoor, MainPowerSwitch, Scan GoldHighlightLifecycle, PatrolWeaponDummy, and SpiderTankVent PIE tests succeeded. ExplosionInteraction had two NullRHI timing/VFX assertions fail; the rendered RHI rerun succeeded. Evidence and backups: `D:/UnrealWork/ActorsCategoryReorg/`; local safety ref `refs/wip/before-actors-category-reorg`. All 89 delivery paths were submitted as P4 CL 122066 after the owner requested MainGame release; server readback confirms MainGame head/have #9 and no client checkout. CL 122047 remains separate. No Git push.
