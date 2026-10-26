> Current runtime authority (2026-09-19): FEAT-077 arms-only weapon layers and body upper-body Copy Pose, plus FEAT-091 CharacterMesh0 dynamic hidden shadow, supersede the legacy layout/shadow setup instructions below. Do not disable CharacterMesh0 CastShadow/CastDynamicShadow/CastHiddenShadow or reactivate the deprecated empty shadow meshes.

## FEAT-298 window-frame traversal (in progress, 2026-10-17)

`AFPSCharacterBase::JumpOrVault` uses a placed `AWindowVaultFrame` candidate only when paired body/first-person montages are assigned. The frame validates a forward sill sweep, landing and a clear capsule path sampled from the body montage's extracted root motion. Invalid candidates fall back to ordinary Jump. During traversal the character locks movement and weapon inputs, lowers the shared viewmodel for 0.22s, hides only the weapon actor while both montages play, then restores capsule, weapon, root-motion modes and movement. The first-person arms mesh aligns to the body mesh for hand contact and blends back to its authored camera framing. A short camera arc moves back up to 50cm, lowers eye height by 65% of the root rise, and looks down up to 45 degrees during the crossing; the original camera/control rotation mode restores afterward. No equipment `Unequip` is performed. Final animation assets remain external pending visual approval; see FEAT-298 archive.

# 角色基类（Character Base）

## FEAT224 foam traction (2026-09-30)

AFPSCharacterBase overrides SetBase and OnMovementModeChanged to update foam traction immediately. Walking on ARepairGunBullet.BulletMesh snapshots GroundFriction, BrakingFriction and BrakingDecelerationWalking and doubles them once. Leaving/jumping restores the exact snapshots; switching between foam bases does not stack multipliers. Existing bUseSeparateBrakingFriction and all movement speeds remain unchanged. No actor Tick polling or physical-material friction dependency. Foam alone has a60degree walkable slope increase; ordinary floor angle is unchanged.

- 2026-09-05爆炸反馈修正：爆炸CameraShake只旋转、不平移，避免后处理视点离开HeadCamera而ArmsViewMesh仍留在原组件位置造成穿模。未修改此处相机/手臂组件层级、静态构图或近距材质；普通武器穿墙不属于此项修复。参数在Explosion GC，详见arch10。


**何时读取：** 修改角色通用行为、组件布局、GAS 初始化流程、增加所有角色共享的属性或函数时。

> ⚠️ **唯一玩家基类为 `AFPSCharacterBase`。旧 `ATheManCharacterBase` 双骨骼系统已于 FEAT-041 删除（不再有"保留弃用"代码）。**

