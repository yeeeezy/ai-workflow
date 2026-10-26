> Current authority (verified 2026-09-19): FEAT-077 supersedes historical single-host / dual-link sections below. ArmsViewMesh uses ABP_MaintenanceWorker_FirstPerson and owns the weapon Linked Anim Layer. CharacterMesh0 uses ABP_MaintenanceWorker_Body, retains body locomotion and copies the completed arms upper-body pose; no second weapon layer link. CharacterMesh0 casts the complete hidden shadow; old ShadowBodyMesh/ShadowUpperBodyMesh remain empty. Three playable characters reuse worker host assets by prior agreement. See arch/03-character-base.md and EquipmentBase.cpp::GetAnimLayerMeshes.

## FEAT-298 traversal montage candidate (in progress, 2026-10-17)

Paired MaintenanceWorker body and first-person Hurdle sequences/montages are authored in external TMIIR and target the existing respective skeletons. Both montages use the host graphs' `FullBodySlot` and share a 1.6667s root track (185cm across, 100cm peak rise). The runtime ignores montage root motion for actor movement and samples the body montage track to move the capsule through a validated frame; it aligns first-person arms to the body component while hands contact the sill. No final montage has been migrated or assigned to production AnimBPs yet. Visual review gate and PIE validation remain; see FEAT-298 archive.

The first-person visual review rejected the retargeted Hurdle arm clip and world alignment. The adopted correction is a separate camera-framed FP arm montage while the Hurdle body montage supplies root/pelvis/legs and the capsule path. Keep the ordinary Body AnimBP `Copy Pose From Mesh(FirstPersonPoseSource) -> Layered Blend Per Bone(spine_01)` active during vault: the FP performance intentionally overrides the body upper body. This correction is pending implementation and external preview; the preceding paragraph describes the currently compiled candidate, not the accepted final setup.

# 动画蓝图架构（ABP 层 / Slot / 节点图）

## FEAT-296 当前 Enemy 主图（2026-10-17）

唯一的正式 Enemy 主图是无骨架 `TABP_Enemy_Main`，节点顺序固定为 `ALI_EnemyBasePose.EnemyBasePose → ALI_EnemySpecialPose.EnemySpecialPose → DefaultSlot → HitReactionSlot → Output`。没有绑定 SpecialPose 时接口直通输入姿态。`AEnemyBase::BeginPlay` 创建 `MainEnemyAnimClass` 实例，再通过 `LinkAnimClassLayers` 连接 BasePose 与可选 SpecialPose。三个类设置在敌人 BP 继承的 Enemy 动画配置中；不在物种 C++ 类中重复装配。

| Enemy | Main 子图 | BasePose 图层 | SpecialPose 图层 |
|---|---|---|---|
| Phantom | `ABP_Phantom_Main` | `ABP_Phantom_BasePose`（继承 `TABP_Humanoid_BasePose`） | 无 |
| Medusa | `ABP_Medusa_Main` | `ABP_Medusa_BasePose` | `ABP_Medusa_SpecialPose` |
| VoidGazer | `ABP_VoidGazer_Main` | `ABP_VoidGazer_BasePose` | 无 |

三者的 Main 均继承 `TABP_Enemy_Main`，C++ 父类均为 `UEnemyAnimInstance`。BasePose 内保留骨架和物种需要的移动姿态逻辑；共享主图统一接收技能 Montage 与受击 Montage。VoidGazer 的程序姿态在共享实例的代理完成最终姿态处理，状态从 `UVoidAnimationComponent` 取得。下文描述旧主图或旧物种 C++ AnimInstance 的段落仅作迁移历史参考。

地面移动的公共内部图是无骨架 `TABP_EnemyGroundLocomotion`（`UEnemyGroundLocomotionAnimInstance`）。它只做一件事：用调用方传入的 `GroundBlendSpace`、`GroundX`、`GroundY`、`GroundPlayRate` 评估 BlendSpace。BasePose 用 **Linked Anim Graph** 调用它：Phantom 的 `TABP_Humanoid_BasePose` 在巡逻和瞄准状态各调用一次，分别传入速度与方向/速度；Medusa 的 `TABP_EnemyGround_BasePose` 传入方向/速度；VoidGazer 的 `ABP_VoidGazer_BasePose` 在地面时调用，传入方向/速度和播放速率。各物种 BasePose 子图设置自己的 BlendSpace 资产，公共图不持有任何物种骨架或动画。转身状态机、IK、Medusa SpecialPose、Void 墙面/扑跃/攻击序列仍在各自图层或组件。Void 地面 BlendSpace 是 `BS_VoidGazer_Ground`，其五段样本均是已迁入本工程的成品动画；`UVoidAnimationComponent` 的地面/特殊标志驱动 BasePose 的分支。

## 当前受击链（2026-09-05）

