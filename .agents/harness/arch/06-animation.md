# 动画实例

## FEAT-296 Enemy 动画统一（2026-10-15）

正式 Enemy 主实例统一为 `UEnemyAnimInstance`，由 `AEnemyBase` 的 `MainEnemyAnimClass`、`BasePoseAnimationLayerClass` 和可选的 `SpecialPoseAnimationLayerClass` 装配。Phantom、Medusa、VoidGazer 不再需要各自的 C++ AnimInstance 子类。共享实例计算移动速度、方向、转身权重；Humanoid 的巡逻/瞄准/手部 IK 数据与 Medusa 的平滑移动数据由同一实例按 Owner 类型更新。Void 的片段播放时钟属于 `UEnemySequencePlaybackComponent` 派生的 `UVoidAnimationComponent`，末端触手/墙面姿态由 `FVoidGazerPoseProcessor` 处理。

统一的无骨架主图 `/Game/Enemy/_Shared/Animations/Logic/TABP_Enemy_Main` 为 `EnemyBasePose → EnemySpecialPose → DefaultSlot → HitReactionSlot → Output`。Phantom、Medusa、VoidGazer 的 `ABP_*_Main` 是该模板绑定各自骨架的子蓝图；各自的 BasePose 层负责具体姿态，Medusa 另有 SpecialPose 实现。各角色资源仍需匹配自己的骨架，动画重定向只在批准的外部资源项目完成。下文关于旧 `ABP_Phantom_Native`、`ABP_Medusa_Locomotion`、`ABP_VoidGazer` 作为正式主图的记录为历史状态。

地面 BlendSpace 求值复用 `/Game/Enemy/_Shared/Animations/Logic/TABP_EnemyGroundLocomotion` Linked Anim Graph。调用方提供同骨架 BlendSpace 与轴值；Phantom 巡逻/瞄准、Medusa 地面、VoidGazer 地面分支共享这一节点逻辑。Void 的方向样本保存在 `BS_VoidGazer_Ground`，墙面、扑跃、攻击仍用其原序列时钟及程序姿态后处理。该共享图不做资产重定向，也不决定各物种何时进入地面状态。

## FEAT268 Void wall facing (2026-10-08)

Every wall tentacle attack plays the full existing LookBack.Start (Ledge Lean_Out) before extension; active windup is max(configured windup, authored Start length). Wall-relative target direction chooses Left/Right/Back/Up/Down Hold, with250ms captured-pose transitions and a continuous hold clock across attack phases. Full authored lean replaces the old attenuated idle/procedural charge on walls only. Retraction/cancellation starts LookBack.End and pauses crawl until it finishes; new grabs reject during this recovery. Ground locomotion/poses are unchanged.

BP_VoidGazer adds LookUp/LookDown triplets plus WallLeap StartLeft/Right and LandLeft/Right. Ten final AnimSequences were prepared only in TMIIR and migrated without source dependencies. Ordinary wall looks can also play all five Start/Hold/End sets. Side launches leave the wall at4/30s, continue the remaining1/6s of their0.3s push-off in early flight, then use Air. Side grabs use the aligned0.3..0.6s reach portion before contact and recover from0.6s; their initial fast spin is not played against an attached wall. Center transfer and ground traversal retain their prior clips.

Root capsule/support orientation is unchanged during wall attack. This is authored directional facing selected during windup, not unlimited continuous aim IK or corner navigation. Original tentacle skin resolution and existing contact limitations remain.

FEAT169 directory update: MaintenanceWorker animation paths are now `Gameplay/FirstPerson/Animations` and `Gameplay/Body/Animations`; Native variants remain in their corresponding Native subdirectories. The current native gameplay and lobby skeleton/body dependencies live in `MaintenanceWorker/_Shared/Body`. Lobby-specific sequences remain `Lobby/Animations`. Graphs, poses and the three-weapon loadout are unchanged. Older paths below are historical; use archive/FEAT-169-asset-mapping.json for exact current locations.

FEAT-085执行官：TheExecutive/Lobby/Animations/AS_Executive_Lobby_RelaxedIdle 与 AS_Executive_Lobby_ReadyIdle 使用执行官原CAT衍生63骨Skeleton，均30fps/3.2667秒；在TMIIR重定向原Rifle_01 Relaxed/Aim Idle后，外部烘焙左臂握持修正。成品无运行时IK，保留原比例；BP_Executive_Lobby配置两个序列及各自枪挂点，复用0.5秒姿态混合。Character／Back=Relax；Weapon=Ready。Standing未制作，非本批范围。

## 大厅持枪姿态（FEAT-082，2026-09-14）

FEAT-082大厅：已移除旧LobbyCharacterAnimInstance及左臂IK；运行时由LobbyPoseBlendAnimInstance混合每枪成品。每把正式武器配置独立两条Relaxed与一条Rifle成品序列，Blender对照Phantom修正左前臂、手腕及拇指／四指前两节旋转，TMIIR生成最终序列；上臂与肘部位置、身体／右手保持。共九条按枪命名成品，目标不保留动画制作Rig或Retargeter。

- 2026-09-05当前受击仅播放成品动画：EnemyHitReactionComponent仅按Actor局部入射方向选择，BP_Phantom直接配置前后左右4条AS_Humanoid_BlastRifle_{Front,Back,Left,Right}，来自Mixamo并在外部适配现役70骨。原始时长/30fps保留，握枪和腿长差异已调整。
- 共享无骨架后处理位于Humanoid/_Shared/Animations/Logic/ABP_Humanoid_HitReaction。输入Pose缓存与动态SequenceEvaluator混合后输出；ReactionAnimation/Time/Alpha、UseFullBodyReaction由NativeUpdateAnimation提供。旧ControlRig节点、模式选择、RigUnit和Frame/旋转/包络参数已删除，不存在切回Rig的入口。
- ApplyAnimationRootMotion默认开启：非下落时临时暂停CharacterMovement模式，按游戏时间提取Root水平位移，经Mesh方向/缩放转换，用SafeMoveUpdatedComponent移动胶囊；结束或关闭恢复，死亡不恢复。动画force_root_lock避免重复位移。下落不消费Root且仅上半身；关闭参数时沿用静止全身/移动上半身的混合规则，没有部位例外。碰墙可截短步长，不进行落脚重规划。
- 2026-09-06用户澄清：仅爆炸弹延时范围爆炸在实际Health下降且目标存活后请求动画，不再限定AttachedHitActor。首次中弹保留伤害和附着，但不触发击退动画。范围爆炸为每个目标在伤害前按当前朝向和爆心计算局部方向。零伤害、未扣血、墙后、范围外和死亡目标不播放。重复反应不重启，主AnimBP继续运行；声音/伤害/死亡布娃娃和附着清理独立。
- 部位分类、BodyAnimations、ActiveRegion、BoneMapping、HumanoidReactionBones.h和HeavyFront参数均已删除。具体动画引用归Enemy组件，共享ABP不引用Phantom资产。其他骨架须在外部资源项目适配成品。
- hand_r_wepSocket仍挂动画武器骨骼，单位武器缩放，Relax/Aim不切换静态手部Socket。

**何时读取：** 新增动画变量、修改动画状态机所需的驱动参数时。

> 完整 ABP 架构（层结构、Slot、节点流程、扩展策略、旧双骨骼系统）见 `.agents/harness/arch/12-anim-blueprint.md`