**唯一玩家基类（FPS）：**

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Characters/CharacterBase/FPSCharacterBase.h` | HeadCamera / **ViewmodelRoot / ArmsViewMesh**（FEAT-042/077 独立 FP viewmodel）/ **BodyRoot / ShadowBodyMesh / LegsMesh**（FEAT-038 三件套，含 Getter）/ EquipmentManager；**`GetArmsMesh()` 返回 `ArmsViewMesh`**，武器挂载/开火蒙太奇/装备渲染走相机子级 FP 手臂；`GetMesh()` 是 GASP/MM 宿主，驱动身体/影子/腿；`ArmsHiddenSections` / `LegsHiddenSections`（EditDefaultsOnly 材质段隐藏）；CharacterData / InitGEClass；通用 `DefaultAbilityClasses`；`PrimaryFire()` / `SecondaryFire()` / `Reload()` 发送对应 Gameplay Event；`IsSprinting()`（冲刺键状态，停步走/跑档用）|
| `Source/TheManTest/Private/Characters/CharacterBase/FPSCharacterBase.cpp` | 组件挂载：**HeadCamera←RootComponent(capsule)**，**ViewmodelRoot←HeadCamera，ArmsViewMesh←ViewmodelRoot**。`ArmsViewMesh` 构造默认构图为 Location `(-14.766994,-4.322017,-134.599387)`、Rotation `(Roll=5.752239,Pitch=0.077753,Yaw=-114.908449)`；C++ 不在 BeginPlay/Tick 覆盖该静态 Transform，具体角色 BP 仍可覆盖。`ViewmodelRoot` 保持零 Location，Tick 只写实际速度驱动的冲刺 Pitch；不用它调静态构图。ArmsViewMesh 独立求值，GetMesh() 在 Body AnimBP 内合成其上半身 Pose。`ShadowBodyMesh` / `ShadowUpperBodyMesh` 为弃用空组件；唯一完整影子来自 `GetMesh()` 的 CastHiddenShadow。|

**FEAT-077：** `ArmsViewMesh` 独立运行第一人称 AnimBP，并作为 `CharacterMesh0` 上半身 Copy Pose 来源。构造时 `CharacterMesh0` 添加 `ArmsViewMesh` Tick prerequisite；完整身体 AnimBP 保留自己的 root/pelvis/腿部 locomotion，只在 `spine_01` 以上混入 Arms 局部骨骼 Pose。唯一完整影子仍由 `CharacterMesh0` 的 CastHiddenShadow 产生。

**第一人称手臂近距离裁切（session236，session267 参数统一）：** `SKM_MaintenanceWorker_FirstPersonArms` 唯一材质槽使用 `/Game/Characters/MaintenanceWorker/FirstPerson/Materials/MI_MaintenanceWorker_FirstPersonArms`。其专属 Masked 父材质计算 `Distance(CameraPositionWS, AbsoluteWorldPosition)`，以 `(Distance - Proximity Clip Radius) / Proximity Fade Width` 经 Saturate 和引擎 `DitherTemporalAA` Material Function 输出到 Opacity Mask：贴近相机的肘部裁掉，较远的手和前臂保留。实例参数为 40cm / 8cm；只影响第一人称 Arms，不影响完整身体、影子或武器材质。

**第一人称原材质与近距离裁切（session263-264）：** session263 的临时统一灰模方案已被替代。手臂从外部原 VFXPack 项目正式迁入 `MI_Placeholder_Lambert_INST`、`M_Placeholder_Lambert_Master`、`MF_Disintegration` 与 Noise 贴图；本地专属副本保留原参数 `Color=0.802083 / Metallic=0.5 / Specular=0.5 / Roughness=0.8`、Noise/Dissolve 图，并在原 Opacity Mask 后乘入 40cm / 8cm 相机距离抖动裁切。腿部不使用手臂材质；`M_MaintenanceWorker_FirstPersonLegs` 由项目现有 `M_UE4Man_Body` 复制并只追加 55cm / 12cm 裁切，完整身体和影子的原材质不变。

**MaintenanceWorker 第一人称 Skeleton（session241）：** `SKM_MaintenanceWorker_FirstPersonArms` 已改为直接使用 `/Game/Characters/MaintenanceWorker/FirstPerson/Meshes/SKEL_MaintenanceWorker_ReferenceArms`，与 `ABP_MaintenanceWorker_FirstPerson`、`AS_MaintenanceWorker_FP_*` 和 `BS_MaintenanceWorker_FP_WalkRun` 统一。旧 `SKEL_MaintenanceWorker_FirstPersonArms` 暂未删除；不要再为该旧 Skeleton 新增动画。

**扫描组件（FEAT-104，2026-09-17）：** `UScanEffectComponent` 由玩家基类创建，默认使用 `MPC_ScanEffect` 驱动 `Mat_Outline_library` 场景轮廓扫描。`MaxScanRadius` 同时写入 MPC `ScanRadius` 并用于目标检测，默认3000cm；材质不再独立使用30000cm半径或Frac循环。`ScanDuration` 默认2s扩张，结束后保持 `ScanTime=1`、`ScanAlpha=1`，直到再次按键调用 `RetractScan`；没有自动淡出或超时。保持期间持续Tick更新相机相对 `ScanOrigin`，避免移动视角拖动扫描世界中心。目标高亮独立保留，`RetractScan`／重启／EndPlay立即清理。额外地形Decal创建与TerrainOverlay属性已移除；旧 `M_InfiltratorScanTerrainAdaptive` 资产保留但组件不再加载或使用。

**FEAT-038 三件套组件布局（详见 archive/FEAT-038）：**

```
Capsule(root) [bUseControllerRotationYaw=true, bUseControllerRotationPitch=FALSE]
├─ HeadCamera = 挂 capsule 固定眼高(相对 Z≈+77)，bUsePawnControlRotation，稳定 gameplay 相机
│   └─ ViewmodelRoot = 默认 identity，C++ 只写冲刺动态 Pitch
│       └─ ArmsViewMesh = 蓝图全部静态构图 Transform + 武器挂载目标
├─ GetMesh()  = GASP/MM Leader+动画宿主+根运动源，OwnerNoSee，CastShadow=false
└─ BodyRoot (SceneComponent, 绝对旋转, Tick 每帧只取 Yaw → 直立；相对 Location=(0,0,0))
    ├─ ShadowBodyMesh = 已弃用空组件；完整影子直接由 GetMesh() CastHiddenShadow
    └─ LegsMesh       = Follower，只渲染腿材质段，OnlyOwnerSee，无影