- `/Game/Enemy/Humanoid/_Shared/Animations/Logic/ABP_Humanoid_HitReaction`是无骨架后处理模板，父类EnemyHitReactionAnimInstance，由Humanoid基类默认软类和具体Enemy配置接入。旧ControlRig目录、CR_Humanoid_HitReaction、Rig节点/模式开关/原生RigUnit已删除。
- 图：LinkedInputPose→ReactionInputPose缓存；SequenceEvaluator读取ReactionAnimation/ReactionTime→AuthoredReactionPose缓存；TwoWayBlend全身或LayeredBoneBlend(spine_01)上半身按ReactionAlpha混合，再由UseFullBodyReaction选出直接接Output。无UseAnimationReaction字段。InstallEnemyReactionAnimationBranch现在重建纯动画图，旧Rig安装API已删除。
- 六部位分类已删除，BP_Phantom仅配置4条Mixamo持枪成品方向动画，不同命中部位共用方向判断。2026-09-06用户澄清为仅延时范围爆炸实际扣血的存活目标触发，首次中弹不触发；爆炸按各目标相对爆心的方位，主AnimBP不换。动画评估不触发Notify；根位移由EnemyHitReactionComponent显式消费并碰撞扫掠，Sequence force_root_lock，避免重复位移。
- 非下落根位移反应临时暂停CharacterMovement，因速度归零使用全身；结束/关闭恢复移动模式。下落保留上半身与原空中运动。关闭根位移时仍保留原站立全身/移动上半身、腿受击全身的规则。
- 具体资产/骨架、武器挂点均归Phantom；共享图无Phantom动画依赖。正式安装/冷读/PIE脚本使用Scripts/VFX/*mixamo*，旧Rig和旧动画安装验证脚本已删除。

## 总体原则

- **当前 ABP 系统：纯第一人称手臂（FPS Arms，单一骨骼）**，Locomotion 共享基础动画，武器差异化通过 Linked Anim Layer 注入。
- 武器动画遵循 **“共享基础 + 按需覆盖”**：普通武器只实现持枪姿态和瞄准层，特殊武器（需要专属跑跳动画）在其 Layer 内自带 StateMachine。
- **旧双骨骼系统（`ATheManCharacterBase` / `UTheManAnimInstanceBase`）C++ 已删（FEAT-041）**；`ABP_MainCharacter` 等旧蓝图待编辑器删除。文末旧节点图仅作历史参考。

---

## 核心文件（当前活跃）

| 资产 / 类 | 路径 | 作用 |
|---|---|---|
| `TABP_CharacterBase_BodyLocomotion` | `Content/Characters/CharacterBase/Animations/Body/Logic/` | 无骨架玩家身体 locomotion 模板；MaintenanceWorker 最终身体子类为其角色目录下的 `ABP_MaintenanceWorker_Body` |
| `TABP_CharacterBase_FirstPerson` | `Content/Characters/CharacterBase/Animations/FirstPerson/Logic/` | 无骨架第一人称宿主模板；拥有基础 locomotion/Jump、武器层路由及最终 Lean/Look 图 |

> CharacterBase 的 Body/FirstPerson Template AnimBP 不得直接绑定具体角色动画资产。模板保留 `Sequence Player` 与专用 `Blend Space Player` 占位节点，具体 Sequence/BlendSpace 由角色子 AnimBP 的 Parent Asset Overrides 提供。第一人称模板状态机名为 `FirstPersonLocomotionSM`；资产为空时，跳跃过渡使用状态机原生自动剩余时间规则，避免在模板中保留指向具体 Sequence Player 的 Getter。
| `ABP_MaintenanceWorker_FirstPerson` | `Content/Characters/MaintenanceWorker/Animations/FirstPerson/Logic/` | 绑定 MaintenanceWorker Skeleton 的模板子类；无本地 AnimGraph，通过继承模板作为 ArmsViewMesh 常驻宿主 |
| `ALI_WeaponAnim` | `Content/Weapons/_Shared/Animations/Interfaces/` | 武器动画层接口，定义两个层：`WeaponAimOffset`、`WeaponUpperBody` |
| `TABP_FirstPersonFirearmBase` | `Content/Weapons/_Shared/Animations/Templates/` | 第一人称枪械 Template AnimBP，实现 `ALI_WeaponAnim`；统一持枪 Idle/WalkRun 状态逻辑，具体武器子类替换动画资产 |

> `TABP_FirstPersonFirearmBase.WeaponUpperBody` 的 Idle 使用空 `Sequence Player`，WalkRun 使用空 `Blend Space Player` 并由 `Speed` 驱动 X；具体枪械子类通过 Parent Asset Overrides 分别提供静止 Sequence 和移动 BlendSpace。不要用 Run Sequence 代替 WalkRun BlendSpace，也不要保留断线的旧播放器或输入节点。
| `UCharacterBaseAnimInstance` | `Source/.../Characters/CharacterBase/Animations/CharacterBaseAnimInstance.h` | 第一人称宿主/枪械模板共享的 C++ 数据基类；统一映射移动、腾空与通用 Lean/Look 变量 |
| `UFPSCharacterAnimInstance` | `Source/.../Characters/CharacterBase/Animations/FPSCharacterAnimInstance.h` | 身体 locomotion 模板仍在使用的 C++ 父类；旧 `UFPSArmsAnimInstance` 已无源码和 CoreRedirect |
| `UFirearmAnimInstance` | `Source/.../Equipment/Firearms/FirearmAnimInstance.h` | 当前枪械动画层模板 `TABP_Firearm_UpperBodyBase` 的 C++ 父类，持有四个 AimIK 变量 |

> 敌人 ABP（`ABP_HumanoidEnemy`）见下方“敌人动画架构”节及 `11-enemy-ai.md`。

---

## ABP_BodyLocomotion（玩家全身主 ABP，普通 locomotion）

> `ABP_BodyLocomotion` 是项目自己的玩家全身主 ABP，不应因为放弃 Motion Matching 而直接删除。若它曾在实验中加入 `PoseSearchHistoryCollector` / Motion Matching / Stop 状态，需要在编辑器里清掉这些节点，保留为普通 locomotion 模板。父类推荐继续用 `UFPSCharacterAnimInstance`，该类现在是很薄的玩家专属子类，只比 `UBaseLocomotionAnimInstance` 多 `AccelDirection` / `bHasAcceleration`。

**当前身体路径：** `Content/Characters/CharacterBase/Animations/Body/Logic/TABP_CharacterBase_BodyLocomotion`

**推荐 LocomotionSM：**

```
Idle ──(Speed>3)──> Run/Walk(BS Direction×Speed) ──(Speed<=3)──> Idle
State Alias "To Falling"（= Idle / Run/Walk）
   ├─(bIsFalling && Velocity_Z>0)─> Jump_Start ─(JumpStart 剩余时间<0.1)─> Jump_Loop
   └─(bIsFalling && Velocity_Z<=0)────────────────────────────────────> Jump_Loop
State Alias "To Land"（= Jump_Loop）─(自动规则: 落地)─> Jump_End
Jump_End ─(Speed<3 && Land 剩余<0.3)─> Idle ；─(Speed>3)─> Run/Walk
```

- **Jump_Start / Jump_Loop / Jump_End** 三状态各用 Sequence Player 播 JumpStart / Fall_Loop / Land（子 ABP Asset Override 填各角色动画；模板里不填）。
- **不再使用 Stop 状态**：删除 `StopAnimIndex`、`bShouldStop`、`bShouldMove`、`bLeftFootUp`、`StopDirectionIndex` 等变量节点和过渡。
- ⚠️ **状态机直接播的 clip 必须 No Additive**（被当差值叠加 → 塌进地里，见 BUG-039-002，最初 Land 动画踩过）。
- **根运动**：普通 Idle/WalkRun 方案建议 `Root Motion from Montages Only`，地面移动交给 CharacterMovement。原地 locomotion clip 的 Enable Root Motion 应关闭。
- AnimGraph 输出链：LocomotionSM → DefaultSlot → 主 ABP 唯一的 Layered Blend per Bone.BasePose；`WeaponUpperBody` Linked Layer 只产出纯武器 Pose并接 BlendPose（当前纯武器实现不读取 `UpperBodyInPose`）→ WeaponAimOffset → UpperBodySlot → FullBodySlot → `Two Way Blend(A=Pose Snapshot "WeaponTransitionPose", B=FullBodySlot, Alpha=WeaponTransitionAlpha)` → Output。武器层图 `WeaponAimOffset` / `WeaponUpperBody` 的 Graph Blend 保持默认关闭；末端不使用 Inertialization。分层与切枪 Pose 事务由主 ABP 统一拥有，武器层不得重复混合全身 locomotion。注意 AnimGraph 的 Pose 输出不能一对多，不能把 `DefaultSlot.Pose` 同时直接接给 BasePose 与 Linked Layer 输入，否则后一次连接会顶掉前一条。

---

## 旧 ABP_FPS_Arm_MainCharacter AnimGraph 流程（仅迁移历史）

> ⚠️ 旧手臂主 ABP，FEAT-039 全身主 ABP（上方 `ABP_BodyLocomotion`）验证通过后将取代它。下面节点图在迁移期仍作参考。

```
Locomotion StateMachine（Idle / Run/Walk / Jump_Start / Jump_Loop / Jump_End）
         │
         ▼
SaveCachedPose "Cache_Locomotion" → UseCachedPose
         │
         ▼
Slot "DefaultSlot"          ← 全身蒙太奇（装备切换等通用动作）
         │
         ▼
WeaponAimOffset (LinkedAnimLayer, ALI_WeaponAnim)   ← 输入 WeaponAimInPose ← DefaultSlot.Pose
         │
         ▼
Slot "UpperBodySlot"        ← 上半身武器动作蒙太奇（开火、换弹）
         │
         ▼
WeaponUpperBody (LinkedAnimLayer, ALI_WeaponAnim)   ← 输入 UpperBodyInPose ← UpperBodySlot.Pose
         │
         ▼
Slot "FullBodySlot"         ← 全身覆盖蒙太奇（预留，死亡/特殊动作）
         │
         ▼
Output Pose (Root)
```

当前主 ABP 不含武器专属 IK 节点；AimIK 应封装在当前武器层模板 `TABP_Firearm_UpperBodyBase` 或其骨架兼容子层内。

---

## 武器动画扩展策略

### 装备动画父类选择

```text
UEquipmentAnimInstance
  └─ UFirearmAnimInstance
```

- 所有装备通用上半身动画层优先继承 `UEquipmentAnimInstance`，可直接读取 `Speed` / `Direction` / `Velocity_Z` / `bIsFalling`。
- FEAT-046 原计划把步枪上半身改为普通 1D BlendSpace，但 session80 MCP 复核发现实际仍为 `Idle <-> WalkRun`，且现有 `BS_Rifle_UpperBody_IdleWalkRun` 是 2D、0 samples；该切片已转为 `needs_improvement`，后续由 FEAT-051 按用户保留的动画资产重新落地。不要依赖已删除的 Start/End 状态机专用变量。
- 枪械动画层继承 `UFirearmAnimInstance`，在通用移动变量之外额外获得 AimIK 变量：`AimSourceLocalTransform` / `AimTargetComponentSpace` / `bHasValidAimTarget` / `bIsAiming`。
- 步枪 Template ABP 若需要 BBBAimIK，应以 `UFirearmAnimInstance` 为父类；非枪械装备可用 `UEquipmentAnimInstance`。

### 普通武器（共享 Locomotion）
- 新建武器 AnimBP，实现 `ALI_WeaponAnim`。
- `WeaponAimOffset` 层：持枪姿态 + 瞄准 IK，直接用 `WeaponAimInPose` 输入。
- `WeaponUpperBody` 层：上半身持握叠加。
- 跑跳动画自动复用基础 ABP 的 StateMachine。

### 特殊武器（需专属跑跳动画）
- `WeaponAimOffset` 层内不用 `WeaponAimInPose` 输入，自带完整 StateMachine（武器专属 Idle/Walk/Run/Jump）。

### Slot 使用约定

| Slot 名 | 蒙太奇类型 | 示例 |
|---|---|---|
| `DefaultSlot` | 全身，在瞄准前播放 | 装备切换、死亡 |
| `UpperBodySlot` | 上半身，瞄准 IK 后叠加 | 开火、换弹 |
| `FullBodySlot` | 全身覆盖，最高优先级 | 特殊动作（预留） |

> Slot 名称需在 ArmsMesh 使用的骨骼资产的 **Anim Slot Manager** 中注册。

---

## ALI_WeaponAnim 接口层

| 层名 | 输入引脚 | 作用 |
|---|---|---|
| `WeaponAimOffset` | `WeaponAimInPose`（Pose） | 武器持枪姿态 + 瞄准 IK |
| `WeaponUpperBody` | `UpperBodyInPose`（Pose） | 上半身武器持握叠加 |

## 玩家身体/手臂共用主 ABP + 双 mesh 武器层（session70）

> **FEAT-077 已替代本节旧的“共用主 ABP”运行方案。** 当前仅保留两个 AnimBP 类：Arms 独立 FP AnimBP；Body 完整 locomotion AnimBP 在最终输出前执行 `Copy Pose From Mesh(FirstPersonPoseSource)`，并以 `Layered Blend Per Bone(spine_01)` 合成上半身。Copy Pose 使用局部骨骼变换，不复制 root/pelvis，也不复制 ArmsViewMesh 的相机空间组件 Transform。Arms 必须先于 Body Tick，装备层只链接 Arms。

`GetMesh()` 和 `ArmsViewMesh` 可以使用同一个主 ABP（如 `ABP_BodyLocomotion`），因为两者骨骼一致。`EquipmentAnimLayerClass` 装备时同时链接到这两个 mesh 的 AnimInstance：

```
GetMesh.AnimClass      = ABP_BodyLocomotion
ArmsViewMesh.AnimClass = ABP_BodyLocomotion

ABP_BodyLocomotion:
  Locomotion
  → WeaponAimOffset（Linked Anim Layer，装备时替换）
  → WeaponUpperBody（Linked Anim Layer，装备时替换）
  → Output Pose
```

- `ArmsViewMesh` 与 `GetMesh()` 各自有独立 AnimInstance。BBBAimIK 使用同一相机射线目标，但会分别转换到各自 Component Space，因此第一人称手臂和身体/影子都能瞄准相机指向位置。
- `ShadowBodyMesh` / `LegsMesh` 不设置自己的 AnimClass，继续 `SetLeaderPoseComponent(GetMesh())`。
- 不使用 `Copy Pose From Mesh`。武器切换仍靠武器 BP 的 `EquipmentAnimLayerClass`。
- 主 `TABP_BodyLocomotion` 是武器上半身分层的唯一所有者：它把基础 locomotion 与 `WeaponUpperBody` 产出的纯武器 Pose 从 `spine_01` 以 Depth 4 渐进混合，并开启 `Mesh Space Rotation Blend`。武器模板不再放置第二个 `Layered Blend per Bone`；这样所有武器统一避免 pelvis/root 横移旋转带偏枪口，并保持身体、影子和 FP 手臂共用同一 locomotion 时序。影子腰部缺口若来自身体模型的几何分段，应修改模型，不能用动画混合作为修复手段。

---

## UFPSCharacterAnimInstance 变量

继承 `UBaseLocomotionAnimInstance`：

| 变量 | 类型 | 说明 |
|---|---|---|
| `Speed` | `float` | 地面移动速度（Size2D） |
| `Velocity_Z` | `float` | 垂直速度 |
| `bIsFalling` | `bool` | 是否在空中 |
| `AimPitch` | `float` | 瞄准俯仰角，归一化到 [-1, 1] |
| `Direction` | `float` | 移动方向角 [-180, 180] |

**玩家专属追加（session63 清理后）**：

| 变量 | 类型 | 说明 |
|---|---|---|
| `AccelDirection` | `float` | 加速度方向角 [-180,180]，可选用于起步 Lean |
| `bHasAcceleration` | `bool` | 当前是否有移动输入加速度 |

> 走跑档由 `AFPSCharacterBase::StartSprint/StopSprint` 改 `CharacterMovement->MaxWalkSpeed`，ABP 通常只读 `Speed` 进入同一个 Walk/Run BlendSpace。

**MaintenanceWorker 步速标定（FEAT-070）：** `BS_RunWalk_MaintenanceWorker` 的 Speed 轴为 0~300，Idle=0、Walk=100、Jog=300；`BP_MaintenanceWorker` 同步覆盖 WalkSpeed=100 / SprintSpeed=300。全部方向样本保持 RateScale=1.0，使 Walk/Jog 在对应速度点完整播放原动画，不被 Idle/Jog 权重稀释；不能把这组值直接用于其他动画集。

**原地转身：** session89 起暂停。玩家 ABP 不应包含 Turn 状态、`TurnRootYaw` 曲线读取或 root-bone 转向抵消节点；Pawn 与 `BodyRoot` 直接跟随 Controller/Actor yaw。后续转体方案另立功能重新设计。

未来重新设计时可参考本机 `D:\Unreal Projects\GameAnimationSample`，但不要直接移植整套 Motion Matching。该样例通过 TurnInPlace/Pivot/Spin Pose Search Database、未来轨迹、左右支撑脚动画版本、`Offset Root Bone`、`Orientation Warping` 和 `Foot Placement` 共同减少滑脚；适合提炼为“脚相选择 + 左右脚转身动画 + 明确的 Actor/Root 同步 + Foot IK”方案。

---

## TABP_Firearm_UpperBodyBase（武器层模板）AimIK 目标流程

> **第一人称持枪空中分支（session253）：** `TABP_FirstPersonFirearmBase.WeaponUpperBody` 在最终输出前由 `bIsFalling` 选择 Pose：地面使用武器层自己的 Idle/WalkRun 状态机；空中使用接口传入的 `UpperBodyInPose`，从而保留宿主 `JumpStart/JumpLoop/JumpEnd` 三段已持枪骨骼动画。该选择属于共享武器基类，具体枪械无需复制通用跳跃转换；将来若某把武器拥有专属空中动画，可在其武器层覆盖 Air Pose，而不是修改角色移动状态判断。
> session254 已删除 `ConfigureFirstPersonFirearmLinkedLayer` 中旧的“宿主 AnimGraph 外部空中旁路”生成分支；已存在的武器层必须由专用配置函数原位维护，不能再次向宿主添加重复的 `bIsFalling` Blend。
> **session255-256 修正：** 不能直接删除宿主资产中历史已有的顶层空中 Blend；该 Blend 同时承担 `WeaponUpperBody` 输入/输出路由，删除会使整个第一人称上半身失去动作。资产已恢复到检查点。随后确认第一人称模板有 3 个 Jump Transition 仍读取旧 `Is_InAir` 图节点，已保留原连线并统一重绑到原生 `bIsFalling`；`Velocity_Z` 分支保持不变。
> **session257 最终职责收敛：** `TABP_CharacterBase_FirstPerson` 已移除本地 `FirstPersonLocomotionSM` 与顶层空中 Blend，只保留 `WeaponUpperBody` Linked Layer 路由及通用 Lean/Look。完整 Idle/WalkRun/JumpStart/JumpLoop/JumpEnd 状态机迁入 `TABP_FirstPersonFirearmBase.WeaponUpperBody`；具体动画由 `ABP_RepairGun_FirstPerson` Parent Asset Overrides 提供。换武器只替换 Linked Layer，即可整体替换地面与空中第一人称动画。

> **FEAT-080 三武器副本边界：** `ABP_ElectricGun_FirstPerson` 与 `ABP_ExplosionGun_FirstPerson` 从 `ABP_RepairGun_FirstPerson` 复制，当前动画内容保持一致，但归属各自武器目录。开火序列/蒙太奇和装备蒙太奇同样按武器独立：`AS/AM_<WeaponName>_FP_Fire`、`AM_<WeaponName>_FP_Equip`。复制蒙太奇后必须用 `UTheManAnimationAssetLibrary::ReplaceMontageAnimation` 将内部 Segment 改指所属武器的开火序列，禁止只复制外壳却继续引用 RepairGun 序列；此流程不属于动画重定向。
> **session258 状态图清理：** 删除素材遗留 `Reset Animation` 及其 6 条放射状转换；Entry 直接进入 `Idle`。状态统一命名为 `Idle / Move / Jump Start / Fall Loop / Land`，并用 `Grounded` State Alias 合并 Idle 与 Move 到 Jump Start 的共同转换。状态图由 21 个节点降为 14 个节点，运行转换保持 7 条。

> **FEAT-074 session148 临时边界：** `TABP_BodyLocomotion` 的第三人称身体输出已旁路 `WeaponUpperBody/AimOffset` 支路，改由 `DefaultSlot.Pose` 直接进入 `UpperBodySlot.Source`。原因是现有 RepairGun Linked Layer 在稳定状态返回参考姿势并覆盖 spine_01 以上，造成 CharacterMesh0/影子 T-Pose。第一人称 ArmsViewMesh 已独立使用 `ABP_VFXPack_FirstPerson`。在武器层能够保证有效 Pose 输出前，不得重新把该支路接回身体最终输出。

```
WeaponAimInPose（输入 Pose）
         │  LocalToComponentSpace
         ▼
BBBAimIK 节点
  ├─ BoneChain：spine_01(0.2) → spine_02(0.4) → spine_03(0.6)
  ├─ AimSourceBoneName：hand_r
  ├─ AimAxis：(1, 0, 0)
  ├─ AimSourceLocalTransform ← UFirearmAnimInstance.AimSourceLocalTransform
  ├─ AimTarget            ← UFirearmAnimInstance.AimTargetComponentSpace
  ├─ bHasValidAimTarget   ← UFirearmAnimInstance.bHasValidAimTarget
  └─ Alpha                ← UFirearmAnimInstance.bIsAiming（bool→float）
         │  ComponentToLocalSpace
         ▼
输出 Pose
```

> 玩家武器层用 `AimSourceBoneName = hand_r`；敌人因 UE4/UE5 骨骼轴向差异改用 `hand_r` 子插槽 `AimSocket`（见敌人节 / `11-enemy-ai.md`）。

## UFirearmAnimInstance 变量

| 变量 | 类型 | 更新时机 | 说明 |
|---|---|---|---|
| `AimSourceLocalTransform` | `FTransform` | 装备时写入一次 | 枪口相对 hand_r 骨骼的局部偏移 |
| `AimTargetComponentSpace` | `FVector` | 每帧 NativeUpdateAnimation | 目标在 Component Space 的坐标 |
| `bHasValidAimTarget` | `bool` | 每帧 | 射线是否命中有效目标 |
| `bIsAiming` | `bool` | 每帧 | 是否处于瞄准状态，控制 IK Alpha |

---

## 武器动画扩展指南

- **新增不需要 AimIK 的武器**：新建该武器 `WeaponAimOffset` 层实现，直接 pass-through 输入 Pose，不放 BBBAimIK 节点。
- **新增需要 AimIK 的武器**：让该武器动画层 ABP 继承或复用与玩家 Skeleton 兼容的 `TABP_Firearm_UpperBodyBase` 路线。
- **新增需要专属跑跳的武器**：在 `WeaponAimOffset` 层内自建 StateMachine，忽略 `WeaponAimInPose` 输入。
- **修改骨骼链权重**：在实际使用的武器层模板/子 AnimBP 的 BBBAimIK 节点 BoneChain 属性里调各骨骼 Weight。

---

## 敌人动画架构（人形怪）

### 通用非人形 Enemy 模板（FEAT-288）

新建的 `/Game/Enemy/_Shared/Animations/Logic/TABP_Enemy_Locomotion` 是无骨架 Template AnimBP，父类为 `UBaseLocomotionAnimInstance`。图链路为 `BlendSpace(Direction, Speed) → EnemySpecialPose(InPose) → DefaultSlot → Output`。`ALI_EnemySpecialPose` 位于 `/Game/Enemy/_Shared/Animations/Interfaces/`，模板中的实现默认直通；需要专属姿态的 Enemy 才创建同骨架的层 ABP，并通过 `LinkAnimClassLayers` 接入。具体骨架、移动 BlendSpace 和最终子 ABP 放在 Enemy 自己的 `Animations/` 下，子 ABP 用 Parent Asset Override 指定 BlendSpace。

FEAT292 清理后，Medusa 仅保留正式的 `ABP_Medusa_Locomotion` 和 `ABP_Medusa_SpecialPose`；旧 `ABP_Medusa` 与教学示例均已删除。Void 使用 `ABP_VoidGazer` 播放选中的片段，程序化触手仍在原生 Pose Proxy 中。普通 Enemy 接入步骤见 `Documentation/Enemy/EnemyAnimationTemplate.md`。

### 总体原则

- **Template ABP 不绑骨骼**：`ABP_BaseLocomotion` 作为无骨骼 Template，所有角色/敌人 ABP 继承它，状态机逻辑零重复。
- **人形怪 ABP 用 Asset Override 填动画**：子 ABP 在 Asset Override Editor 覆盖各 Sequence Player 资产，骨骼在子 ABP 指定。
- **C++ AnimInstance 驱动变量**：蓝图状态机只读变量，逻辑判断在 C++。
- **无 Stopping 状态**：Walk→Idle 通过虚拟减速（`VirtualSpeed`）在 blend space 内平滑完成。
- **Turn 用原地动画 + 代码驱动旋转**。

### 核心文件（人形怪）

| 资产 / 类 | 路径 | 作用 |
|---|---|---|
| `TABP_CharacterBase_BodyLocomotion` | `Content/Characters/CharacterBase/Animations/Body/Logic/` | 无骨架玩家身体 Template ABP，提供完整 Locomotion 状态机与上半身 Pose 合成框架 |
| `ABP_HumanoidEnemy` | `Content/Enemy/Phantom/Animations/Logic/`（ABP_Phantom 等） | 人形怪 ABP，Parent Class = `UHumanoidEnemyAnimInstance` |
| `UBaseLocomotionAnimInstance` | `Source/.../Characters/_Shared/Animations/BaseLocomotionAnimInstance.h` | 所有 AnimInstance 的 C++ 基类 |
| `UHumanoidEnemyAnimInstance` | `Source/.../Enemy/Humanoid/HumanoidEnemyAnimInstance.h` | 人形怪专属 AnimInstance |
| `UAnimNotify_TurnComplete` | `Source/.../Enemy/Humanoid/AnimNotify_TurnComplete.h` | Turn 动画末尾 → `AHumanoidEnemy::OnTurnComplete()` |

### ABP_HumanoidEnemy AnimGraph 节点链

```
PatrolSM（状态机）
  → Convert to Component Space
  → Two-Bone IK（hand_l 左手贴前握把）
      IKBone=hand_l；Joint Target=Bone Space/lowerarm_l；Effector=LeftHandIKTarget（CS）；Alpha=bHasWeapon
  → BBBAimIK（脊柱瞄准）
      AimSourceBoneName=AimSocket（hand_r 子插槽，+X 朝枪管）；AimAxis=(1,0,0)
      AimSourceLocalTransform ← 变量；AimTarget ← AimTargetComponentSpace；Alpha ← AimAlpha
  → Convert to Local Space
  → Output Pose
```

### PatrolSM 状态机结构

```
SA_CanTurn（State Alias = Idle + Patrol_Walk_Run）
   → Turn_L45/L90/L135 / TurnAround / R45/R90/R135 : bIsTurning && TurnAnimIndex==0..6
SA_Turning（全部 Turn 状态）→ Idle : bIsTurning==false（AnimNotify_TurnComplete 触发）
Idle ↔ Patrol_Walk_Run : Speed >/< 10（虚拟减速驱动，无 Stopping 状态）
Idle ↔ PatrolScan : bIsPatrolScanning
SA_PatrolStates → Aim : AIState==Aim（Aim 状态用 BS_HumanoidCombat，Direction/Speed 驱动）
Aim → Idle : AIState!=Aim（Blend 0.2s）
Any State → Dead : bIsDead
```

### Patrol_Walk_Run（BlendSpace1D，X=Speed）
| Speed | 动画 |
|---|---|
| 0 | Idle（过渡缓冲） |
| ~150 | Walk 循环 |
| ~350 | Run 循环（预留） |

**停步**：PathFollowing 到点强制 CMC 速度归零；AnimInstance 检测 `bIsStoppingAtPoint` 上升沿，`VirtualSpeed` 从 `LastWalkSpeed` 以 `StopDecelerationRate`(默认 300 cm/s²) 线性降到 0 并覆盖 `Speed`，blend space 平滑过渡，Speed<10 自动切 Idle。Aim 停车（速度骤降）共用同一套虚拟减速。

### PatrolScan（Blend Poses by Int，Active Child=PatrolScanAnimIndex）
- C++ 在 `bIsPatrolScanning` 上升沿随机生成 index（1 ~ `PatrolScanAnimCount`，从 1 避免 A-Pose）。
- `PatrolScanAnimCount` 在子 ABP Class Defaults 设置。

### Turn（7 个原地动画，无 Root Motion）
| TurnAnimIndex | 状态 | 角度范围 |
|---|---|---|
| 0 | Turn_L45 | (-67.5°, 0°) |
| 1 | Turn_L90 | (-112.5°, -67.5°) |
| 2 | Turn_L135 | (-157.5°, -112.5°) |
| 3 | TurnAround | \|角\| ≥ 157.5° |
| 4 | Turn_R45 | (0°, 67.5°) |
| 5 | Turn_R90 | (67.5°, 112.5°) |
| 6 | Turn_R135 | (112.5°, 157.5°) |

实际旋转由代码动画期间平滑插值 Actor Yaw；`AnimNotify_TurnComplete` 挂动画末尾通知结束。

### UHumanoidEnemyAnimInstance 变量

继承 `UBaseLocomotionAnimInstance`（Speed/Direction/bIsFalling/AimPitch），追加：

| 变量 | 类型 | 说明 |
|---|---|---|
| `AIState` | `EHumanoidEnemyAIState` | Patrol / Aim / SearchRush / SearchScan / Dead |
| `bIsTurning` / `TurnAngle` / `TurnAnimIndex` | bool/float/int32 | 转身触发 / 有符号角度 / 0-6 索引 |
| `bIsDead` | bool | Any State → Dead |
| `bIsPatrolScanning` / `PatrolScanAnimIndex` / `PatrolScanAnimCount` | bool/int32/int32 | 扫视开关 / 随机索引(从1) / 总数(EditDefaultsOnly) |
| `StopDecelerationRate` | float | 虚拟减速率（cm/s²，默认 300） |
| 左手 IK：`LeftHandIKTarget` / `bHasWeapon` / `WeaponGripLeftSocket` | FVector/bool/FName | Two-Bone IK 目标 / Alpha / 握把 socket(grip_l) |
| AimIK：`AimTargetComponentSpace` / `AimSourceLocalTransform` / `AimAxisSocketName` / `AimAxis` / `bHasValidAimTarget` / `bIsAiming` / `AimAlpha` / `AimAlphaInterpSpeed` | — | BBBAimIK 脊柱瞄准（详见 11-enemy-ai.md / FEAT-031） |

**私有内部变量（不暴露 ABP）：** `LastWalkSpeed` / `VirtualSpeed` / `bIsVirtualDecelerating`（虚拟减速）；`bAimSourceInitialized`（AimSource 首帧初始化标志）等。

### State Alias 约定
| Alias | 代表状态 | 用途 |
|---|---|---|
| `SA_CanTurn` | Idle / Patrol_Walk_Run | 这些状态下可发起转身 |
| `SA_Turning` | 全部 Turn 状态 | 转身结束统一回 Idle |

### 扩展指南（人形怪）
- **新增具体人形怪**：继承 `ABP_HumanoidEnemy`，Asset Override 覆盖动画，Class Defaults 设扫视数量，无需重搭状态机。
- **Search 状态（FEAT-026）**：状态机加新大状态，过渡基于 `AIState`，新变量在 AnimInstance 追加。
- **射击蒙太奇插槽（FEAT-027）**：AnimGraph 输出链末尾插 Slot 节点（`EnemyUpperBody` / `EnemyFullBody`），骨骼 Slot Manager 注册同名。

---

## 动画重定向项目边界

- 主项目 `TheManTest` 只保存已经完成重定向并验收的最终动画资源。
- 禁止在主项目中创建或执行 IK Retargeter、批量重定向、源骨架/源 Mesh 导入等操作，也不得保留重定向中间目录。
- 重定向必须在资源来源项目中完成，验证后仅将最终动画迁移至主项目。
- 外部资源项目：`D:\Unreal Projects\TMIIR`、`D:\Unreal Projects\FPSShooter1`。
- Shooter 动画必须在 `FPSShooter1` 内完成所需重定向，再迁移最终产物。

---

## 旧系统（双骨骼，C++ 已删除，仅历史参考）

> `ABP_MainCharacter` 曾对应 `ATheManCharacterBase`（双骨骼：GetMesh 全身 + FirstPersonMesh 手臂 + HeadCamera 挂 head 插槽）。
> **C++（`ATheManCharacterBase` / `UTheManAnimInstanceBase`）已于 FEAT-041 删除**（备份 scratchpad/deprecated-char-backup-session43）；`ABP_MainCharacter` / `ABP_FirstPerson_MainCharacter` 蓝图待编辑器删除。下文仅作历史参考。

旧 AnimGraph 流程（已不使用）：
```
Cache_Locomotion → WeaponUpperBody → Slot"UpperBody" → WeaponAimOffset → Slot"FullBody" → Root
```

旧动画实例 `UTheManAnimInstanceBase`（已删除）变量（仅 Locomotion）：

| 变量 | 类型 | 说明 |
|---|---|---|
| `Speed` | `float` | 地面移动速度（Size2D） |
| `Velocity_Z` | `float` | 垂直速度 |
| `bIsFalling` | `bool` | 是否在空中 |
| `AimPitch` | `float` | 瞄准俯仰角，归一化到 [-1, 1] |
| `Direction` | `float` | 移动方向角 [-180, 180] |
## 2026-08-01 session149 — 玩家统一 AnimBP 修正

- MaintenanceWorker 的 `CharacterMesh0` 与 `ArmsViewMesh` 必须统一使用 `ABP_MaintenanceWorker`；不得再为第一人称直接挂独立 VFX AnimBP。
- `ShadowBodyMesh` / `LegsMesh` 是 `CharacterMesh0` 的 Leader Pose 跟随者；影子姿势以主身体最终 Pose 为准。
- RepairGun 的 `ABP_RepairGun_AnimLayer` 同时链接到 `CharacterMesh0` 和 `ArmsViewMesh`。基础链为 `DefaultSlot -> Layered Blend Per Bone.BasePose`，武器层只覆盖配置的上半身分支，再进入 WeaponAimOffset/Slots。
- 当前 WalkRun 状态直接播放 `AS_Rifle_A_Run`（0.5×）；这是因为已验证 BlendSpace Player 在该模板继承链运行时返回参考姿势。不要重新接回旧 BlendSpace，除非先在 PIE 同时证明身体、第一人称和影子均输出非参考 Pose。

## 2026-08-02 session152 — VFXPack 第一人称上半身例外

- 用户明确要求先忽略下半身速度、让玩家视角上半身与 VFXPack 一致；MaintenanceWorker 的 `ArmsViewMesh` 因此直接使用已整理的 `ABP_VFXPack_FirstPerson`，复用原版 Idle/Run 状态机和播放速度。
- `CharacterMesh0`、`ShadowBodyMesh` 与 `LegsMesh` 继续保留项目主 ABP/Leader Pose 链，下半身 locomotion 不在本次调整范围内。
- 这是 RepairGun/VFXPack 第一人称视图的明确特例；不要据此把其他角色或武器改成独立第一人称 AnimBP。

## 2026-08-02 session153 — 移动反馈与投影 Pose 同源

- `ABP_VFXPack_FirstPerson` 的 Idle/Run 继续负责主要武器姿态偏移；C++ 只复刻原 Walking/Running CameraShake 的频率、振幅和淡入状态，输出改写到 `ViewmodelRoot`，不启动 gameplay CameraShake。
- 已删除此前自创的移动方向位移/旋转和鼠标旋转滞后，避免与原动画重复叠加。
- 运行时逐骨审计确认旧 `ShadowBodyMesh` 虽正确 Leader=`CharacterMesh0`，但因此复制了与第一人称不同的身体 locomotion 上半身。现改为 Leader=`ArmsViewMesh`；`spine_03/hand_r/hand_l` 组件空间 Pose 与 VFXPack 第一人称逐项相同。`LegsMesh` 仍 Leader=`CharacterMesh0`，下半身速度体系本轮不改。

## 2026-08-02 session156 — VFXPack 同源 AnimBP 最终修正

- session153 的 `ShadowBodyMesh Leader=ArmsViewMesh` 已撤销。MaintenanceWorker 恢复本项目既有的同源方式：`CharacterMesh0` 与 `ArmsViewMesh` 使用同一原版 VFXPack AnimBP 类，各自拥有 AnimInstance；Shadow/Legs 均 Leader=`CharacterMesh0`。
- VFXPack `Walk_Run_1D` 只有 Speed 轴。方向姿态由原角色 Body_Sway 写入 AnimBP，不是 2D BlendSpace：原始目标为 `Clamp(MoveRight+MouseX,-1,1)` 和 `Clamp(-MoveForward-10×LookUp,-1,1)`，Walk 插值速度 2、Sprint 8；写入 Modify Bone 前原 EventGraph 还精确乘以 `Lean_Sides_Offset=8.0` / `Look_Up_Offset=2.0`。
- session224 当前边界：按用户决定恢复 `e1c24eb` / session220 的完整75° `RemappedLeanRoll/RemappedLookPitch` 骨骼映射。A/D 继续通过 `spine_03/hand_l` Modify Bone 链带动手臂与枪；不旋转装备 Actor，也不移动 `ArmsViewMesh` 做圆弧补偿。该版本倾斜观感获用户认可，但保留枪口随骨骼枢轴上下平移的已知问题。
- 普通移动 Body Sway 的进入/回弹速度由 `AFPSCharacterBase.ViewmodelBodySwayInterpSpeed` 控制，Class Defaults 分类为 `Viewmodel|Movement`，默认 `6.0`；冲刺保持 `8.0`。
- 静态第一人称构图不再使用自定义 Offset 属性；所有位置与朝向只编辑蓝图组件树中 `ArmsViewMesh.Transform`，C++ 不覆盖 Arms Transform。
- 冲刺压枪使用独立 `SprintVisualAlpha`：按实际 `Velocity.Size2D()` 在 `WalkSpeed..SprintSpeed` 映射0..1并驱动 `ViewmodelRoot.Pitch`。`ViewmodelRoot` 不用于静态构图；Shift 意图的 `SprintTransitionAlpha` 只切换 MaxWalkSpeed，原地 Shift 不触发视觉下压。
- `Viewmodel Offset Location` 与 `Viewmodel Arms Rotation` 支持 PIE 实例 Details 编辑；`PostEditChangeProperty` 在编辑发生时立即应用一次。Tick 不轮询或覆盖这两个静态参数；BeginPlay 仍负责初始应用。
- C++ 必须把 `Is_Moving`、`Is_InAir`、`Character_Speed`、`Lean_Sides_Amount`、`Look_Up_Amount` 同帧写给两个 AnimInstance，避免第一/第三人称及影子上半身再次分叉。
- session157 修正：前后倾斜的原版 AnimBP 变量实际名为 `Look_Up_Amount`，不是 `Look_Up_Down_Amount`。方向倾斜必须缓存 Enhanced Input 原始移动轴，A/D → `Lean_Sides_Amount`、W/S → `Look_Up_Amount`；不得只用 CharacterMovement 速度反推后就以变量数值代替最终姿势验收。正式 AnimGraph 为 `spine_03` Additive Roll/Pitch，加上 `hand_l` 的 0.5× Additive Roll。
- session165：取消项目补充的 5cm `ViewmodelRoot` Y 向横移；A/D 最终视觉只来自上述 AnimBP 骨骼链，构图根节点不再随侧移输入改变位置或旋转。
- session166：冲刺恢复原蓝图职责：按键意图驱动 0.2s 可逆时间线，同时插值 MaxWalkSpeed 550→750 并旋转 `BodyRotator` 等价节点 `ViewmodelRoot` Pitch 0→-12.5°；不再把压枪角写入 `spine_03` 的 `Look_Up_Amount`。
- session259-260：`ViewmodelRoot.RelativeLocation` 增加纯位置式鼠标观察滞后，默认水平 1.8cm、垂直 1.2cm、进入速度 12、回正速度 16、死区 0.01。BeginPlay 保存蓝图 authored RelativeLocation，Tick 只写“该基准 + 瞬时滞后偏移”，停止输入后回到蓝图基准；不得把构图原点强制归零。右看时 Viewmodel 左移、上看时下移；不增加旋转，不修改 `ArmsViewMesh` 静态 Transform，也不影响相机、瞄准或弹道。旧停用的 Arms Pitch Follow 字段和无效鼠标缓存已删除。
- session261：位置滞后由目标插值升级为欠阻尼弹簧。鼠标 delta 只施加一次反向速度冲量，之后通过刚度 85 / 阻尼 12 自然回正；默认水平/垂直冲量 7/5，位移仍限制在 1.8/1.2cm。Enhanced Input 向上为负 Y，因此垂直冲量同号写入 Viewmodel Z，保证向上看时枪先向下。纯位置、原构图基准及其他系统边界不变。

## 2026-08-04 玩家最终统一链

> HISTORICAL / SUPERSEDED by FEAT-077 and current names; not an implementation instruction. - MaintenanceWorker 四个 SkeletalMesh 均指定 `ABP_CharacterBase_Body` 最终骨架子类；Shadow/Legs Leader=`CharacterMesh0`。
> HISTORICAL / SUPERSEDED by FEAT-077 and current names; not an implementation instruction. - RepairGun 遵循统一动画架构：`ArmsViewMesh` 与 `CharacterMesh0` 使用同一 `ABP_CharacterBase_Body` 主 AnimBP，装备时同时链接同一个 `ABP_RepairGun_AnimLayer`；不再存在 `BodyEquipmentAnimLayerClass` 或 `ABP_RepairGun_BodyAnimLayer` 分流。维修枪 Idle/Run 使用外部批准资源项目 `FPSShooter1` 生成并迁入的方向修正最终资产 `AS_VFXPack_FP_Idle/Run`。Shadow/Legs 继续跟随 `CharacterMesh0`。
- MaintenanceWorker 身体资产的视觉正前方为 Mesh 局部 `+Y`；`CharacterMesh0`、`ShadowBodyMesh`、`LegsMesh` 必须使用蓝色 Z/Yaw `-90°`，把视觉 `+Y` 对齐 Character/Actor 的 `+X` 前向箭头。第一人称 `ArmsViewMesh` 的相机空间构图旋转保持独立，不得用它反推或覆盖身体朝向。Unreal Python 的 `Rotator` 位置参数为 roll/pitch/yaw，脚本写 Yaw 时必须使用第三个参数。
- 初始装备完成链接后，在首个渲染帧前对 CharacterMesh0 与 ArmsViewMesh 执行零时长动画评估，避免冷启动入口 Pose 闪帧。
> HISTORICAL / SUPERSEDED by FEAT-077 and current names; not an implementation instruction. - 无引用旧 `ABP_CharacterBase` 已删除；正式运行类仅保留模板 `TABP_BodyLocomotion` → 最终 `ABP_CharacterBase_Body` → RepairGun Linked Layer。

## FEAT-089 Executive drone animation

/Game/Characters/TheExecutive/Drone/Animations/ABP_TheExecutive_Drone derives from UTheExecutiveDroneAnimInstance. Three looping sequence players feed BlendListByInt using native FlightPose (0Idle,1TurnLeft,2TurnRight),0.3s blend. Pawn yaw rate selects turning above18deg/s and returns to idle below8deg/s. No retargeting or root-motion locomotion. Attack/Landing/Death sequences are migrated but intentionally not connected until those mechanics are authorized.


## FEAT-105 current Phantom native rig (2026-09-17)

This supersedes FEAT-084 mesh/animation configuration and FEAT-080 stationary target configuration. BP_Phantom uses `/Game/Enemy/Humanoid/Phantom/Meshes/SK_Phantom_Native`, its native Skeleton/PhysicsAsset, and `Animations/Logic/ABP_Phantom_Native`. Mesh scale1, relative Z=-90/Yaw=-90. The 31 finalized sequences live in `Phantom/Animations/Native`; two BlendSpaces in `Phantom/Animations/BlendSpaces`. OriginalRifle still owns the live M4_Rifle_01 mesh and materials; its unreferenced animation prototypes were removed by FEAT292. Final animations were retargeted in TMIIR and grip-corrected in external Blender only; never retarget inside TheManTest.

Weapon attaches with identity transform to hand_r_wepSocket on hand_r_wep. Native AimSocket and measured rifle Muzzle socket initialize existing AimIK. Both arm poses/support contact are baked into final clips; reload retains release motion. Four native BlastRifle sequences drive the existing reaction component with horizontal root displacement. Burst/SuppressiveFire/Reload ability references use native clips.

BP_Phantom disables StationaryHitTest, restores shared BP_HumanoidAIController, and uses Patrol150/Combat300/Turn50/SearchRush600 cm/s. SpaceStation saves Phantom_Patrol and four Phantom_Patrol_01..04 points in Gameplay/PhantomPatrol, around X1800..2400/Y300..900, with Nav_PhantomPatrol. Existing PlayerStart is preserved. Temporary validation cameras/lights must not be saved.

## FEAT-122 native fire animation contract (2026-09-17)

First-person graphs now read inherited `UCharacterBaseAnimInstance::FireAnimationWeight`; do not recreate the old local `Arm_Group_Animation_Weight_Multiplier` variable. Graph links and concrete class defaults are preserved across all ten affected AnimBPs. The unused worker `_0` duplicate was removed after checking for references. `UTheManAnimationAssetLibrary::MigrateFireAnimationWeight` is an editor-only migration implementation: it redirects variable nodes to the native field, removes obsolete declarations, compiles, and restores the saved default. Gameplay uses the typed property directly; it does not invoke this migration helper.

## FEAT-126 Infiltrator independent first-person layer (2026-09-18)

BP_InfiltratorLaserRifle now points to its own Animations/FirstPerson/Logic/TABP_InfiltratorLaserRifle_FirstPerson -> ABP_InfiltratorLaserRifle_FirstPerson. The copied graph retains the existing WeaponAimPoseAlpha ADS blend and full Idle/Move/JumpStart/FallLoop/Land behavior. Concrete sequences and BS_InfiltratorLaserRifle_FirstPerson_WalkRun belong to this owner; no Laser/Worker animation playback references remain. Shared interface and existing compatible Skeleton remain shared. Owner Fire/Equip montages use owner Fire; unmodified owner FireBase preserves the original additive reference pose. Six nonadditive clips have authored upper/lower arm and hand rotations that move the held gun down7cm/right3cm with camera-space yaw-10deg, preserving bone translations and rigid two-hand contact. Host/character transforms and other weapons remain unchanged. This is same-skeleton pose editing, not retargeting.

Editor helper ReplaceMontageAnimation now updates montage linkable elements after replacing segments, so sections/notifies do not retain stale source sequence caches. RestoreLocomotionOverrides accepts this owner when remapping duplicated parent graph GUIDs. Runtime does not call either helper. Actual PIE and cold saved-pose verification: D:/UnrealWork/InfiltratorFinish.

FEAT127 follow-up: the same six Infiltrator nonadditive clips now use tighter lower-right framing and a5cm forward support-hand position. Hand/gun orientation remains exactly126 across196 samples (max rotation delta3.7e-14deg). Only positional framing and arm joint rotations needed to reach it changed; bone lengths and shared skeleton unchanged. See09-equipment-system for hand_l attachment compensation. No new C++ or animation graph changes.

FEAT127 final user-approved direction change: add5deg clockwise camera-space cant around right-hand pivot to hip poses (camera-X -5deg before existing yaw-10deg). This supersedes the prior exact-orientation preservation request. Framing translation/support-hand compensation unchanged. Existing ADS alignment returns the sight upright. Six owner pose clips updated; evidence D:/UnrealWork/InfiltratorCant.


## FEAT128 approved Infiltrator pose (2026-09-18)

FEAT128 supersedes127 pose values. Six owner nonadditive clips derive idempotently from original source poses: camera correction(-8,7,-14.2)cm with RY(-5)*RX(-5)*RZ(5), then gun camera-relative positions/size scale .65. Both wrists target mesh-space contacts recorded in09; left elbow pole camera(0,-1,-.7), forearm twist aligned from wrist orientation. Clavicle and bone translations unchanged; all196 sample contacts exact within1.69e-13cm. Fire additive/FireBase and AnimBP graph remain unchanged. Owner arm proximity override in03 prevents old40cm fade clipping the visible forearm. Same-skeleton editing only, no retargeting. Actual PIE and cold evidence D:/UnrealWork/InfiltratorApprovedPose.

FEAT128 latest feedback: level reference gun cross-axis against world-up using fixed3.504deg roll correction around gun forward direction; retain authored yaw/pitch/origin. Left contact is now(6,14,-11)cm. Mesh relativeZ4.722728 (X/Y unchanged), matching aim/muzzle localZ compensation. Six owner clips and weapon BP updated. Authoritative scripts/checks D:/UnrealWork/InfiltratorLevelPose supersede prior ApprovedPose values.

2026-09-19 model5 follow-up: six InfiltratorLaserRifle owner nonadditive clips now use compact-rifle vertical-grip poses. Prior FEAT128 contact values are superseded for this gun. No shared character/worker clips, skeletons, graph hierarchy or additive Fire/FireBase changed. Socket compensation is baked into Compact mesh orientation with identity component rotation; see current arch09 and D:/UnrealWork/A1553Install/solve_pose.py.


## User-authored camera framing applied (2026-09-19)

User-saved gun framing applied to the six owner Still/Idle/Run/JumpStart/JumpLoop/JumpEnd clips. Character ArmsViewMesh base transforms, shared skeleton/socket, animation graphs and additive Fire/FireBase remain unchanged. Same-skeleton local-rotation editing only; no retargeting. The old mesh installation translation/scale is baked into Compact vertex positions, preserving UV/topology/material slots. BP_InfiltratorLaserRifle.StaticMesh is now Location(0,0,0), Rotation(0,0,0), Scale(1,1,1). Aim and muzzle transforms retain their correct root-space coordinates, not zeroed. Original Glass/Emitter and laser .27/2.4/.9 retained; no surface texture/material edits in this batch.

Rigid camera delta = user target * inverse(previous camera-relative Gun). Transform both wrist targets and orientations by this delta in Arms space, then solve upper/lower/hand rotations while retaining local translations/scales. Six owner clips only; no shared skeleton, socket, character transform or retargeting assets changed. Current normalized preview and pose source: D:/UnrealWork/GunFramingApply/solve.json and corrected-poses.json. Supersedes previous A1553Install pose solver output. D:/UnrealWork/GunFramingApply: author.json, generic.json, visual.json (18 actual PIE checks), four native gameplay captures, cold.json, pose-cold.json (1568 rotations), preview-cold.json (target preserved, mount identity, zero material overrides), framing-match.json. Actual hip target error .001684cm and .005785 degrees. Solver wrist error <=1.50e-13cm across392 samples; local bone translations/scales and clavicles unchanged. New Blueprint cold load and scoped zero redirectors verified. No C++ edits/build required.


2026-09-19 canonical mesh-axis correction supersedes the baked compensation above. Six GunFramingApply poses remain unchanged; equipment actor attachment compensates the canonical SM. Firearm passes MuzzleLocalTransform * GetEquipAttachmentTransform() to the linked animation layer because its aim source is socket-space. No character/skeleton/socket/animation edits. See arch09 and GunAxisFix evidence.


## InfiltratorScopedRifle animation owner (2026-09-19)

User-saved target is camera-relative location(24,18,-23)cm, quaternionxyzw(-.0173242734,.0680859016,-.7160806879,.6944729137), scale1. Captured class and new saved preview instance agree; user replaced the earlier preview actor. Created independent Still/Idle/Run/JumpStart/JumpLoop/JumpEnd, Fire/FireBase, Fire/Equip montages, WalkRun blend space and template/concrete animation layer. Six nonadditive poses fit new rifle contact targets; only local clavicle/upperarm/forearm/hand rotations authored, preserving bone translations/scales, shared skeleton/socket, character ArmsViewMesh base and old gun clips. Same-skeleton editing, no retargeting. New mesh component remains identity; explicit EquipAttachmentTransform handles the socket basis. Source grip solver and exact transforms: D:/UnrealWork/InfiltratorScopedRifle/solution.json and corrected-poses.json. Additive Fire uses its own FireBase and both montages reference the new owner's Fire. No playback dependency on old owner remains.

LaserRifleAssetLibrary::RestoreLocomotionOverrides now also accepts this owner path. Editor-only graph GUID remapping; gameplay animation logic unchanged. Parent TABP and child ABP are duplicated under the new owner and all sequence/blendspace overrides remapped. Validation in InfiltratorScopedRifle/cold.json and visual.json.

## FEAT-140 Executive turret animation
Owner: Characters/TheExecutive/Drone. ABP_TheExecutive_DroneTurret derives from UTheExecutiveTurretAnimInstance: PoseIndex(Deploy/Idle/Fire),PoseTime,YawRotation,PitchRotation. Three SequenceEvaluators -> BlendListByInt with zero transition time -> LocalToComponent -> ModifyBone AimYaw -> ModifyBone AimPitch -> ComponentToLocal. Deployment samples the clip using the same progress as the material scan; successful shots select the short native recoil clip. The base stays fixed. Component mesh has yaw=-90degrees and Z46.55cm ground offset; pitch axis is transformed into the yawed mesh component frame.
Approved source native Take001 was split externally into Deploy(-90..-1,2.9667s),Idle(11..12,.0333s),Fire(1..10,.3s), rebaked to positive frame numbers. Runtime export merges six meshes into one6517-vertex skin and adds TurretRoot/AimYaw/AimPitch/Muzzle for23bones. This is final native turret animation, not retargeting. No IK rig/source retarget content imported. Original approved blend/download untouched.
External final export must bake centimeters into mesh vertices, rest bones and local animation translations, apply armature/mesh object transforms, and preserve normalized deformed positions against source samples. Simply exporting Blender Armature object scale/rotation caused UE animation-scale/axis mismatch. Source: D:/Blender Projects/DownloadedTurretLED/ApprovedLED/export_runtime.py and Runtime final FBXs. Material slot arrays must be converted to Python lists before mutating struct entries and reassigned; mutating temporary UE array element copies left saved slots empty.

FEAT-140 v0.14 Fire recoil refinement: existing Fire sequence reimported from ApprovedLED/Runtime/RecoilTuned/TheExecutive_DroneTurret_Fire.fbx, authored by export_recoil.py externally. Only Bone004/Dummy001/Bone006/Dummy002 local translation deltas from native Idle scaled0.30; rotations, clip0.3s, endpoints, mesh, other clips and ABP unchanged. Independent184 pose checks pass. Original Runtime Fire is preserved historical strong recoil; use tuned source for future reimport. Resumed live audit verifies4.9041cm forward recoil step (30%); the separate Idle/Fire close-target aim oscillation was subsequently fixed in v0.15. FEAT-140 is complete.

FEAT-140 v0.15: no ABP or clip changes. Native turret aiming now recovers stable offsets using the rotations stored by the evaluated UTheExecutiveTurretAnimInstance. Previous animated muzzle-to-target absolute-angle feedback caused near-target oscillation even during Idle; offset-aware analytical solving removes it. AimYaw/AimPitch pivot sockets and evaluated rotations are required. Runtime forward recoil remains4.9041cm/frame at60Hz; Fire source remains Runtime/RecoilTuned.


## FEAT-146 native Infiltrator gameplay (2026-09-23)
Infiltrator now uses its original lobby native target skeleton/fullbody plus owner FirstPerson/Meshes/SKM_Infiltrator_FirstPersonArms and Body/Meshes/SKM_Infiltrator_LowerBody. Arms/body run ABP_Infiltrator_FirstPerson and ABP_Infiltrator_Body under their owner Animations folders. This supersedes worker-asset reuse for Infiltrator only; Worker/Executive unchanged. Body retains locomotion/hidden shadow and copies native upper-body pose; legs follow body. Native graphs replace spine/hand names and omit independent left-hand sway to preserve two-hand contact. The two Infiltrator gun BPs use native first-person layers/montages under Characters/Infiltrator/Animations/FirstPerson/{LaserRifle,ScopedRifle}, attach to Bip01-R-Hand with independently measured offsets. Default loadout remains silver only. Shared templates and gun optical/gameplay properties unchanged. Finalized locomotion20clips and weapon16clips were adapted externally in TMIIR; no runtime/destination retargeting. Four native arm materials add16cm/8cm camera-distance dither clipping; original lobby/body materials unchanged. Editor authoring helper is Characters/Infiltrator/Editor/InfiltratorAnimationAssetLibrary; gameplay does not call it. See FEAT146 archive and external InfiltratorNativeWeapons evidence.


## FEAT-149 native role integration (2026-09-23)
Characters/_Shared/Editor/CharacterNativeAnimationAssetLibrary is editor-only graph authoring support shared by Worker and Executive. CreateNativeGraph flattens inherited templates with node overrides, replaces animation assets and native bone names; FinalizeNativeGraph removes independent support-hand ModifyBone sway. Runtime does not call it. Native body/host/layer graphs are in owner Animations/*/Native. No retarget work inside destination. See archive/FEAT-149-native-worker-executive.md and D:/Blender Projects/WorkerExecutiveNative for evidence.