**基类（骨骼无关）：**

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Characters/_Shared/Animations/BaseLocomotionAnimInstance.h` | 所有动画实例的 C++ 基类；输出 Speed / Velocity_Z（垂直速度，上升+下落-）/ bIsFalling（离地即 true）/ AimPitch（[-1,1] 归一化）/ Direction |

> **session47 起 `AimPitch` 是玩家上下瞄准的唯一来源**：身体已关物理俯仰（`bUseControllerRotationPitch=false`），看上下不再转 mesh，须由 ABP 用 `AimPitch` 驱动上半身 AimOffset/脊柱 Modify Bone 绕肩俯仰（FEAT-039）。`bIsFalling`/`Velocity_Z` 驱动跳跃状态机过渡。

**玩家动画：**

> FEAT-077 当前双类结构：`ArmsViewMesh=ABP_MaintenanceWorker_FirstPerson` 继承无骨架 `TABP_CharacterBase_FirstPerson`，通过 `ALI_WeaponAnim` 链接当前枪械层；`CharacterMesh0=ABP_MaintenanceWorker_Body` 继承 `TABP_CharacterBase_BodyLocomotion`，生成完整身体 locomotion，并通过 `UFPSCharacterAnimInstance.FirstPersonPoseSource` 在主图末端 Copy Arms 局部 Pose、从 `spine_01` 以上混合。禁止让 Arms 再反向 Copy Body，否则形成循环依赖。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Characters/CharacterBase/Animations/CharacterBaseAnimInstance.h` | FEAT-074 预留的第一人称强类型 C++ 数据源。当前完整原版 AnimBP 尚未安全迁入该父类，因此 `AFPSCharacterBase` 对它继续使用兼容反射写值 |
| `Source/TheManTest/Public/Characters/CharacterBase/Animations/FPSCharacterAnimInstance.h` | 身体 locomotion 模板仍在使用的玩家动画实例。继承共享 Locomotion 变量并提供 `AccelDirection` / `bHasAcceleration` 及当前身体模板仍引用的切枪 Pose 变量；不是弃用手臂类。旧 `UFPSArmsAnimInstance` 源码和 CoreRedirect 均已删除 |
| `Source/TheManTest/Public/Weapons/_Shared/Animations/EquipmentAnimInstance.h` | 装备/武器动画层通用父类。输出 Speed / Direction / Velocity_Z / bIsFalling，供通用装备上半身 Idle/Walk/Run/Jump 动画层使用。FEAT-046 session75 回退为普通 1D BlendSpace 方案后，不再暴露 Start/End 状态机专用临时变量 |
| `Source/TheManTest/Public/Weapons/_Shared/Animations/FirearmAnimInstance.h` | 4 个 AimIK 变量：AimSourceLocalTransform / AimTargetComponentSpace / bHasValidAimTarget / bIsAiming |

**玩家 locomotion（session63 简化）：**

- 当前第一人称无骨架模板是 `/Game/Characters/CharacterBase/Animations/FirstPerson/Logic/TABP_CharacterBase_FirstPerson`，具体宿主是 `/Game/Characters/MaintenanceWorker/Animations/FirstPerson/Logic/ABP_MaintenanceWorker_FirstPerson`；身体无骨架模板位于 `CharacterBase/Animations/Body/Logic`，具体身体子类位于 MaintenanceWorker 对应目录。
- 两个 CharacterBase 模板只拥有动画图结构，不直接引用 MaintenanceWorker 的 Sequence/BlendSpace；具体资产通过角色子 AnimBP 的 Parent Asset Overrides 下沉。第一人称 locomotion 状态机统一命名为 `FirstPersonLocomotionSM`。
- MaintenanceWorker 的 Body/第一人称 Sequence、BlendSpace 与动作资产全部归入具体角色的 `Animations/Body`、`Animations/FirstPerson`；`CharacterBase` 不再保存具体角色表现资产。
- 武器动画接口与模板分别位于 `/Game/Weapons/_Shared/Animations/Interfaces/ALI_WeaponAnim` 和 `/Game/Weapons/_Shared/Animations/Templates/TABP_FirstPersonFirearmBase`；RepairGun 子层位于 `/Game/Weapons/RepairGun/Animations/FirstPerson/Logic/ABP_RepairGun_FirstPerson`。
- FEAT-288 新增 Enemy 无骨架动画模板 `/Game/Enemy/_Shared/Animations/Logic/TABP_Enemy_Locomotion`：`UBaseLocomotionAnimInstance` 驱动 `Direction/Speed`，子 AnimBP 按原始 Skeleton 覆盖移动 BlendSpace；共享 `DefaultSlot` 供技能 Montage，`ALI_EnemySpecialPose` 提供可选特殊姿态层，默认直通。现有正式 Medusa/Void 动画尚未切换；新怪使用步骤见 `Documentation/Enemy/EnemyAnimationTemplate.md`。
- 2026-08-21 root 方向纠正：原始/外部最终 VFXPack 第一人称动画 root 参考方向为 Yaw=0°；TheManTest 曾错误恢复为约-90°。当前7条正式序列的首尾 root Yaw 均已审计为0°。组件构图和 GripPoint 必须以这批正确动画为基准，不得再通过动画 root ±90° 补偿组件方向。
- FEAT-074 MaintenanceWorker 特例：`CharacterMesh0` 与 `ArmsViewMesh` 同时使用整理后的原版 VFXPack 第一人称 AnimBP，C++ 向两者同步写原变量。原 `Walk_Run_1D` 是 Speed 一维轴；Body Sway 目标为 Side=`Clamp(MoveRight+MouseX)` 与 Forward=`Clamp(-MoveForward-10×LookUp)`，Walk/Sprint 插值速度分别为 2/8。原 AnimBP EventGraph 随后还会应用 authored `Lean_Sides_Offset=8` / `Look_Up_Offset=2`；由于该 EventGraph 的示例角色硬 Cast 已被移除，C++ 写 AnimBP 时必须保留这两个倍率。鼠标轴按帧消费，移动输入 Completed/Canceled 时目标清零。A/D 不额外平移或旋转 `ViewmodelRoot`。原版冲刺层级是 `FPS_Camera -> BodyRotator(相机原点) -> SK_ArmMesh(位置偏移)`；当前必须同样让零位置的 `ViewmodelRoot` 在 0.2s 内压至 Pitch -12.5°，并让 `ArmsViewMesh` 持有 SK_ArmMesh 偏移，不能把偏移放到旋转节点。Shadow/Legs 仍 Leader=`CharacterMesh0`。
- 不再使用 `StopAnimIndex` / `bShouldStop` / `bShouldMove` / 脚相位 / 延迟停 / 保持原速滑行。
- ABP 应采用最小状态机：`Idle <-> WalkRun BlendSpace`，常用条件为 `Speed > 3` 与 `Speed <= 3`；跳跃可继续用 `bIsFalling` / `Velocity_Z`。
- `UFPSCharacterAnimInstance` 继承 `UBaseLocomotionAnimInstance`，保留 `AccelDirection` / `bHasAcceleration` 给起步 Lean 或后续玩家专属动画使用。若 ABP 不需要 Lean，可以完全不读这两个变量。
- session89 起玩家暂不使用原地转身：`UFPSCharacterAnimInstance` 只保留 locomotion 与可选加速度变量，不再输出 Turn In Place 变量或读取 `TurnRootYaw` 曲线。玩家身体直接跟随 Pawn yaw；具体转体方案以后单独设计。
- 若蓝图仍报缺变量，说明 ABP 里残留 FEAT-039 停步节点，需要删除 Stop 状态、Stop 过渡和相关变量引用。

**人形怪动画：**