```
- session89 起暂停原地转身方案：`bUseControllerRotationYaw=true`，Pawn 直接跟随 Controller yaw；`BodyRoot` 每帧直接使用 Actor yaw 并保持 Pitch/Roll 为 0。旧 `BodyVisualYaw` 滞后、固定 45 度转步和动画曲线驱动代码均已删除，后续转体作为独立方案重新设计。
- 三 mesh 须引用**同一 Skeleton**；几何分离只用材质段（`ShowMaterialSection`）/OpacityMask，**禁用 HideBoneByName**（会改共享姿势）。本项目实际用「物理拆 mesh」（Blender 拆 Arms/Legs，原整块当 Shadow），`ArmsHiddenSections/LegsHiddenSections` 留空。
- 蓝图侧：Legs 组件相对变换需与 GetMesh() 完全相同（当前 Location=(0,0,-90)、Yaw=-90），保证下半身根点与脚底影子起点重合。ShadowBody/ShadowUpperBody 不再持有 Mesh，避免编辑器显示旧副本而与 PIE 的权威 CharacterMesh0 不一致。
- 蓝图侧 **Mesh（GetMesh()）组件**：指定全身骨架 mesh + locomotion AnimClass + 相对 Transform；**Cast Shadow 取消勾选**（BP 勾选会覆盖 C++ 的 false，造成手臂形状影子与 ShadowBodyMesh 穿帮）。`OnlyOwnerSee` 已由 C++ 设——故编辑器视口（无 owner）看不见手臂，PIE 里可见，属正常。

**根运动历史决策（FEAT-039，已被 FEAT-042/session62 的独立 Viewmodel 方案取代）：**
Current arrangement supersedes the FEAT-039/session70 historical host: GetArmsMesh returns ArmsViewMesh; CharacterMovement owns locomotion. FEAT-077 links the weapon layer only to ArmsViewMesh; CharacterMesh0 copies the completed upper-body pose.
- 前提条件成立才安全：session47 相机已从 head 骨骼挪到 capsule → 手臂无人依赖，可合并。
- **根运动方案（详见 arch/12 + archive/FEAT-039）**：ABP 根运动模式与「输入驱动 locomotion」的取舍是本项目踩过的大坑，务必看 12。一句话：**`Root Motion from Everything` 会让动画接管移动、压制 WASD 输入**——铁律「原地 clip（idle/走跑循环/跳跃）Enable Root Motion 必须关，只有位移 clip（停步等）才开」；漏关一个原地 clip（尤其 idle）→ 移动卡死+抖动。
- **session62 FP viewmodel 方案修正（重要）**：相机仍挂 capsule 固定眼高、只用控制器旋转转向；FP 手臂不再绕肩部 `ArmsPivot` 手动俯仰，而是 `HeadCamera -> ViewmodelRoot -> ArmsViewMesh`。这样旋转支点就是相机原点，手臂屏幕位置不随 pitch 漂移。不要把相机挂到动画 head 骨骼下，也不要旋转 Character capsule pitch。后续相机/移动惯性优先叠到 `ViewmodelRoot` 或 `ArmsViewMesh`，不用 SpringArm。
- 第一人称视点横向/纵向微调不需要 SpringArm，继续保持 `Capsule -> HeadCamera -> ViewmodelRoot -> ArmsViewMesh`。`HeadCamera.RelativeLocation` 只定义真实观察点；`ViewmodelRoot` 必须留在相机原点，以保持原版 BodyRotator 的旋转半径；最终手臂/武器构图偏移由 `ArmsViewMesh.RelativeTransform` 持有。
- **FEAT-065 最终构图（session126）：** `ViewmodelRoot.RelativeLocation=(0,0,-7)`、`RelativeRotation=(0,0,0)`，HeadCamera 保持 110° FOV 与真实观察点不变。构造、OnConstruction、BeginPlay 均通过 `ApplyViewmodelFraming()` 应用，避免 BP 旧默认值覆盖；装备继续挂 `ArmsViewMesh` Socket。`TheManTest.Player.Viewmodel.FramingCapture` 会以稳定 Idle 帧执行 1920×1080 截图及层级/FOV/Transform/Socket 断言。
- C++ 基类速度默认（session47 调）：`WalkSpeed`=250 / `SprintSpeed`=550；`PitchMin`=-75 / `PitchMax`=40。FEAT-070 对 MaintenanceWorker 的实际 Walk/Jog 脚速逐帧标定后，`BP_MaintenanceWorker` 专属覆盖为 Walk=100 / Sprint=300 cm/s，与其 BlendSpace 的原速样本网格点精确一致；其他角色不得无依据照搬该动画专属值。
- FEAT-038 C++ 已完成（session43 编译通过，session47 改相机/俯仰/BodyRoot/速度）；身体 mesh 已物理拆好导入，角色 BP 三件套装配在蓝图侧进行。

> 旧 `ATheManCharacterBase` / `Infiltrator` / `MaintenanceWorker` / `TheExecutive` 及 `UTheManAnimInstanceBase` 已于 FEAT-041 删除，备份在 scratchpad/deprecated-char-backup-session43。

> **MCP 审计遗留项（2026-07-26）：** 活动资产 `BP_Infiltrator` 仍硬引用 `/Game/Characters/Infiltrator/Blueprint/BP_Infiltrator_Old`。当前仅确认依赖存在，尚未定位具体属性或图节点；在用户手动确认前不得自动删除或重写该引用。
## 2026-09-01 session265：第一人称材质与腿部邻近渐隐

- `ArmsViewMesh` 使用 UE 小白人 Mask/Normal 黑白灰分区材质，并保留 40cm/8cm 相机近距 `DitherTemporalAA`。
- `LegsMesh` 不使用相机距离裁切。角色运行时为腿材质创建 MID，并写入双臂上臂、前臂、中点、手部 8 个世界空间采样点；腿材质按最近手臂点以 32cm/18cm 进行局部抖动渐隐。
## 2026-09-01 session266：正式 UE4 Mannequin 材质源

- 第一人称手臂/腿部材质底稿位于 `/Game/Characters/MaintenanceWorker/FirstPerson/Materials/UE4Mannequin/M_UE4Man_Body_Source`，其 Mask、Normal、Logo 与 Material Layer 依赖均位于同目录树。
- 不得再使用 `Niagara_Stylized_Footsteps` 示例材质或项目内编译失败的旧 `M_UE4Man_Body` 作为第一人称专用材质底稿。
## 2026-09-01 session267：统一 Proximity 参数与右手单点

- 手臂和腿部材质实例统一暴露 `Proximity Clip Radius`、`Proximity Fade Width`。
- 腿部渐隐只读取 `ArmsViewMesh.hand_r` 的世界位置，通过 `Proximity Origin WS` 写入运行时 MID；不再计算左右双臂多点最短距离。

## 2026-09-01 session268：腿部双来源邻近渐隐

- 腿部材质同时计算像素到 `hand_r` 世界位置与 `CameraPositionWS` 的距离；两条归一化渐隐遮罩取 `Min`，所以靠近任意来源都会渐隐。
- 右手侧参数沿用 `Proximity Clip Radius/Fade Width`；相机侧独立使用 `Camera Proximity Clip Radius/Fade Width`，不得由 C++ 每帧重复写相机位置。

## 装备初始化职责（2026-09-04）

FPSCharacterBase 继续指定 EquipmentManager.AttachTargetMesh=ArmsViewMesh、初始化背包并预热初始姿势。首装与切装的延迟显示/VFX 统一归 EquipmentManager.QueueEquipPresentation，旧 PlayInitialEquipEffect 方法及角色定时回调已删除。

## 2026-09-16 temporary combat character framing

Executive/Infiltrator reuse Worker first-person assets and must also copy ViewmodelRoot SceneComponent transform. Current offset is(-6.153601,30.210103,-12.871626), now copied to both BPs. Do not reset to identity or historical -7cm. Rendered PIE verified root/arms/camera positions,90-degree FOV,anim class and RepairGun attached to ArmsViewMesh/GripPoint1 match across all3 characters. Lobby uses independent presentation settings.

## FEAT-091 dynamic body shadow parity

BP_FPSTheExecutive and BP_Infiltrator CharacterMesh0 now explicitly enable CastDynamicShadow like BP_MaintenanceWorker; this was missing from the temporary setup copy. Runtime body CastShadow/CastDynamicShadow/CastHiddenShadow/OwnerNoSee must all be true. ArmsViewMesh/LegsMesh CastShadow=false, old ShadowBodyMesh/ShadowUpperBodyMesh remain empty. Verified in actual TestMap PIE for all3 characters.

## FEAT-099 scan-owned highlight lifetime

ScanEffect.HighlightDuration removed. A scan requests indefinite Highlight interface activation and stores weak references to the actual Actor/component receivers. RetractScan stops all surviving receivers and clears detection state; starting a new scan first retracts the old one; EndPlay also retracts. HighlightComponent keeps its generic timed-call API for compatibility, but indefinite starts cancel any prior fade timer. ScanDuration still controls wave expansion, not highlight lifetime.

SpaceStation now includes existing Mat_Outline_library/M_Highlight postprocess blendables, plus Station_ScanCube near the user's current PlayerStart. /Game/Actors/Interactables/ScanCube/Blueprint/BP_ScanCube derives from InteractableBase and uses the plain engine cube/material. Under the current FEAT302 scan gate, it has no `UScannableInteractableComponent`, so it remains directly interactable and does not receive scan highlights. No fancy interactable-effect material. Source station grading and original actor transforms preserved.

MainGame's unbound `Level01_PostProcess_Exposure` also includes the existing `/Game/Characters/Infiltrator/Gameplay/Abilities/Scan/Materials/M_Infiltrator_Highlight` as a weight-1 postprocess blendable. The scannable RobotVent cover's CustomDepth/stencil-1 highlight needs this level blendable to render an outline; the scan component alone only marks the mesh.

TestMap Dev Panel `Props x4` fixture no longer has `UHighlightComponent` as of 2026-10-22. Its meshes remain query-collidable for disintegration and explosion tests but do not receive scan highlighting or interaction prompts. The production RobotVent cover remains scan-gated.


## 2026-09-17 LaserRifle GAS ADS

FEAT-107: AFPSCharacterBase binds SecondaryFire Started plus Completed/Canceled ReleaseSecondaryFire. BeginWeaponAim caches original weapon, initial FOV and viewmodel transform; update composes sight-relative ADS alignment after existing per-frame offsets, with30/20 enter/exit interpolation. Reject dead/unpossessed/unsupported weapons. Release, weapon switch, focus loss, death and unpossess cancel/restore camera, speed and aimed weapon; never assume current equipment is the old weapon during Unequip. CharacterBaseAnimInstance exposes WeaponAimPoseAlpha and recoil weight; LaserRifle owner template blends source Still over locomotion in.2s. Existing weapons stay ADS-disabled.

## FEAT-109/111 Lobby presentation reuse (2026-09-17)
Infiltrator BP_Infiltrator_Lobby reuses ALobbyCharacterBase and ULobbyPoseBlendAnimInstance without new C++. Native owner skeleton and externally finalized O7665 clips, Bip01-R-Hand gun attachment, two standing/two relaxed/one RifleIdle. Executive and Infiltrator each use one WeaponPresentations entry with owner Lobby thumbnail; Worker retains three. Widget invalid SizeBox slots collapse/restored per actual entry count. Gameplay character mesh/loadout is independent and unchanged by Lobby integration. See arch13 for final paths/verification.


## 2026-09-17 follow-up integration

FEAT-117: AFPSCharacterBase now owns camera-child WeaponFillLight (Point35lm/radius160cm, no shadows) and ForwardLight (Spot450lm/radius1400cm, inner28/outer58deg, shadows). Both default enabled, native editable component settings in each playable BP. No map/exposure/animation changes. Current3 BPs had no existing light components; constructor supplies new defaults.

ForwardLight on all three playable BPs references /Game/Characters/_Shared/Effects/Materials/M_PlayerForwardLightFalloff. Light Function = Saturate((Distance(WorldPosition, CameraPositionWS)-125)/100), suppressing close gun overexposure while lighting the forward scene. WeaponFillLight supplies the near gun light. Actual SpaceStation on/off/turn rendered checks passed.

2026-09-17 user tuning: ForwardLight default intensity450lm in native constructor and all three playable BPs. DevelopmentEditor build and Blueprint compile/save/property readback passed; existing packaged executable predates this tuning.


## FEAT128 approved Infiltrator pose (2026-09-18)

FEAT128: BP_Infiltrator.ArmsViewMesh overrides slot0 with Characters/Infiltrator/FirstPerson/Materials/MI_Infiltrator_FirstPersonArms, derived from existing Worker first-person material. Proximity Clip Radius12cm/Fade Width4cm replaces40/8 only for this character to retain the newly visible support forearm. Worker/Executive and shared mesh/material unchanged.


2026-09-18 Infiltrator-only user override: BP_Infiltrator WeaponFillLight and ForwardLight both intensity0 and visible=false (all2 character-attached lamps). Native35/450lm defaults and other playable BPs unchanged. Cold saved CDO and actual PIE verified; no runtime force toggle.


## FEAT-135 God Mode

AFPSCharacterBase owns UGodModeComponent under Characters/CharacterBase/Debug. It binds the controller-registered debug actions itself. Only a living locally controlled combat pawn with initialized PlayerState ASC can enable. Component snapshots collision/damage/movement/camera pitch, cancels Ability.Movement abilities, releases ADS/sprint/jump, then disables ordinary CharacterMovement (MOVE_None). Tick applies normalized camera-relative forward/right plus world-Z movement directly to the actor, without sweep or inertia. Camera, arms and equipment keep their existing hierarchy/transforms. RMB gates ordinary mouse look, temporary pitch range is +/-89 degrees. Speed defaults1000cm/s, clamped100..10000, retained across toggles on the current pawn.

A high-priority IMC_GodMode owns WASDQE/RMB and blocks jump/sprint. Modal UI clears pending flight input, pause does not advance flight. Last non-overlapping capsule location is tracked above KillZ; disabling inside geometry returns to the last still-valid point, otherwise refuses disable with UI feedback. Normal walking restores via Falling and automatic floor detection; original custom/flying modes restore as saved. UnPossessed/EndPlay forcibly remove context/tag and restore captured state. FellOutOfWorld and OnDeath are guarded while God Mode is active; ordinary game flow remains unchanged.


## FEAT-146 native Infiltrator gameplay (2026-09-23)
Infiltrator now uses its original lobby native target skeleton/fullbody plus owner FirstPerson/Meshes/SKM_Infiltrator_FirstPersonArms and Body/Meshes/SKM_Infiltrator_LowerBody. Arms/body run ABP_Infiltrator_FirstPerson and ABP_Infiltrator_Body under their owner Animations folders. This supersedes worker-asset reuse for Infiltrator only; Worker/Executive unchanged. Body retains locomotion/hidden shadow and copies native upper-body pose; legs follow body. Native graphs replace spine/hand names and omit independent left-hand sway to preserve two-hand contact. The two Infiltrator gun BPs use native first-person layers/montages under Characters/Infiltrator/Animations/FirstPerson/{LaserRifle,ScopedRifle}, attach to Bip01-R-Hand with independently measured offsets. Default loadout remains silver only. Shared templates and gun optical/gameplay properties unchanged. Finalized locomotion20clips and weapon16clips were adapted externally in TMIIR; no runtime/destination retargeting. Four native arm materials add16cm/8cm camera-distance dither clipping; original lobby/body materials unchanged. Editor authoring helper is Characters/Infiltrator/Editor/InfiltratorAnimationAssetLibrary; gameplay does not call it. See FEAT146 archive and external InfiltratorNativeWeapons evidence.


## FEAT-149 native role integration (2026-09-23)
Worker and Executive now use own native lobby skeleton/fullbody, SKM_<role>_NativeFirstPersonArms (forearms/hands) and SKM_<role>_NativeLowerBody. Fullbody remains hidden shadow caster; lower body follows body pose. Native arm weights/proportions retained; source lobby assets unchanged. See archive/FEAT-149-native-worker-executive.md and D:/Blender Projects/WorkerExecutiveNative for evidence.

FEAT151: Shipping builds disable Dev Panel, test-map travel, debug time skip, debug character switching and God Mode at input and implementation boundaries (UE_BUILD_SHIPPING). FPS display/sampling excluded; version/round/countdown retained. DebugGame/Development behavior preserved. Top-row0 maps IA_SkipTime for150s fast-forward in non-Shipping; lobby normal Tab selection unaffected. Serialized asset references retained for BP compatibility. Both Editor and Shipping compilation and focused editor tool/HUD check passed; no new package. Details archive/FEAT-151-shipping-developer-restrictions.md.


## FEAT163 current scan presentation
ScanEffect now exposes IsScanActive/GetCurrentScanRadius/GetScanOrigin and native start/stop delegates. It retains the existing fixed activation origin,2s expansion,3000cm max range and second-toggle lifetime. Opt-in interface/component detection also runs during hold. Infiltrator's native ScanPresentation disables only legacy ScanAlpha and renders migrated07 instead; other consumers default to legacy presentation. Highlight eligibility is unchanged and enemies are NOT automatically given HighlightComponent. This supersedes FEAT157/104 visual rendering, not generic opt-in highlight semantics.


## FEAT180 single scan owner (supersedes legacy scan layout)
FEAT180 completed2026-09-26. One InfiltratorScanPresentationComponent now owns scan state,2s expansion,3000cm radius,10Hz opt-in highlight detection/cleanup, source grid/return/audio and disabled smoke. Existing native class/subobject names intentionally retained to preserve BP settings. Removed generic ScanEffectComponent h/cpp and base attachment/getter, merged needed logic; eliminated legacy MPC driver and optional unused GAS hologram/sound/state code. GAS verifies Infiltrator owner and only toggles the component. Base tracks only its default-ability grant handles and cancels/clears them on re-possession/unpossess/EndPlay; weapon grants remain equipment-owned. Worker Scan grant removed; Executive unchanged empty config; Infil settings copied from captured old component. Three role BPs and Scan BGA compiled/saved. Removed only old Contours blendable from TestMap and SpaceStation PP actors, retained M_Infiltrator_Highlight. Deleted M_Infiltrator_ScanContours, M_Infiltrator_ScanTerrainOverlay, MPC_Infiltrator_ScanEffect,T_Infiltrator_Scan_MatCap after refreshed AssetRegistry proved no outside referencers; empty Scan/Textures removed. Disabled smoke and previously retained hologramUI assets preserved.

## FEAT216 jump air steering (2026-09-28)
All three playable Gameplay/Blueprint/BP_<role> CharacterMovement defaults use AirControl=.65 and AirControlBoostMultiplier=1.0. Airborne WASD uses existing Move/AddMovementInput; no new input or runtime overrides. Falling friction and braking remain0 to retain inertia; JumpZVelocity420 and GravityScale1 unchanged. Tune gameplay BPs, not lobby pawns or animation root motion.

## FEAT219 Infiltrator ADS zoom and optic size (2026-09-29)
AFirearm exposes opt-in ADSZoomMultiplier (default0 uses legacy ADSFieldOfView) and ADSLookSensitivityMultiplier (default1). Character UpdateWeaponAim computes 2*atan(tan(originalFOV/2)/zoom) for positive zoom and blends with existing WeaponAimAlpha; Look multiplies both axes by lerp(1,weapon sensitivity,alpha), without overwriting LookSensitivity. Existing release/cancel/switch/unpossess cleanup restores both automatically. Only BP_InfiltratorLaserRifle/ScopedRifle opt into2x/.75. User then requested optic body screen dimensions doubled again at that same world FOV: AimSightLocalTransform translation silver(.769527581,-19.413053553,4.958324651), scoped(0,-.7111947,19.0188), existing yaw90/scale1. Eye relief13.0345/12cm respectively. Hip framing/mesh/muzzle/animation unchanged. Silver lens geometry audited from final canonical mesh, not stale historical focus-position defaults. Build/live53checks/cold verification pass; see FEAT219 archive.


FEAT275 Executive remote control: AFPSTheExecutive owns UTheExecutiveDroneRemoteComponent and grants UGA_TheExecutiveControlDrone. Q uses existing BlinkAction; possesses existing ATheExecutiveDrone, pauses its AI, retains real body ASC/grants/health. Remote transition bypasses FPS base possession initialization via ACharacter to prevent healing/regrant. Companion eligibility accepts remote session. Drone existing swept Flight handles manual WASD/QE; actual camera class BPC_TheExecutive_DroneBodyCamera under Executive/Gameplay/Abilities/DroneControl/Camera, FOV112..45, mouse look/wheel/F return. Source BodyCam DisplacePower setter fixed0 (original ArtifactsAmount*5 wiring); Distortion default0 alone does not disable water. Glitch1 owner PP lasts.28s and cleans on return/cancel. Restore body visibility and AI. Drone EndPlay destroys AIController only; remote return tolerates PC Pawn already cleared. External role change cleans abandoned body/drone without stealing new possession. No core/base edits. See FEAT275 archive for evidence/settings.


FEAT275 HUD/view update: UTheExecutiveDroneWidget only updates telemetry (ALT is downward world trace in meters; FOV actual camera; mode text). Layout is editable WBP_TheExecutive_Drone under Gameplay/Abilities/DroneControl/UI, referenced by DroneRemote.RemoteWidgetClass. Remote lifecycle adds/removes HUD and temporarily hides/restores owning-player CombatHUD to avoid duplicate reticles. V binds on possessed drone, toggles CalcCamera chase view; sphere10cm sweep on Camera channel, distance180cm/height45cm; same BodyCam PP/zoom preserved. First-person actual camera offset(-8,0,18) leaves real shell/fin edge at bottom with level wide view, varies with pitch/zoom. Real drone mesh OwnerNoSee=false. Full character CharacterMesh0 BP has Visible=false, so cache/restore IsVisible as well as OwnerNoSee and HiddenInGame; force full body visible only during remote. No source/skeleton/animation changes. Build and actual DroneRemote automation pass. See FEAT275 archive.


FEAT274 input: PrimaryFire Started still sends Input.Weapon.PrimaryFire. Completed calls ReleasePrimaryFire -> Input.Weapon.PrimaryFireReleased; Canceled calls CancelPrimaryFire -> Input.Weapon.PrimaryFireCanceled. Reload, OnDeath, UnPossessed and EndPlay cancel primary fire before lifecycle teardown. These methods transmit events only; GA_ChargedShoot owns charge decisions.


FEAT302 input split: `IA_Interact` stays on E and calls the possessed player's controller `TryInteract` for nearby world actors. `IA_CharacterAbility` is mapped to F; `AFPSCharacterBase::ActivateCharacterAbility` sends `Input.Character.Ability` to GAS. `GA_InfiltratorScan` and `GA_TheExecutiveDeployDrone` listen to that tag; Executive placement release also binds F Completed/Canceled. The legacy `Input.Character.Interact` tag is defined for compatibility but no current ability listens to it. Infiltrator scan discovers only interactables with `UScannableInteractableComponent`; the component unlocks E and owns the transient scan highlight.

FEAT304 player wall clearance, revised 2026-10-22: AFPSCharacterBase::PossessedBy sets a minimum 40 cm capsule radius when owned by APlayerController. The three human Blueprint CDOs remain at radius 34 cm and half-height 88 cm; runtime enlargement leaves about 10 cm between the roughly +30 cm first-person eye offset and the collision boundary. The former 46 cm radius blocked visually open narrow gaps. AI and Spider Tank dimensions are unaffected. The earlier local camera sweep and firearm viewmodel retreat were reverted at the owner's request. A long gun may still overlap a wall at extreme proximity. See FEAT304 archive for build and PIE state.

FEAT304 first-person rendering: HeadCamera enables UE First Person Scale at 0.1 for player view geometry. ArmsViewMesh is a FirstPerson primitive; BeginPlay reapplies these settings after Blueprint component defaults. Project NearClipPlane is 0.5 cm in Config/DefaultEngine.ini: the previous 0.4 scale with 10 cm near plane cut visible holes in equipped guns and did not reliably clear walls in the owner's main level. Camera FOV and ADS values remain authored as before. The render adjustment keeps the visible arms and gun in front of a near wall without changing world collision or the full-size body shadow. See FEAT304 archive for three-role rendered PIE evidence.

2026-10-22 Infiltrator arms correction: BP_Infiltrator's four ArmsViewMesh slots use the existing opaque Infiltrator Outfit/Accessories/Backpack/Body materials. The previous FirstPerson_*_Proximity materials clipped pixels within 16 cm of CameraPositionWS while the 0.1 first-person scale moved most arm pixels inside that radius. The mesh visibility flags and Spider Tank return restoration were already correct. A rendered PIE frame with the non-clipping materials shows the right hand and forearm gripping the current rifle; the old proximity assets remain unused by the arms.


2026-10-22 backpack item initialization: AFPSCharacterBase has an EditDefaultsOnly InitialInventoryItems array of FInventoryItemStack. BeginPlay passes it to EquipmentManager.InitializeItems before initializing the existing weapon actors. BP_TheExecutive sets one BP_AccessCard; BP_Infiltrator and the other human roles use the empty inherited default. This item list is role-local and separate from InitialEquipmentClasses.

FEAT-302 proximity highlight (2026-10-22): AInteractableBase arbitrates white proximity (stencil 2) and scan yellow (stencil 1), preserving each mesh's original CustomDepth/stencil. UScannableInteractableComponent forwards scan start/stop instead of writing depth directly. M_Infiltrator_Highlight renders both colors; yellow has priority. MainGame, SpaceStation and TestMap have the blendable at weight 1. See Documentation/Interaction/InteractableHighlight.md.