FEAT-149 user correction: Executive has exactly ONE weapon, the native BP_TheExecutive_ExecutiveSlowRifle. Extra Executive-only InfiltratorLaserRifle/ScopedRifle weapon and animation folders were deleted, not available loadout options. Worker keeps its three guns. Obsolete /Game/Weapons/RepairGun/Meshes/SM_RepairGun deleted and its static-component references cleared in both RepairGun BPs; actual SKM_RepairGun remains. This supersedes earlier six-profile/secondary-Executive framing notes.

FEAT235 path update: the skeleton-free shared postprocess now serves Phantom and Medusa and is moved with AssetTools to /Game/Enemy/_Shared/Animations/Logic/ABP_Enemy_HitReaction. Old Humanoid path is obsolete. Per-enemy HitReactionConfig selects the postprocess.

FEAT235 final shared animation graph: raw SequenceEvaluator pose is cached, converted against cached incoming pose via ApplyAdditive or ApplyMeshSpaceAdditive according to EnemyHitReactionAnimInstance flags, or passed through for absolute sequences. The converted pose feeds the prior full/upper-body blend. Medusa and Phantom configs reference /Game/Enemy/_Shared/Animations/Logic/ABP_Enemy_HitReaction; the old Humanoid-only asset path is retired.

FEAT237 editable ABP_Medusa graph: Directional BlendSpace (MovementDirection X, MovementSpeed Y) and Turning BlendSpace (RotationRate X) -> TwoWayBlend(TurnWeight) -> DefaultSlot -> Result. MedusaAnimInstance derives shared locomotion variables without changing other enemy/player ABPs. Reparent authoring regenerates skeleton class before binding new inherited variable pins. Both new BlendSpaces and eleven native sequence copies are Medusa-owned. Shared hit postprocess remains a separate overlay.