> **FEAT-051 原始骨架策略：** 保留 `UBaseLocomotionAnimInstance -> UHumanoidEnemyAnimInstance -> 无骨架 Template AnimBP` 的驱动架构；每种 Enemy 优先使用其动画原始 Skeleton，并创建对应骨架的子 AnimBP，通过 Asset Override 填入兼容动画。状态机逻辑可复用，但 `hand_r` / `hand_l` / spine 链 / `AimSocket` / IK 骨骼与 Socket 必须按每套原始骨架核对。玩家仍让 `GetMesh()`、`ArmsViewMesh` 和武器层共用玩家 Skeleton，下半身可继续使用效果合格的重定向动画。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Enemy/Humanoid/HumanoidEnemyAnimInstance.h` | 继承基类；AIState / bIsTurning / TurnAngle / TurnAnimIndex；bIsPatrolScanning / PatrolScanAnimIndex；**无 Stopping 状态**：StopDecelerationRate / VirtualSpeed / bIsVirtualDecelerating 驱动虚拟减速；**AimIK**：AimTargetComponentSpace / AimSourceLocalTransform / AimAxisSocketName / AimAxis / bHasValidAimTarget / bIsAiming / AimAlpha / AimAlphaInterpSpeed；**左手 IK**：LeftHandIKTarget / bHasWeapon / WeaponGripLeftSocket |
| `Source/TheManTest/Private/Enemy/Humanoid/HumanoidEnemyAnimInstance.cpp` | NativeUpdateAnimation：巡逻停车（bIsStoppingAtPoint 上升沿）和 Aim 停车（RealSpeed 骤降，bIsAimStopping 标志）共用同一套虚拟减速；bAimSourceInitialized 首帧从武器 `Muzzle` socket 计算 AimSourceLocalTransform，同时从骨骼 `AimAxisSocketName`（默认 `AimSocket`，挂在 `hand_r` 下，+X 朝枪管方向）计算 AimAxis；AimAlpha 用 FInterpTo 平滑 |
| `Source/TheManTest/Public/Enemy/Humanoid/AnimNotify_FootDown.h` | 已弃用空壳（FEAT-029 移除 Stopping 状态后废弃），代码保留 |

**ABP_HumanoidEnemy AnimGraph 节点链：**

```
PatrolSM（状态机）
  → Convert to Component Space
  → Two-Bone IK（hand_l，左手贴前握把）
      IKBone = hand_l
      Joint Target Space = Bone Space，Joint Target Bone = lowerarm_l，Location = (0,0,0)
      Effector = LeftHandIKTarget（Component Space）
      Alpha = bHasWeapon（float）
  → BBBAimIK（脊柱瞄准）
      AimSourceBoneName = AimSocket（hand_r 子插槽，+X 朝枪管）
      AimAxis = (1,0,0)
      AimSourceLocalTransform ← AimSourceLocalTransform 变量
      AimTarget ← AimTargetComponentSpace 变量
      Alpha ← AimAlpha 变量
  → Convert to Local Space
  → Output Pose
```

**ABP_HumanoidEnemy 状态机结构（PatrolSM 内）：**

```
SA_PatrolStates（State Alias，含巡逻类状态）→ Aim：AIState == Aim
SA_AnyState（含所有状态含 Aim）→ Dead：IsDead
Aim → Idle：AIState != Aim，Blend 0.2s

内部状态：
  Idle / Patrol_Walk / Scan / Turn系列  ← 巡逻层（SA_CanTurn 子状态机）
  Aim                                   ← 锁定玩家状态，使用 BS_HumanoidCombat（Direction/Speed）
  Dead                                  ← 死亡