FEAT246 movement ownership: ABP_Phantom_Native uses Root Motion from Montages Only. Ordinary navigation must drive CharacterMovement velocity; prior Everything mode consumed small animation root drift and blocked normal patrol. Sequences, retargeting, blend graph and montage displacement are unchanged. See arch/15-enemy-movement.md and FEAT246 evidence.


FEAT271 procedural blink: Infiltrator FirstPersonArms mesh uses owner BlinkV6/Animations/ABP_Infiltrator_BlinkPose as postprocess. Its graph is InputPose -> Result. Parent UInfiltratorBlinkPoseAnimInstance computes view-relative offsets on the game thread; FInfiltratorBlinkPoseProxy snapshots them in PreEvaluateAnimation, evaluates the original graph, then solves native right/left UpperArm-Forearm-Hand chains with AnimationCore::SolveTwoBoneIK. Both hand targets receive one rigid transform about the right wrist; a shared reach limit retains two-hand contact without stretching. Only six arm bones change; spine/clavicles/finger local poses remain unchanged. Moving Spine4 was rejected because it exposed chest geometry in first person. Pose ramps18ms, holds through70ms, returns by340ms. View-space defaults(6,-1,2)cm and pitch3/yaw0/roll-1.5degrees convert to component space. No blink AnimSequence/Montage, source skeleton, rig or retargeting. Existing scoped InitializeBlinkPoseGraph helper can rebuild only this BlinkV6 graph and remove the obsolete ModifyBone chain.


FEAT274 adds three same-skeleton AS_MaintenanceWorker_Lobby_DisintegrationGun_* clips under Worker/Lobby/Animations with underhand support at gun scale0.7; source ExplosionGun clips unchanged. Independent Gameplay/FirstPerson/Animations/Native/DisintegrationGun now contains8 fitted same-skeleton clips, its own ABP/blend and equip/fire montages. Target is approved0.7 framing at(13,22,-25), Pitch3/Yaw-7/Roll0/FOV90. Only arm-chain local rotations changed; bone translations/scales preserved. Fire references owner FireBase. Actual movement/fire/switch checks passed; visual review remains available. No retargeting or source rigs added.

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

FEAT289: `TABP_Enemy_Locomotion`, `ABP_HumanoidEnemy`, and `ABP_Medusa` route their final pose through `HitReactionSlot` after the existing `DefaultSlot`, then to Output Pose. Concrete Medusa and Phantom skeletons register that slot in `HitReactionGroup`; child AnimBPs inherit the connection. Existing `ABP_Enemy_HitReaction` remains the Post Process provider for Data Assets using Animation mode. Montage mode plays through the main AnimInstance and does not drive the Post Process sequence sample. See `Documentation/Enemy/EnemyHitReactionModes.md`.

FEAT290 clarification: Post Process Animation samples a sequence, so `bApplyRootMotion` permits the component to extract its Root Motion and move the capsule; a request's `bInPlace` suppresses that move. Rule/global Post Process blend times and rule `MaxDuration` control that sampled path (and Control Rig where applicable). Montage instead uses its authored blend, duration and Root Motion together with the main AnimBP Root Motion mode; only shared `PlayRate` applies. DA tooltips and `Documentation/Enemy/EnemyHitReactionModes.md` state this split. The team-facing template document now includes the saved `HitReactionSlot` after `DefaultSlot`.