```

**EHumanoidEnemyAIState 枚举（HumanoidEnemyTypes.h）：**

| 值 | 含义 |
|---|---|
| Patrol | 巡逻（C++ 路点循环自驱） |
| Aim | 锁定玩家追击（BT 接管，SetFocus 驱动朝向） |
| SearchRush | 冲向玩家最后已知位置，使用非 Aim 的移动表现 |
| SearchScan | 到达后复用 Relaxed Fgt v1～v4 随机环视；连续两次不重复 |
| Dead | 死亡 |

**Phantom 原始 Rifle_01：** FEAT-084 将外观替换为 `/Game/Enemy/Humanoid/Phantom/Meshes/SK_Phantom_Soldier`；外部重新蒙皮后共用 `OriginalRifle/Meshes/UE4_Mannequin_Skeleton` 与原 PhysicsAsset，原 SK_Mannequin 保留。继续使用 `ABP_Phantom_OriginalRifle`，不含 IK Retargeter。`BS_Phantom_RelaxedPatrol2D` 仅 Relaxed，`BS_Phantom_AimLocomotion` 仅 Aim；Burst/Continuous/Aim Reload 直接使用同 Skeleton 的 TMIIR 最终动画。武器没有 `grip_l` 时左手 IK 自动关闭，避免把手拉到组件原点。

**已删除（FEAT-041）：** 旧 `UTheManAnimInstanceBase`（绑定旧 `ATheManCharacterBase`）已删除。对应旧 ABP（`ABP_MainCharacter` / `ABP_FirstPerson_MainCharacter`）需在编辑器一并删除。


FEAT-082电击枪扳机补修：其三条大厅成品动画额外直接修index_01_r／index_02_r／index_03_r旋转，使食指进入上方独立扳机口。hand_r和其他右手骨骼、枪挂点及左手修复不变；因此此前“右手保持”的历史描述不再包含这三节食指。最新微调工程LobbyElectricTriggerFine：用户确认指尖仍低后，仅进一步抬高中/末节，近节与其他部分保持，无IK。


2026-09-14：运行时新增ULobbyPoseBlendAnimInstance（继承UAnimSingleNodeInstance），从当前显示的局部骨骼姿势快照向继续播放的目标成品动画混合；ALobbyCharacterBase.PoseBlendDuration默认0.5秒，SmoothStep进度同步静态/骨骼枪挂点。反向切换重新捕获当前姿势，重复相同状态不重启。初次显示及换枪立即应用；编辑器静态预览保留OverrideAnimationData/PlayAnimation。没有IK，既有每枪独立成品不变。


## FEAT-105 current Phantom native rig (2026-09-17)

This supersedes FEAT-084 mesh/animation configuration and FEAT-080 stationary target configuration. BP_Phantom uses `/Game/Enemy/Humanoid/Phantom/Meshes/SK_Phantom_Native`, its native Skeleton/PhysicsAsset, and `Animations/Logic/ABP_Phantom_Native`. Mesh scale1, relative Z=-90/Yaw=-90. The 31 finalized sequences live in `Phantom/Animations/Native`; two BlendSpaces in `Phantom/Animations/BlendSpaces`. OriginalRifle still owns the live M4_Rifle_01 mesh and materials; its unreferenced animation prototypes were removed by FEAT292. Final animations were retargeted in TMIIR and grip-corrected in external Blender only; never retarget inside TheManTest.

Weapon attaches with identity transform to hand_r_wepSocket on hand_r_wep. Native AimSocket and measured rifle Muzzle socket initialize existing AimIK. Both arm poses/support contact are baked into final clips; reload retains release motion. Four native BlastRifle sequences drive the existing reaction component with horizontal root displacement. Burst/SuppressiveFire/Reload ability references use native clips.

BP_Phantom disables StationaryHitTest, restores shared BP_HumanoidAIController, and uses Patrol150/Combat300/Turn50/SearchRush600 cm/s. SpaceStation saves Phantom_Patrol and four Phantom_Patrol_01..04 points in Gameplay/PhantomPatrol, around X1800..2400/Y300..900, with Nav_PhantomPatrol. Existing PlayerStart is preserved. Temporary validation cameras/lights must not be saved.

## FEAT-122 typed first-person fire weight (2026-09-17)

`UCharacterBaseAnimInstance::FireAnimationWeight` is the shared native double property consumed by first-person animation graphs. Each animation update starts from that AnimInstance class's saved default; an equipped ADS-capable firearm overrides it with `HipFireAnimationWeight` or `ADSFireAnimationWeight`. Switching to a non-ADS weapon restores the animation class default. Runtime code no longer finds Blueprint fields by name. The ten existing base/worker/weapon templates and concrete AnimBPs retain their original defaults (base/worker 1, weapon layers 0).


## FEAT-138 Infiltrator lobby grips and transition contact

Infiltrator lobby now has six independent final sequences: AS_Infiltrator_Lobby_{LaserRifle,ScopedRifle}_{RelaxedIdle,RelaxedIdle_02,RifleIdle}. External same-skeleton arm/hand/finger rotation editing fits current actual meshes: silver right hand on main grip and left hand on vertical foregrip; scoped right hand on pistol grip and left hand under forward handguard. Source body motion and local bone translations/scales preserved; no first-person changes, retargeting or work rigs in destination. Final external files are under D:/Blender Projects/InfiltratorLobbyWeapons/GripCorrection; initial parent-directory pose files are superseded.

ALobbyCharacterBase.BlendSupportHandBone defaults None. BP_Infiltrator_Lobby opts in with Bip01-L-Hand, using existing WeaponAttachSocket=Bip01-R-Hand as the main hand. Only while Alpha<1, ULobbyPoseBlendAnimInstance preserves interpolated authored support-hand contact relative to the weapon hand using a nonstretch two-bone solve and the current blended elbow pole. It changes only upper-arm/forearm/hand rotations, retains translations/scales and exact authored sequences at blend completion. Source contact comes from the displayed pose snapshot, so reversing mid-transition remains continuous. No per-frame actor transform correction or character gameplay rig changes. Worker/Executive keep None and existing FK blending. This optional transition constraint supersedes the earlier blanket 'no lobby runtime IK' description for Infiltrator transitions only.

FEAT-138 ready-only refinement: final two RifleIdle sources supersede parent GripCorrection with GripCorrection/ReadyRefinement. Scoped ready mount compensates raised right hand; relaxed clips/mounts remain unchanged. Silver ready thumb curls updated.


## FEAT-146 native Infiltrator gameplay (2026-09-23)
Infiltrator now uses its original lobby native target skeleton/fullbody plus owner FirstPerson/Meshes/SKM_Infiltrator_FirstPersonArms and Body/Meshes/SKM_Infiltrator_LowerBody. Arms/body run ABP_Infiltrator_FirstPerson and ABP_Infiltrator_Body under their owner Animations folders. This supersedes worker-asset reuse for Infiltrator only; Worker/Executive unchanged. Body retains locomotion/hidden shadow and copies native upper-body pose; legs follow body. Native graphs replace spine/hand names and omit independent left-hand sway to preserve two-hand contact. The two Infiltrator gun BPs use native first-person layers/montages under Characters/Infiltrator/Animations/FirstPerson/{LaserRifle,ScopedRifle}, attach to Bip01-R-Hand with independently measured offsets. Default loadout remains silver only. Shared templates and gun optical/gameplay properties unchanged. Finalized locomotion20clips and weapon16clips were adapted externally in TMIIR; no runtime/destination retargeting. Four native arm materials add16cm/8cm camera-distance dither clipping; original lobby/body materials unchanged. Editor authoring helper is Characters/Infiltrator/Editor/InfiltratorAnimationAssetLibrary; gameplay does not call it. See FEAT146 archive and external InfiltratorNativeWeapons evidence.


## First-person hand pose requirement (2026-09-23)
User reference: C:/Users/ROG/Pictures/Screenshots/???? 2026-09-23 171553.png (FAMAS first-person screenshot).
For future weapon animation authoring/review, match the reference's diagonal support-arm silhouette: left forearm rises diagonally from the lower-left/bottom toward the weapon support grip; wrist and palm follow the grip naturally. Avoid the currently rejected horizontal forearm across the screen. Review both left and right hand target positions AND orientations against each weapon's actual support grip/pistol grip, including finger contact and natural wrist/elbow alignment. Do not apply one arbitrary wrist rotation or identical target coordinates to every gun. Weapon camera placement awaits the user's separate reference; no weapon/animation assets changed for this feedback. Retarget work stays in approved external projects; migrate finalized clips only. Keep verification proportional: targeted pose/camera/contact review, expand only if a problem appears.

FEAT-148 supersedes FEAT-146 first-person left-elbow poses:16 owner clips now use downward elbow poles for a diagonal support forearm, preserving wrist targets and fingers. Latest editable animation sources D:/Blender Projects/InfiltratorPoseFraming/Final; do not reimport older InfiltratorNativeWeapons/Final without reapplying pose correction. BP_Infiltrator ArmsViewMesh relative location is(-5.694741,-31.049028,-130.791799)cm, original rotation/scale retained. Both gun camera captures reviewed.

FEAT-148 scoped wrist follow-up: use D:/Blender Projects/InfiltratorPoseFraming/ScopedWristFix/Final for8 scoped clips. Forearm-axis roll now absorbs twist previously concentrated at wrist; contact and finger poses unchanged. Parent Final scoped poses were rejected for cuff distortion; silver parent Final accepted and unchanged.


## FEAT-149 native role integration (2026-09-23)
Worker/Executive native body locomotion and four weapon animation profiles live under Characters/<role>/Animations/{Body,FirstPerson}/Native. Retarget only in TMIIR. Scoped/Repair/ExecutiveSlow frozen GunFraming targets applied to both gun and hand poses; diagonal support forearms. Infiltrator scoped latest source WorkerExecutiveNative/ScopedFinal; accepted silver source unchanged. Executive secondary poses additionally use executive-secondary-deltas.json. See archive/FEAT-149-native-worker-executive.md and D:/Blender Projects/WorkerExecutiveNative for evidence.

FEAT-149 user correction: Executive has exactly ONE weapon, the native BP_TheExecutive_ExecutiveSlowRifle. Extra Executive-only InfiltratorLaserRifle/ScopedRifle weapon and animation folders were deleted, not available loadout options. Worker keeps its three guns. Obsolete /Game/Weapons/RepairGun/Meshes/SM_RepairGun deleted and its static-component references cleared in both RepairGun BPs; actual SKM_RepairGun remains. This supersedes earlier six-profile/secondary-Executive framing notes.

FEAT150 in progress: user rejected high/unnatural hands and requested per-weapon acceptance. ExplosionGun8clips latest source D:/Blender Projects/FirstPersonHandReview/ExplosionGun/Final, lower frozen gun framing plus revised support elbow/forearm roll. Await user acceptance; other profiles not yet revised.

FEAT150 follow-up: ExplosionGun accepted. ElectricGun8clips latest editable source D:/Blender Projects/FirstPersonHandReview/ElectricGun/Final; frozen ElectricGun camera target and revised support elbow/forearm roll now applied. Actual hold/move/fire montage reviewed; ElectricGun aesthetic acceptance pending. Other gun profiles unchanged.

FEAT150 ElectricGun grip follow-up supersedes parent Final: use FirstPersonHandReview/ElectricGun/RelaxedGrip/Final. User wants support hand resting against gun with relaxed fingers, not tightly clenched. Current8clips soften left finger middle/distal curl only; gun/palm/wrist/forearm preserved. Await acceptance; apply this visual requirement to future weapon pose reviews.

FEAT150 RepairGun follow-up: current8clips use D:/Blender Projects/FirstPersonHandReview/RepairGun/RelaxedGrip/Final; frozen skeletal-mesh camera target preserved, support elbow/roll and relaxed finger curl revised. Parent Final is intermediate. Actual hold/move/fire montage reviewed; visual acceptance pending. ElectricGun retained per user.

FEAT150 RepairGun current source supersedes RelaxedGrip: FirstPersonHandReview/RepairGun/FinalContact/Final. Latest user gun target(-5,-92,9)deg at(17,17,-24) parent Gun, apply saved mesh_camera_relative. Thumb01/02 now fits native thumb surface against actual SKM_RepairGun; retain relaxed four fingers. Prior wrist-only contact check missed hovering thumb; future reviews include actual thumb/finger surface contact. Await visual acceptance.

FEAT150 final3gun sources now FirstPersonHandReview/Remaining/{ExecutiveSlowRifle,InfiltratorLaserRifle,InfiltratorScopedRifle}/Final.24owner sequences preserve gun framing, revise support forearm/roll and relaxed fingers with actual render-mesh thumb contact (exclude exported collision geometry). Infiltrator Fire/FireBase use rigid original recoil delta on held pose to retain two-hand contact through additive blend; fix_recoil.py supersedes bake.py outputs for4clips. No graph/runtime changes. Focused gameplay review passed; joint visual acceptance pending. RepairGun final revision accepted.

FEAT150 2026-09-24 supersedes prior Remaining sources: use Remaining/GripRevision/<weapon>/Final for Slow/Laser/Scoped. Relaxed support-hand clearance and Executive both-hand placement revised; Executive weapon equip_attachment_transform compensates changed right hand while preserving saved gun framing. bake.py must be followed by recoil.py for all three Fire/FireBase pairs. GUI BP compile and focused hold/move/fire review passed. User retained this version (可以先这样); stop further pose edits. Actual full hand geometry review is required: nearest thumb vertex distance alone missed palm penetration in previous batch.


## FEAT235 enemy reactions (supersedes earlier explosion-only configuration)

Native reaction classes moved from Enemy/Humanoid/Animation to Enemy/_Shared/Presentation with reflected identities retained. EnemyBase owns the component. UEnemyHitReactionConfig provides Projectile/Explosion directional sequences, optional default sequence, root movement selection, blend/play rate/minimum interval, and Disabled/Animation/ControlRig mode. No config means no reaction. AnimationPostProcessClass selects the common animation graph; ControlRigPostProcessClass must derive from UEnemyHitReactionAnimInstance and implement its own rig graph using LocalSourceDirection, HitWorldLocation, HitBone, RigStrength and RigAlpha. The ControlRig option supplies the integration contract, not an automatically generated rig for every skeleton. No active reaction restarts; presentation does not cancel attack GAS abilities. Death stops live reactions and disables postprocess before its body presentation.

Medusa locomotion uses native Idle/Walk/Run BlendSpace -> DefaultSlot -> output, with native spell/dash/melee/death montages. Source skeleton preview attachments are cleared after copying; no IK rigs or retargeting assets are imported.

FEAT235 path update: the skeleton-free shared postprocess now serves Phantom and Medusa and is moved with AssetTools to /Game/Enemy/_Shared/Animations/Logic/ABP_Enemy_HitReaction. Old Humanoid path is obsolete. Per-enemy HitReactionConfig selects the postprocess.

FEAT235 finalized presentation details: native Sevarog hit sequences are local additive. Shared ABP_Enemy_HitReaction accepts non-additive, local-additive and mesh-space-additive sequences, applies additive sequences to the incoming base pose, then uses the existing reaction envelope/full-body or moving upper-body blend. ReactionIsAdditive/ReactionIsMeshSpace are derived from the sequence, not per-enemy hardcoded. Missing config or selected direction/default sequence skips presentation. ControlRig mode is an extension contract: assign a postprocess class derived from EnemyHitReactionAnimInstance and consume reaction direction/point/bone/strength/envelope in its rig graph; no finished ControlRig solver is supplied by this feature. The Medusa config uses Animation.

Death montages hold their final frame (disable auto blend-out per montage instance), then apply configured KeepPose/Ragdoll/Destroy. Medusa uses its native Death_front montage then KeepPose; native source physics does not provide a fully articulated character ragdoll. Phantom/FlyingBug2 preserve ragdoll; CoreMorph preserves its own cleanup/pose. Additive lifecycle callbacks remain available with all body modes.

## FEAT237 Medusa directional movement and smooth facing (2026-09-30)
Supersedes FEAT235 speed-only movement graph. UMedusaAnimInstance derives from BaseLocomotionAnimInstance, owns interpolated MovementSpeed/MovementDirection, normalized actor-yaw RotationRate and TurnWeight. Editable ABP_Medusa uses BS_Medusa_Directional (Direction -180..180, Speed0..600; native Idle + four Walk clips at300 + four Run clips at600), blends BS_Medusa_Turning by TurnWeight, then DefaultSlot. All existing attack/death montages and shared hit postprocess remain. Direction wraps across +/-180; backward samples exist at both ends. Rotation-rate pose samples reproduce original Sevarog_TurnInPlace ordering (-360LBC,-180LC,0CC,180RC,360RBC); weight fades with movement and while attacking. Original source AnimBP speed/acceleration/yaw interpolation and directional blend assets were inspected, not imported wholesale (player input/combo/IK graphs are not enemy gameplay).

Medusa CharacterMovement uses controller desired rotation,180deg/s yaw, acceleration700, braking850, friction4 and acceleration-driven paths. UseControllerRotationYaw is false. AI path completion allows deceleration. ChooseAction/BeginAction require facing target within15deg; BeginAction no longer snaps actor rotation and holds facing while attacking, restored by CancelAction. Attacks, GAS pools/Task, damage and nightmare are otherwise retained. Eleven native sequences copied using AssetTools onto the already-copied identical native skeleton (bone names/parents validated); no retargeting. Fullbody native walk/run directional clips were selected because original Jog_Fwd has missing additive base in source load. Source project was read-only.


FEAT238: AS_Medusa_Intro is original Sevarog LevelStart (5s, nonadditive, native identical skeleton, no retargeting). MedusaIntroSubsystem plays a transient DefaultSlot montage and advances only this mesh in real time during world pause. No AnimBP graph changes. First activation only; no attack animation/GAS changes.


FEAT239 supersedes automatic FEAT238 intro: BP_Medusa.bPlayIntroduction=false while user hand-edits /Game/Enemy/Nightmare/Medusa/Sequences/LS_Medusa_Intro. This6s30fps LevelSequence contains spawnable native preview mesh/cine camera,editable Transform/LevelStart/CameraCuts/Fade tracks. Preview in TestMap; no persistent map actors. Fullframe28mm lens,DOF disabled,motionblur0,no aspect lock/YFOV framing. Draft is not bound to gameplay activation. Native800cm range and all GAS remain. Automatic replacement handoff is a future user step after shot editing.


FEAT240 supersedes FEAT239 draft-only behavior: BP_Medusa.bPlayIntroduction=true and IntroSequence references LS_Medusa_Intro. Existing800cm/LOS decision gate plays it once per combat world. Native MedusaIntroSubsystem uses LevelSequencePlayer and DefaultLevelSequenceInstanceData.TransformOrigin: first authored skeletal Transform inverse * actual Enemy mesh world transform. Handles UE5.7 custom spawn bindings via SkeletalAnimation-track ownership. Sequence keys/lens/animation/duration unchanged; real gameplay actor stays stationary/hidden while authored visual actor plays. World pause freezes combat/countdown; real-time subsystem seeks sequence and refreshes skeletal pose. Paused Fade renderer workaround samples authored active Fade section value/color/audio at the same time.0.3s entry and0.35s return bridge around authored shot. Stop restores state/destroys spawned actors,view,input,visibility,fade and owned pause; death/EndPlay/unpossess/travel cancel. Legacy IntroAnimation property is compatibility-only; edit IntroSequence for playback. No shared activation redesign.


FEAT241: native/BP Medusa ActivationRadius1200cm supersedes800cm. Intro visibility track hides preview frames0..12; main LevelStart12..162 preserved; separate fixed-time last-frame hold162..180 prevents idle reset during fade. Camera/mesh transform keys preserved. Death_front is only.966667s and final tilted pose is unsuitable for corpse hold; current death behavior unchanged pending separate presentation decision.


## FEAT243 Void Gazer wall crawl
VoidGazer uses independent UVoidGazerAnimInstance (AnimSingleNode proxy,180ms captured-pose interpolation) for finalized Idle/CrawlUp/Down/Left/Right sequences. Native tentacle local motion baked into target sequences in TMIIR; root fixed in place. No runtime retargeting. Tentacle wall contact remains deferred; current original motions can intersect walls.


## FEAT244 Void Gazer ledge look
Nine finalized AS_VoidGazer_Look{Left,Right,Back}{Start,Hold,End} sequences live beside its crawl clips. TMIIR-only LedgeAnimationSet retarget includes source neck/head into target neck; native tentacles baked, pelvis reoriented onto wall and offset against crawl reference. Start/End nonloop, Hold loop, existing180ms captured-pose blend. No target rigs or source assets. This is discrete authored directional looking, not continuous skeletal aim tracking.


## FEAT245 Void Gazer in-place correction
Four finalized crawl clips now remove accumulated pelvis translation across each cycle, not merely root motion. TMIIR source helper subtracts first-to-last linear drift then centers planar pelvis mean on Idle; retains cyclic bob/sway, depth baseline, all rotations/scales and other bones/tentacles. Target receives only corrected four sequences at unchanged paths. Actor code still exclusively drives wall travel. Ledge authored lean/recovery remains; no runtime pose cancellation or root-motion switch workaround.

## FEAT247 Void Gazer wall leap
Three finalized AS_VoidGazer_WallLeap{Start,Air,Land} clips derive from TMIIR LedgeAnimationSet Jump_Up/Falling/Grab. Existing source-only retargeter and preparation helper preserve native tentacles, reorient pelvis onto the crawler basis and lock root. Start uses the first9 frame intervals/0.3s of Jump_Up, prepared in TMIIR. It replaces Jump_Out because that clip authored a139deg pelvis turn during the attached windup. The component derives takeoff timing from this asset length. Air samples one full clip across actual bounded flight duration; Land plays1s. VoidGazer's existing captured-pose animator uses120ms transitions for leap, returns to180ms crawl/look transitions afterwards. No looping travel accumulation, runtime retargeting or root-driven actor movement. Source rigs/raw clips remain exclusively in TMIIR; target animation references are configured on BP_VoidGazer.WallLeap.


## FEAT246 movement responsibility update
See `15-enemy-movement.md` for current selection/request/execution ownership. Ground queries are shared by Phantom/Medusa; Void mechanics move into WallCrawlMovementComponent with tree scheduling. Existing animation assets, retargeting and CoreMorph flight presentation remain unchanged. Implementation verification is tracked in FEAT246 archive. This supersedes earlier actor-Tick random roaming and Controller-Tick Phantom combat requests.

FEAT246 user correction: FlyingBug2 is retired. Its owner assets/native class/dedicated tests are removed; historical FlyingBug sections above are archival, not current setup instructions. Do not restore or continue FEAT076.


FEAT246 movement ownership: ABP_Phantom_Native uses Root Motion from Montages Only. Ordinary navigation must drive CharacterMovement velocity; prior Everything mode consumed small animation root drift and blocked normal patrol. Sequences, retargeting, blend graph and montage displacement are unchanged. See arch/15-enemy-movement.md and FEAT246 evidence.


FEAT248 Void ground presentation: seven native finalized sequences GroundIdle/GroundAir/GroundLand/Melee1..4 prepared in TMIIR by assigning the equivalent target skeleton, preserving original tracks/rates and removing source ControlRig user data. Ground clips use upright native basis. Single BGA_VoidGazer_Melee owns an Animations array and randomly chooses one of four0.5s attacks at activation; component anim playback keeps selected clip until completion,80ms blend, common hit0.22s. This is animation variation within ONE ability, not four skills. Ground landing clip0.2s and idle1.65s; existing wall clips unchanged.


## FEAT251 restrained grab breathing (2026-10-07)
While Tentacle presentation is active, Void plays its existing ground/wall idle at.75 rate with ground GrabBodyMotionWeight=.12 and wall GrabWallBodyMotionWeight=.65 (wall raised after user found it too static). VoidGazerAnimInstance proxy blends evaluated local pose toward the same clip time0 reference, keeps root/pelvis reference translations and retains small rotational/breathing motion. This replaces both full-amplitude1.65s idle bob and the intermediate fully frozen pose rejected by user. Normal weight1/rate resumes outside grab. No new skeletal animations, retargeting or player struggle clip.


### FEAT259 Void ground locomotion / charge
VoidCombatComponent holds GroundWalk/Back/Left/Right and GroundIdleRelaxed finalized native animations prepared in TMIIR. VoidGazer chooses directional walk from actual local velocity, playback speed tied to ground speed; existing wall crawl selector remains. Idle_2 original has18cm hip oscillation, so ordinary grounded idle uses configurable .18 pose-motion weight, fixed root/pelvis translations and .65playrate through the existing animation proxy (skill grab retains its prior independent motion weights). No target-project retargeting.
Tentacle windup adds a procedural upper-spine charge: smooth rearward bend during Windup, hold in Locked, release over first.15s of Extend; wall version uses40% weight. Lower-body/root transforms are untouched by the charge layer. SetChargeWeight is cleared automatically whenever no charge phase is active. Attack Niagara components are created hidden/inactive; first activation/visibility occurs at Extend, with reach growing from0. Existing warning, lock, hit, QTE and GAS timing unchanged.


### FEAT260 current Void behavior (2026-10-07)

FEAT260: ground idle/walk and four melee clips now use finalized AS_VoidGazer_GroundCrawlIdle / GroundCrawl / GroundCrawlAttack1..4, retargeted from TMIIR Animations_Creature Sit_Idle_1 / Sit_Walk / Sit_Attack variants entirely inside TMIIR. Native tentacle tracks are composited there; in-place root, native SKEL_VoidGazer, no source rigs/meshes/ControlRig user data migrated. All directional walk slots reference GroundCrawl because movement now faces travel. Ground speed140; existing stable-idle weighting retained. Single melee ability randomly selects four low attacks, hit time0.5s. GroundTakeoff is a finalized native0.4667s jump-start prepared in TMIIR; existing air/landing/wall clips remain. This supersedes FEAT248/259 ground idle/walk/melee clip assignments.


### FEAT261 wall transfer polish (2026-10-07)
FEAT261 wall-only polish reuses existing finalized native clips: airborne local pose blends toward wall Idle frame0 with72% peak tuck, fades before reach; distributed spine_01/spine_02/neck_01 world-turn lead capped16deg. Proxy resets both weights outside wall transfer. Air samples1x looping time rather than duration stretch. Existing Land enters min(.20s,28% flight) before contact; first1/30s is approach, same sequence/time continues after contact. No asset/retarget changes; ground variants retain old sampling. No tentacle contact IK.


FEAT262/264 ground eye presentation: GroundEyeLift defaults35degrees (supersedes50), distributed25/75percent on spine_01/02 because eye-bearing geometry is upper torso. Before captured-pose blending, two-bone shoulder/elbow solve follows original per-frame tentacles1_l/r wrist positions using original elbow pole. No limb stretch; unreachable goals clamp. Twist/tentacle local tracks untouched. Root/pelvis/legs and supplied stride preserved. This preserves authored contact intent, not terrain IK. Wall/leap/melee still use authored poses.


FEAT265 supersedes FEAT260 ground Creature and FEAT262/264 correction: restored native GroundIdleRelaxed/directional GroundWalk assets, full idle weight/rate1, native Melee1..4 with.22s hit. Removed GroundEyeLift and arm IK code/property. Original full mesh/tentacles retained. Existing ground AI approach/near stop/no random strafe remains. Creature assets retained unused; wall animations unchanged.

2026-10-15 Void idle tune supersedes the full idle weight/rate1 setting above: production BP_VoidGazer still references native standing `AS_VoidGazer_GroundIdleRelaxed`, but sets GroundIdleMotionWeight=.10 and GroundIdlePlayRate=.65. The native proxy blends each idle frame with the sequence's first pose, retaining subtle motion without prominent authored swings. No idle asset, skeleton or retargeting change. TMIIR's GroundCrawlIdle is quieter but has a low crawling posture and was not migrated.

2026-10-15 user review supersedes the preceding tune: the .10/.65 idle looked nearly static. Production BP_VoidGazer is restored to GroundIdleMotionWeight=1.0 and GroundIdlePlayRate=1.0 with the same `AS_VoidGazer_GroundIdleRelaxed`. Cold editor reload verified saved values. A separate neutral standing loop with occasional fidgets is only a future design idea.

2026-10-15 user approved the externally previewed calmer standing loop. Production BP_VoidGazer GroundIdle now references `/Game/Enemy/Nightmare/VoidGazer/Animations/Locomotion/AS_VoidGazer_GroundIdleCalm`; GroundIdleMotionWeight and GroundIdlePlayRate remain 1.0. The finished 2.4s sequence was authored in TMIIR on Void's existing skeleton and migrated alone. The original GroundIdleRelaxed remains available. No destination-project retargeting or proxy changes.


FEAT266 ground descent only: FlightRotation holds initial6%, SmoothSteps to upright at62% flight; native GroundAir samples authored1x looping time; Land transition45ms (native200ms recovery retained). Wall transfers/ground takeoff keep prior timing. Original assets/mesh/tentacles unchanged.


### FEAT267 original-mesh tentacle deformation

FEAT268 wall-only curvature follow-up: FVoidNativeTentaclePose includes bWallPose and stable Variation. Wall windup retains the authored tentacle pose at zero reach. Extension unfolds a whole local cubic (opposite lateral controls for broad S bends) with root-to-tip blending and bounded strand-specific motion, rather than stretching the near-root prefix of the full target curve. Ground keeps the prior evaluation. Full native chains/materials and body pose are retained; original limited bone resolution still bounds smoothness. See FEAT268 archive for verification.
VoidTentacleComponent bUseNativeTentacles defaults true (inherited BP); native mode retains existing6..8 count/timing/victim anchors and uses UpdateNativeAnimation snapshots consumed in VoidGazerAnimInstance after authored pose/charge/transition. Three torso pairs13..21,22/23/230/24..27,28..33 plus optional wrist1..6 follow length-sampled cubic/waves/hook with root fixed and bone translations extending original skin. Windup/retract blend,150ms cancel fade, explicit dead-owner layer reset. Proxy stores undeformed transition source to avoid double deformation. Temporary mesh bounds16 restored on end/cancel. Native mode creates no Niagara strands; bUseNativeTentacles=false restores old generated path. Animation assets/full mesh/materials/gameplay unchanged. Limited native chain resolution still exposes corners at large extension. See FEAT267 for actual previews and validation.

FEAT268 ground-to-wall follow-up: uses existing GroundTakeoff/GroundAir and WallLeapLand clips. Ground start keeps upright departure then smooth wall alignment(.18..82 flight); upper-body lead also applies to ground return. Final up-to240ms of flight plays the wall receiving clip, clock0->.18s, then continues from that time after contact. Wall-transfer existing directional reach windows remain; angular admission rejects rotations>120deg. No new animation/retargeting or mesh changes.

FEAT268 latest correction: ground return keeps upright-basis GroundAir during its short flight with <=15deg root forward lean and no upper-body turn lead. Ground grab playback now begins at supported contact (existing .18s clip offset), with root adopting wall basis over180ms recovery. This supersedes the earlier pre-contact ground Land clip and .18..82 airborne root alignment; wall-to-wall directional reach unchanged.

FEAT268 descent attack: VoidWallLeap.GetDescentAttack snapshots time/weight for native AnimProxy.SetDescentAttack; existing Combat.MeleeAnimation sampled across Air toward MeleeHitTime at impact, continues through Land and fades by200ms. Local-pose overlay only spine_01 descendants, waist feather .35/.7 and full arms/neck/tentacles. Root/pelvis/legs retain base falling/landing clip. Applied before transition blend/native strands; cleared outside descent and explicitly on death. No melee ability activation/extra hit/damage. Ground pounce takeoff continues100ms airborne before40ms blend into GroundAir, avoiding two disconnected launch clips.

FEAT268 descent tentacle strike: when no active tentacle ability owns presentation, UpdateNativeAnimation consumes WallLeap.GetDescentTentacles on native-mode ground descent. Six torso strands reuse full curved native renderer; behind/up endpoints swing to a forward/down landing fan, max500cm from root. Air swing42..88%, blend3..18%; Land holds30ms/retracts by200ms. Melee upper-body overlay remains underneath; no wrist strand override/new effects/assets. Death produces empty snapshot; existing150ms cancel fade applies.


FEAT268 final descent windup correction: AVoidGazer keeps the same looping IdleAnimation and continuous clock during GroundLanding.Start (200ms crawl-to-idle blend). GetAnimation also identifies that idle; departure waits WarningDuration>=300ms, no unrelated StartAnimation length. Air uses captured curled pose -> GroundAir/upper-body attack with240ms SmoothStep; contact still45ms. No new clips or retargeting.


FEAT276 electrical damage: ReactToHit adds optional bInPlace=false; true suppresses animation root displacement while preserving per-enemy config, skeleton checks, blend and anti-restart interval. Electrical periodic hits reuse already matched AS_Phantom_BlastRifle_{Front,Back,Left,Right} selected by source direction. TMIIR Rifle_01/Animation contains760 animation asset files, no dedicated hit/react/hurt/flinch/damage/stun names; death/fidget sequences not substituted. No new retargeting or source rig assets in destination.


FEAT-276 final revision (2026-10-11): ElectricGun creates field instantly500cm ahead; no projectile flight or attraction. Periodic GE2damage/.5s,2.5s duration and field Cue retained. Electrical reaction strength1 (supersedes .45); configured Front/Back/Left/Right selected from orb relative direction, in-place flag suppresses animation root displacement without changing movement mode. Four actual selected sequences verified by native automation; actual GAS PIE damage and cleanup passed. See arch/09-equipment-system.md and feature archive.


FEAT276 arrival/pulse revision supersedes constant full-strength electrical reaction: field owns transient arrival light and one-shot .26s core scale envelope, activates main at.07s/branches.14-.24s, independently gates branch visibility (width250 retained). Actual damage Cue schedules reactions per target; first strength1/max.55s, subsequent random.22-.38/max.24s with small independent delay. ReactToElectricalHit reuses direction selection, guard and no-root movement via ReactToHit, then applies1.25x/1.8x rate and shortened fade envelope; SampleAnimation respects ActiveDuration. Ordinary reactions retain full-clip duration. No new animations or global config changes. Death/range/LOS/expiry remove target scheduling with existing field state.

## FEAT277 configurable shared hit reactions (2026-10-12)

This supersedes the historical September5 four-direction/removed-mode descriptions. FEAT235 provider contract remains authoritative. UEnemyHitReactionComponent::RequestReaction(FEnemyHitReactionRequest) is the single presentation entry. Existing ReactToHit / ReactToExplosion / ReactToElectricalHit are adapters; no damage is applied by this component. Projectile defaults Light, explosion Heavy, electric first Medium and subsequent twitch Light. Strength is blend intensity, independent of severity. Direct custom callers may set source/severity explicitly.

Per-enemy UEnemyHitReactionConfig.Rules selects by Source, Severity, eight-way Direction, and Bones. LocalSourceDirection is actor-local pointing TOWARD the incoming source (+X Front,+Y Right), horizontal45-degree sectors; unknown/vertical direction only matches Any. Bones are exact mesh names; IncludeChildBones allows descendants. Empty list means any bone; invalid/absent hit bones do not match a specific bone rule. Bone matching chooses the clip, not a playback mask; existing stationary/full-body and moving/falling upper-body blending remains.

Precedence: larger Priority, then nearest matching bone ancestor (exact beats group, group beats any), then count of specific source/severity/direction filters, finally earlier array index. Empty/incompatible-skeleton animation pools are skipped. A matching Disabled rule suppresses presentation and does not fall through. Animation pools filter valid same-skeleton clips, deduplicate and avoid repeating the last clip when alternatives exist. Per-rule mode override selects Animation/ControlRig/Disabled; false inherits Config.Mode. Rules own PlayRate/BlendIn/BlendOut/MaxDuration(0=whole clip)/ApplyRootMotion. In-place requests always prevent root movement. Root consumption stops at capped duration and restores movement.

If no usable rule matches, existing Projectile/Explosion four-way/default data remains fallback (electric/melee use Projectile). Legacy unconfigured electrical reactions retain .55s first/.24s twitch timing; matched rules own their timing. No-restart, minimum interval and death cleanup preserved. Mode selects/switches only the configured postprocess provider, never the locomotion AnimBP. Missing ControlRig provider fails safely; no finished rig graph supplied.

Current Phantom and test soldier both reference /Game/Enemy/Humanoid/Phantom/Data/DA_Phantom_HitReaction. Add named entries under Rules to configure; a head rule uses Bones=[head], optional Source/Severity/Direction filters, and appropriate finalized clips in Animations. Existing saved assets keep Rules empty and their established animations. JKMotion audit does not prove complete eight-way/body-part coverage; no JKMotion retarget/migration in this feature. Retargeting remains external-only.

Verification: TheManTest.Enemy.HitReactionRules, TheManTest.Player.Weapons.ExplosionDamageReactions, TheManTest.Player.Weapons.AttractionProjectile. Tests use transient TestMap PIE fixtures; no map saves.

## FEAT278 current fixed enemy reactions (2026-10-12)
Supersedes FEAT277's empty production-rule/legacy-animation state. UEnemyHitReactionConfig.bUseDirectionalSets enables Light/Heavy FEnemyDirectionalReaction slots: Front/Back/Left/Right, one fixed clip each. Config.Rules remains empty in production and available for future overrides. Medium maps to Light; incoming actor-local direction uses four quadrants for fixed sets. Legacy Projectile/Explosion remains compatible when flag is false. No armed/unarmed runtime classification.
Phantom + BP_Phantom_WeaponTest share DA_Phantom_HitReaction; Medusa and VoidGazer own their configs. All three use eight finalized AS_<Owner>_Hit_<Light|Heavy>_<Direction> sequences in owner Animations/Reactions. Phantom preserves baked rifle grip; Void preserves native tentacles; both use externally retargeted JKMotion body motion. Medusa light/heavy derives from its own native weapon-compatible four reactions. Light around.6s/heavy around1s; Medusa fractional source frame rates retained. Root movement disabled in these sets. No random alternatives assigned. Existing electrical envelope shortening remains.
AEnemyBase::CanPlayHitReaction() defaults alive; AVoidGazer overrides to exclude IsWallAttached. Shared request and pose/Rig sampling honor eligibility, so wall attachment immediately hides ongoing ground reaction and rejects new ones. No wall reaction animations authored. CoreMorph config remains Disabled/emptyRules; deleted flying bug excluded by user instruction.
All retargeting/baking occurred in TMIIR. Only24 finalized sequences migrated via AssetTools, existing destination mesh/skeleton used; no authoring rigs, source skeletons or work folders in TheManTest. Actual PIE EightReactionAssets, HitReactionRules, AttractionProjectile and ExplosionDamageReactions pass. Evidence D:/UnrealWork/EnemyEightReactions; see FEAT278 archive.

FEAT279: Enemy animation directories now follow Logic/Locomotion/Actions/Reactions, Phantom legacy skeleton variants isolated under Legacy. Native sequences moved by purpose; BlendSpaces join Locomotion. Shared logic and cinematic ownership unchanged. Full232-path mapping in archive/FEAT-279-enemy-animation-paths.json. No clip timing/root flags/config selections changed by this organization. FEAT278 visual acceptance was rejected by user: external rework must retain appropriate root motion and be previewed/approved before production application.

FEAT289: `UEnemyHitReactionConfig.Mode` now also offers Montage, with matching `*Montages` directional sets and optional rule `Montages` pools. A valid hit Montage must use the enemy skeleton and `HitReactionSlot`; hit requests skip Montage playback while any other Montage (skill/death) is active. Existing Phantom/Medusa Data Assets keep Post Process Animation mode and their Sequence assets. Production authoring steps are in `Documentation/Enemy/EnemyHitReactionModes.md`; no retargeting belongs in this project.


FEAT278 approved humanoid replacement (2026-10-12): Phantom/test soldier now use Wraith light V2 and RifleAnimsetPro heavy V3 at existing eight canonical Reactions paths. Heavy Front=C_1, Back=L_2 (fallback, not authentic backward hit), Left=L_1, Right=R_1; original timings/playrate1 preserved. Heavy config ApplyRootMotion=true, sequence EnableRootMotion=true and ForceRootLock=true/AnimFirstFrame: capsule consumes authored root once, evaluator pose root stays locked. Light config ApplyRootMotion=false. Cold PIE verifies eight slots, heavy125–145cm travel with zero duplicated pose displacement and movement restoration. Void/Medusa unchanged; see FEAT278 archive.

FEAT278 light transition correction (2026-10-12): four Phantom/test-soldier Wraith light clips now local-space additive relative to own frame0 (ABPT_LocalAnimFrame), composed over incoming locomotion by existing postprocess; eliminates common AimIdle stance jump. Full source timing/keys retained; no light root travel. DA_Phantom_HitReaction BlendIn=.10/BlendOut=.25 applies to light/heavy. Heavy absolute clips/root extraction/force lock remain unchanged. Real4side firing of both Infiltrator rifles and actual additive pose output verified; direction dispatch was already correct.

FEAT278 current light-contact integration (2026-10-12), superseding additive paragraph above: user rejected additive; all four canonical Phantom light reactions are nonadditive FootContactV1 from external TMIIR,120fps and original durations. Nonstretch leg contact solve maintains native AimIdle feet. Left includes <=3.2354cm downward pelvis compensation to avoid unreachable legs; upper local reaction/grip retained. Current BlendIn=.02/BlendOut=.25, initial test soldier Patrol with normal engagement enabled. Actual stationary four-direction real shots/recovery and separate hit-induced Aim lifecycle verified; full-alpha foot error<.009cm, recovery<=.263cm. Heavy root settings/mapping unchanged. Relaxed stance-entry foot placement and heavy entry/recovery remain unresolved separate issues. Only finished four clips migrated; no runtime IK or new destination source code.


## Current unified humanoid turns (2026-10-12)
UHumanoidEnemyAnimInstance now exposes TurnAnimation/TurnAnimationTime/bUseAuthoredTurn, bEnterAim(Aim&&!turning) and bExitTurnToRelaxed(nonAim&&!turning). Existing bIsTurning/TurnAnimIndex and seven relaxed child asset overrides remain. Actual ABP_HumanoidEnemy template has one Turn state with relaxed BlendByInt plus an optional authored alert SequenceEvaluator selected by bool. Both Patrol and Aim share this state. Alert clips are eight owner-configured native-skeleton assets, not template hard references. RootMotionMode remains MontagesOnly; native owner consumes alert root yaw only, never turn translation. Shared hit postprocess remains ABP_Enemy_HitReaction and contains no turn branch. This supersedes the historical seven separate turn-state description.

FEAT291: `UBaseLocomotionAnimInstance` offers `SetLocomotionSmoothing` so Medusa keeps its prior speed/direction response while using the shared locomotion template. Its linked turning layer still uses `UMedusaAnimInstance` variables. `UVoidGazerAnimInstance` now derives from `UAnimInstance`; `CurrentSequence` and `CurrentTime` drive a real AnimBP graph. Void's `FAnimInstanceProxy` evaluates that graph and then applies prior authored-pose transition, body offsets and procedural tentacles. `AVoidGazer` retains clip choice/playback-rate logic and supplies its configured AnimBP class at BeginPlay. No animation retargeting is performed here.
2026-10-21 FEAT-304 Spider Robot procedural locomotion: `USpiderRobotProceduralAnimInstance` (`Characters/Infiltrator/SpiderTank`) derives from `UAnimSingleNodeInstance`; its custom `FAnimSingleNodeInstanceProxy` evaluates the 28-bone robot reference pose then solves two-bone IK on each Coxa/Femur/Tibia/Foot chain. `AInfiltratorSpiderTank` stores six world-space stance targets, traces support near each leg, and alternates L1/R2/L3 with R1/L2/R3 swing tripods. A 0.14-second swing uses a 4 cm lift arc; stance feet retain world positions as the body moves and rotates. The full world/shadow skeletal mesh evaluates the pose; the leg-only first-person mesh follows it by leader pose. No destination retargeter, IK Rig or baked source animation is part of this path. FEAT-303's SPTANK walk/jump AnimBP remains an older asset but is no longer selected by the approved robot at runtime.

2026-10-21 FEAT-304 Fab mesh replacement: runtime full and leg-only meshes use the 45-bone armored source rig in Characters/Infiltrator/Gameplay/SpiderTank/Meshes/Armored. Its six Front/Mid/Rear_01..04 chains were renamed externally to the existing L1..R3_Coxa/Femur/Tibia/Foot contract. The 85 rigid source parts received single-bone weights before export; piston bones remain in the skeleton. USpiderRobotProceduralAnimInstance still computes every step with runtime IK. Vendor Idle/Walk/Jump actions were not imported and no destination retargeting was performed. Both imported meshes share SKEL_Infiltrator_SpiderRobot_Armored; the leg mesh follows the full mesh by leader pose. The earlier 28-bone Procedural meshes remain as inactive rollback assets.

2026-10-21 FEAT-304 Fab orientation correction: the external Blender conversion rotates the vendor armature, full mesh and leg-only mesh -90 degrees around world X, centers XY and grounds the lowest toe at Z=0 before FBX export. The two reimported skeletal meshes share the corrected skeleton. The runtime IK contact target is 12.5 cm along the surface normal from the trace hit because each `*_Foot` bone pivot sits about 12 cm above its physical toe at mesh scale 0.035. This keeps the authored metal toe on the support plane rather than forcing the ankle pivot into it. The procedural tripod gait, wall support and zero-gravity traversal remain the animation source; no vendor actions or retarget assets were imported.