FEAT291/292: Production `BP_Medusa` uses `ABP_Medusa_Locomotion` (child of the skeleton-free template) and linked `ABP_Medusa_SpecialPose` (turning BlendSpace over incoming locomotion). Production `BP_VoidGazer` uses `ABP_VoidGazer`, a Sequence Evaluator driven by its native AnimInstance's selected clip/time, then `DefaultSlot` and `HitReactionSlot`. Void's native proxy evaluates this graph before the existing body and procedural-tentacle transforms. The pre-migration Medusa ABP and temporary template example were deleted in FEAT292. See `Documentation/Enemy/EnemyAnimationTemplate.md` for paths and authoring steps.


## Authoritative humanoid Turn graph (2026-10-12)
The saved template /Game/Enemy/Humanoid/_Shared/Animations/Logic/ABP_HumanoidEnemy has a SINGLE Turn state (historical seven separate states/SA_Turning description above is stale). ABP_Phantom_Native inherits this template and retains its existing relaxed child overrides. SA_CanTurn includes Idle, Patrol_Walk_Run and now Aim; bIsTurning enters Turn with.04s crossfade. Turn retains existing seven-player relaxed BlendByInt, selects authored alert SequenceEvaluator(TurnAnimation,TurnAnimationTime,nonlooping/teleport) when bUseAuthoredTurn. SA_PatrolStates includes Turn; exit to Aim is bEnterAim=Aim&&!bIsTurning. Turn->Idle is bExitTurnToRelaxed=nonAim&&!bIsTurning; these exits.08s. Dynamic alert selection supports separate L/R45/90/135/180 without duplicating states or placing skeleton assets in the template. Main IK/locomotion/death paths unchanged. Scoped editor helper UHumanoidTurnAnimationLibrary under Enemy/Humanoid/Editor configures/inspects this existing template; no retargeting functionality. Removed temporary ABP_Phantom_HitAndTurn and UPhantomHitReactionAnimInstance; shared reaction-only postprocess restored.

## FEAT293 shared Enemy native AnimInstance (2026-10-14)

`UEnemyAnimInstance` under `Enemy/_Shared/Animations` derives from `UBaseLocomotionAnimInstance` and is the native parent contract for production Enemy AnimBPs. It inherits `Speed`, `Direction`, `Velocity_Z`, `bIsFalling`, `AimPitch` and adds `bIsMoving` (planar speed >5 cm/s). `TABP_Enemy_Locomotion` directly parents to it; `ABP_Medusa_Locomotion` inherits that skeleton-free template. `UHumanoidEnemyAnimInstance` (Phantom), `UMedusaAnimInstance` (Medusa special layer) and `UVoidGazerAnimInstance` derive from it. Species pose graphs retain their own BlendSpace/state machine/Sequence Evaluator, while production graph outputs preserve `DefaultSlot -> HitReactionSlot`. Void's native proxy remains responsible for procedural tentacles. See `Documentation/Enemy/EnemyAnimationTemplate.md`.

## FEAT294 Phantom humanoid graph alignment (2026-10-14)

Phantom's production `ABP_Phantom_Native` is a skeleton-bound child of the skeleton-free `ABP_HumanoidEnemy`. Humanoid parent owns the Patrol/Aim/Turn state machine, left-hand IK and AimIK. Its main graph had `HitReactionSlot` but no `DefaultSlot`; FEAT294 inserted the common skill Montage slot before the hit slot; its graph contract and 7 rendered Phantom tests passed. See `Documentation/Enemy/EnemyAnimationTemplate.md` and the FEAT294 archive for verification state.

## 2026-10-15 shared optional turn and special layer

UEnemyAnimInstance owns RotationRate, TurnWeight and bEnableTurnPose. Humanoid and Medusa enable the optional turn pose by default; Void does not. Humanoid bIsTurning is gated by the flag; Medusa's existing turning layer consumes the common values and retains its idle-action guard. The formal Humanoid and Void graphs now include the same ALI_EnemySpecialPose linked layer immediately before DefaultSlot; its unbound implementation passes the incoming pose through. Medusa already uses this interface in the shared locomotion template with ABP_Medusa_SpecialPose bound. Species movement pose sources remain their existing BlendSpace, humanoid state machine and Void Sequence Evaluator; Void's native proxy still handles procedural tentacles. All formal graphs retain DefaultSlot -> HitReactionSlot.
