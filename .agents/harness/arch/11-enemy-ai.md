# 敌人 AI 与战斗系统

FEAT-299 onboarding scaffold: `/Game/Enemy/Nightmare/TrainingEnemy` owns two meshless Nightmare training enemies (A is `BP_TrainingEnemy`, B is `BP_TrainingEnemy_B`). Each has independent controller, Blackboard, Combat/Ambient trees, engagement profile and one empty `PhaseSkillSets` phase. Both use shared `BT_EnemyLifecycle`, automatic-distance `BTTask_UseCombatSkill` in parallel with `BTTask_GroundMovementPreset(KeepPlayerDistance)`, and Ambient `RandomPoint`. The native controller and editor tree builder live under `Source/TheManTest/{Public,Private}/Enemy/Nightmare/TrainingEnemy/`. Dev Panel uses one test placement preset per enemy. See `Documentation/Enemy/TrainingEnemyOnboarding.md`; empty animation/skill slots are not a production enemy template.

**何时读取：** 修改敌人巡逻、转身、感知、战斗状态、行为树、技能集、GAS 或动画驱动时。

## 当前架构（FEAT-234 / FEAT-282，2026-10-13）

`UEnemyEngagementComponent` 和 Profile 负责激活、目标、追击与记忆范围、离巢限制，以及 `Dormant → Activating → Engaged → Recovering → Dead` 生命周期。控制器和行为树读取这些状态；它们不自行决定目标归属。具体规则见 [16-enemy-engagement.md](16-enemy-engagement.md)，移动职责见 [15-enemy-movement.md](15-enemy-movement.md)。

四类现有敌人控制器均继承 `AEnemyAIController`。`StartLifecycleTree` 用所属家族的 Blackboard 初始化控制器，运行共享主树 `/Game/Enemy/_Shared/AI/BT_EnemyLifecycle`。主树观察 `Engagement.Lifecycle`，状态改变时中止旧分支；`Activating` 等待原有入场流程，`Engaged` 运行 Combat 子树，`Recovering` 运行可选 Recovery 子树，`Dormant` 运行可选 Ambient 子树。缺少可选子树时走等待分支。`UBTTask_EnemyLifecycleSubtree` 通过 `RunBehaviorDynamic` 执行控制器选定的子树。共享 `BB_EnemyLifecycle` 拥有 `TargetActor`、`LastKnownPlayerLocation`、`Lifecycle`、`Activity` 和 `MoveLocation`；家族 Blackboard 以它为父级。

人形控制器配置了 Combat 树时，`OnPossess` 启动共享主树；启动失败会报告原因，最多尝试 20 次、间隔 0.25 秒，然后报错并保持安全空闲，不切回 C++ 自主巡逻。未配置 Combat 树的专用预览仍走原有无树路径。主树 `Activating` 分支已纳入路由回归测试。

| 家族 | Combat | Ambient / Recovery 与特殊边界 |
|---|---|---|
| Humanoid | `BT_HumanoidEnemy` | `BT_Humanoid_Ambient` 管巡逻及失去目标后的回退；`BT_Humanoid_Recovery` 管恢复搜索。静止预览 `BP_Phantom_WeaponTest` 不配置行为树，也不自动寻路。 |
| Medusa | `BT_Medusa` | 原有入场钩子保留；无专属 Ambient / Recovery 子树。 |
| VoidGazer | `BT_VoidGazer` | `BT_VoidGazer_Ambient` 管回墙、跳跃、观察和重新选位。 |
| CoreMorph | `BT_CoreMorphBoss`（由 `ScorpionMovement.BehaviorTree` 提供） | `BT_CoreMorph_Ambient` 在等待期间启动并维持飞行 GA。入场或重组能力先取得飞行姿态再取消飞行；蝎子运动、遭遇入场及变形条件未满足时不运行 Combat 子树。 |

### 当前动作选择与共用 Task（2026-10-15）

任何 Enemy 战斗行为改动，先冷读目标 BP 已保存的 `PhaseSkillSets` 与实际 BT，再列出“阶段 × 近/中/远距离 × 地面/墙面等形态”的候选表。`PhaseSkillSets` 是要由技能集随机选的动作的配置来源；攻击、追击并攻击、跳离、跳向玩家、回墙、墙面随机移动和跨墙跳跃等有开始/结束/取消的动作，由 GA 包装后放进需要的每个距离池，不能只靠固定优先级 BT 分支。某动作横跨多个距离档时，在各档分别登记；`AnyDistanceAbilities` 与 `BackgroundAbilities` 是显式独立池，不会自动并入 Near/Mid/Far。Ambient 和生命周期子树有独立职责，不误作战斗候选池。

开工前把上述候选表、每项 GA 的地面/墙面资格与失败重选条件、BT 节点的等待时间写进当前功能 archive。若现有技能池或通用 Task 已覆盖需求，沿用其入口；确需另建固定优先级分支或专属 Task 时，先记录现有入口不适用的具体原因。

| 通用入口 | 当前职责与核对点 |
|---|---|
| `BTTask_EnemyLifecycleSubtree` | 按 `Engagement.Lifecycle` 执行控制器配置的家族 Combat/Ambient/Recovery 子树。 |
| `BTTask_UseCombatSkill` | 从 Blackboard `TargetActor` 及当前阶段/距离档发起 `AEnemyBase::UseRandomSkill`，等待并仅取消自己启动的 GA；`PostSkillDelay` 和 `UnavailableSkillRetryDelay` 是节点上的停顿，不是 Idle 技能。 |
| `AEnemyBase::UseRandomSkill` | 将所选池洗牌、逐项 `TryActivateAbilityByClass`；失败继续试同池其他候选。形态、范围、冷却、LOS、落点等资格由 GA/其依赖组件核验。新 GA 若激活后立即取消，必须确认选择器能识别并重选，不能假定所有自取消都会自动视为失败。 |

`BTTask_UseCombatSkill.bAutoDistanceRange` 当前按水平距离 Near `≤500 cm`、Mid `>500 且 ≤800 cm`、Far `>800 cm`；也可在节点固定 `Range`。这两个自动阈值写在 C++ Task，不是所有敌人可在 BP 独立调节的参数。全部候选不可用时按 Task 的 `UnavailableSkillRetryDelay` 等待重选或立即失败。选技能与选点属于不同职责；点查询、移动预设、执行器清单见 [15-enemy-movement.md](15-enemy-movement.md)。

Void 的当前 Combat BT 用单个自动距离 `BTTask_UseCombatSkill`，其 BP 阶段池重复配置需要跨档使用的技能；地面近距可选追击并攻击、跳离玩家和回墙，中远距可选触手、跳向玩家和回墙，墙面技能包括触手、跳向玩家、随机移动和跨墙跳跃。实际条目及适用形态必须以保存的 BP/GA 为准。Void 普通近战 GA 是追击并攻击动作的执行者，在 `UseRandomSkill` 中跳过独立随机抽取。墙面技能在地面被抽到应拒绝并继续尝试同池候选；若整池均不可用，要测量 Task 重试与后摇等待，避免将原地停留误诊为动画 Idle。

死亡和解除占有会停止共享 Brain；没有死亡行为树分支。死亡阶段由 `DeathPresentation` 管理，敌人蓝图可在 `AEnemyBase` 的四个 `BlueprintNativeEvent` Overrides 中补充专属死亡处理，详见 `07-character-classes.md`。攻击仍由 GAS 管理，各家族保留自己的移动实现。CoreMorph 的攻击状态属于活动 GA，BT 移动任务调用 `ScorpionMovement`；人形巡逻和转身的细节仍由 `AHumanoidEnemy` 实现。

### 新敌人接入规则

需要通用地面行为时，在家族子树中使用 `BTTask_GroundMovementPreset`，按节点选择 `RandomPoint`／`KeepPlayerDistance`／`FollowActor`；`AEnemyBase.GroundMovement` 负责 NavMesh 完整路径验证、`AIController::MoveTo` 请求和仅取消自有 `FAIRequestID`。BT 只决定何时启动/中止和传入目标。Keep 从 `TargetActor` 读取交战目标，并遵守 Engagement 的最后已知位置；Follow 优先读 `FollowActor` Blackboard 键，未设置时读组件实例 `ConfiguredFollowActor`，不会自动跟随玩家。人形 Combat 使用 Keep；Ambient 顺序为配置 Actor 跟随、原路点/搜索、无路点随机游走、等待。Medusa 和 Void 保留各自必要的策略适配，不能据此跳过通用点查询/预设复用检查。墙面与空中现有入口见 [15-enemy-movement.md](15-enemy-movement.md)。

1. 控制器继承 `AEnemyAIController`，以 `BT_EnemyLifecycle` 为主树；家族 Blackboard 继承 `BB_EnemyLifecycle`。
2. 把家族战斗逻辑放在独立 Combat 树；需要时再配置 Ambient 和 Recovery 树。控制器引用这些子树，不直接把家族树作为主树运行。
3. 让 `EnemyEngagementComponent` 管理激活、目标和生命周期；攻击放在 GAS，移动放在该敌人的移动实现中。
4. 冷加载检查资产引用，并在 PIE 验证 `Dormant`、`Activating`、`Engaged`、`Recovering`、死亡及分支中止。静止测试预览可不配置 Combat 树，不作为普通敌人的接入模板。

### 新敌人的技能、表现与组件边界（用户要求，2026-10-13）

- 攻击和有明确开始／完成／取消的特殊动作由 GAS Ability 拥有生命周期、互斥状态和冷却；需要逐帧执行时使用 AbilityTask。BT 只选择动作并等待／中止它拥有的技能实例。不得让常驻 Combat 组件另起一套攻击计时、冷却或自动调度。
- 技能造成的持续或一次性视听特效优先由 Gameplay Cue 拥有生成、更新与清理。持续效果可由 Actor Gameplay Cue 或 Cue 创建的临时 Actor 承载，结束、取消和死亡都要清理。Cue 不负责目标选择、伤害、受害者位移或技能成败。不要仅因特效需要 Tick、蓝图参数或清理，就给每类敌人新增专属常驻表现组件。
- 组件用于有实际复用消费者的持续能力和底层执行，例如多个爬墙敌人共用的墙面支撑检测、胶囊扫掠与移动。GA 可以调用此类组件作为 motor；组件不自行决定攻击技能的激活和冷却。只有一个敌人使用的状态机先放进其 GA／AbilityTask；确需组件时记录无法由 GA、Task、Cue 或已有 motor 清楚承担的职责。
- 新组件进入 `_Shared` 前必须有至少两个真实消费者；不得仅为减少蓝图组件数量而把物理查询或复用运动能力塞入 GA。新增敌人应按上述归属逐项审查组件、技能与 Cue，并验证中止、死亡和解除占有时的清理。

新敌人实现前，先列出每个动作的激活者、运行状态所有者、冷却所有者、逐帧执行者、表现所有者和可复用底层能力；同一状态只能有一个所有者。若打算新增常驻组件，写明它服务的第二个实际使用者，或说明它为何无法由 GA、AbilityTask、Cue、敌人本体或现有 motor 承担。蓝图中要配置的参数放在对应的拥有者上，资产迁移时逐项比对旧 CDO 值并冷加载检查。

VoidGazer 的实现以 [FEAT-283 archive](../archive/FEAT-283-void-gas-ownership.md) 为准：`AttackVFX` 和 `Combat` 专属组件已删除。`GA_VoidMelee`／`AbilityTask_VoidMelee` 管近战计时、转向、命中和冷却；`GA_VoidTentacle` 管 Windup／Locked／Extend／Hook／Whip／Slam／Retract、锁定目标、挣脱与命中状态，AbilityTask 接收专属组件的 PostPhysics 回调并推进 GA，组件保留受害者扫掠位移、输入恢复和原生触手形变；`GA_VoidWallLeap` 管 Start／Air／Land、计时、结果及 GAS 冷却，AbilityTask 推进 GA，Leap 组件保留轨迹、落点支撑和碰撞验证。EyeCharge Cue 的临时 Actor 拥有眼部粒子、共享预警贴花及备用 Niagara 触手实例的生成、更新与清理；触手组件只计算端点和动画数据，仍保留现有 BP 的 Effect 配置引用。Landing／TentacleImpact Cue 拥有一次性特效；敌人只保存贴花的非拥有引用以供目标位置更新。`WallMovement` 保留为可复用爬墙 motor。历史条目描述了当时实现，不作为新敌人复制组件结构的模板。

架构变更和验证记录见 [FEAT-282 archive](../archive/FEAT-282-enemy-lifecycle-tree.md)。敌人动画见 `06-animation.md`，GAS 技能见 `10-gas-abilities.md`，属性和死亡见 `04-gas-attributes.md`。

## 历史实现记录

以下内容保留历次实现背景，含已删除的测试设施、被覆盖的状态流程和当时的待办；不能作为现行接入步骤。遇到冲突时，以本页“当前架构”、对应系统文档和实际源码为准。

> 当前清理状态（2026-09-13）：用户要求立即清理测试设施，取代下文旧批次“总验收后清理”的安排。5张CoreMorph检查地图、专属地面材质、5个Tests cpp与CoreMorphFlightReview h/cpp均已删除。Boss的StartFlightPreview／ResetFlightPreview及Flight的ResetPreview已删除；Reassembly内部ResetPreview改名ResetAssembly，仍用于正式变形初始化。ReviewTarget已删除，正式BT目标由LastThreat／玩家决定。保留FlightRoute、正式主BT、GA／GE／Cue及490个头领资产。以后只有TestMap作为测试地图，不能再照旧地图入口操作。


- FEAT-081 第三批主 BT：`ACoreMorphAIController` 真正 Possess 唯一头领；`BT_CoreMorphBoss`／`BB_CoreMorphBoss` 位于头领 AI 目录。Selector 内蝠鲼分支 Flight GA→Reassemble GA；蝎子分支按范围／冷却选择 Face→当前阶段近距技能、Approach 或 Idle。`BTTask_CoreMorphAction` 等待并取消具体活动能力句柄；攻击三个动作阶段属于 GA，不拆成三个 BT 技能。`BTService_CoreMorphTarget` 选择检查目标／LastThreat／玩家并更新形态和距离，形态与阶段独立。
- 八足运动沿用源的地面接触、身体扫掠和局部扇形避障，不是 NavMesh 全局寻路；尚未实现绕过任意复杂障碍的路径规划。ScorpionCombat.bEnabled 默认 false，检查地图 V/M 开启；旧飞行／重组检查地图保持原入口。取消、目标销毁、死亡和退出必须同时清 GA／GE 与运动状态；完整三枪适配仍属第四批。

- 当前OnDeath不立即Destroy：先取消ASC技能、停止AI Brain/移动并解除Controller，清Actor计时器/波次订阅，关闭角色Tick与胶囊，进入布娃娃；CorpseLifetime默认5游戏秒后Destroy。Humanoid AIState置Dead，武器网格无碰撞并跟手部骨骼；Phantom取消隐身。UseRandomSkill拒绝死人。

---

## 类层级

- 2026-09-05：Humanoid基类自动接共享骨架无关后处理ABP，不再依赖Phantom资产槽。ExplosionHitReaction可配置38度幅度、.055秒攻击、.85秒恢复、.045秒头肩跟随延迟、7cm腿部缓冲及骨骼映射。Rig降低pelvis并解析求解双腿，使脚保持当前输入动画位置而非固定世界位置，允许移动时受击。共享Rig不含Phantom模型/骨架导入引用。

```
AEnemyBase（Public/Enemy/）  ← 所有敌人基类，ASC+属性挂自身（无 PlayerState）
  ├── AHumanoidEnemy（.../Enemy/Humanoid/）  ← 人形怪：巡逻/转身/战斗/武器/AI
  │     └── APhantom（.../Humanoid/Phantom/）  ← 幻影（空壳，差异化在蓝图）
  ├── ANightmareEnemy（.../Enemy/Nightmare/）  ← 梦魇（直接继承基类，空壳）
  └── ABossEnemyBase（.../Enemy/Boss/） ← 头领公共语义层，不放具体变形
        └── ACoreMorphBoss（.../Boss/CoreMorph/） ← 蝠鲼／蝎子，FEAT-081 分批迁移中
```

---

## 文件清单

### CoreMorph 头领（FEAT-081，第一批）

- 第二批开始：用户已验收飞行观感，授权迁入变形重组。具体头领持有 ReassemblyComponent；飞行结束后的自动变形衔接只在新 L_CoreMorphReassembly Review 地图启用，正式决策仍等待第三批专属 BT。GA／GE／Cue 管理变形事务和表现，完成后停在静态蝎子，不启动源独立移动 Actor。

- `Enemy/Boss/BossEnemyBase.h` 为公共语义层；`CoreMorph/CoreMorphBoss.h/.cpp` 持有具体形态和飞行组件，继承一份 ASC／Health。没有人形骨骼、击退、布娃娃或血肉受击 Cue。
- `CoreMorph/Movement/CoreMorphFlightPath` 只保留源 FEAT058 参考路线的位置数学；`CoreMorphFlightComponent` 管理 154 个同 Owner 静态分件。Actor／Capsule 在编辑态直接位于身体主体，分件用相对变换跟随摆放；用逆起飞偏移乘 Actor Transform 推导参考路线坐标，BeginPlay 固定该坐标并将分件切为绝对世界姿态，实际 Pawn 根位置跟随飞行。CharacterMovement 在飞行预览中关闭。检查地图根位置为 (-16000,0,2500)，对应原路线原点 (0,0,900)，静态装配位置不变。
- `CoreMorph/GAS/Abilities/GA_CoreMorphFlight` 管理飞行生命周期；`GAS/Effects/GE_CoreMorphManta` 持有形态 Tag。阶段仍走 EnemyBase 的 `PhaseSkillSets/SetCombatPhase`，预览复位不重置阶段、Health 或技能授予。
- 第一批全身反馈：`FCoreMorphFlightMotion` 只接收世界位置／DeltaTime，从速度、升降、加速和转向计算身体朝向／侧倾及翅膀动作；尾巴按距离追随有界三维运动历史并叠加错相摆动。种子控制平滑随机节奏、波幅、左右差异，刚性核心仍与躯体统一运动。旧 TailMotion 已合并删除，SourcePose／DivePose 固定动作编排已移除。暂停／取消／死亡冻结，复位清空状态与轨迹。
- 可选 `ACoreMorphFlightRoute` 提供可在编辑器修改的 Spline，FlightComponent.FlightRoute 选择路线、RouteSpeed 控制速度；从首点开始，开放路线末端结束、闭合路线循环，无固定表现时点。空引用仍走源参考路径。MotionRandomness 默认 .18、0 关闭，MotionSeed 默认 0（复位选新种子）、非零可复现。当前是路线跟随与实时表现，正式战斗 AI／避障仍属于后续工作。
- 正式资产 `/Game/Enemy/Boss/CoreMorph/{Blueprint,Data,Meshes,Materials}`。`BP_CoreMorphBoss` 引用 `DA_CoreMorphVisualLayout` 和已有 `GE_EnemyBase_Init`。
- 本批尚未接战斗 BT，检查地图直接请求飞行 GA；默认源参考路线在 13.4 秒粒子释放前冻结，自定义 Spline 按实际末端／循环条件结束。主 BT／形态子树、变形、蝎子运动和攻击按后续批次接入；不得将当前预览误认为完整战斗。
- 检查入口 `/Game/Maps/CoreMorph/L_CoreMorphFlight`，V 播放、R 复位、P 暂停、F 切换相机；控件只属于地图专属 `CoreMorph/Review/CoreMorphFlightReview`。
- 多路线入口 `/Game/Maps/CoreMorph/L_CoreMorphRoutes`，同一 Boss／ASC／GA 配三个 Spline；Review.Routes 保存路线引用、英文 Label 与速度。PIE 自动开始 Gentle Climb（约 20 秒），1／2／3 分别重播 Gentle Climb／S-Turns（约 22 秒）／Orbit and Dive（约 27 秒）。切换先取消旧 GA，再复位并启动新路线；原地图 Routes 为空，不新增键绑定或自动播放。
- Motion 接收提前加速意图和 PowerStroke；Spline 先挥翼再逐步提速，因此实际时长增加起步发力与加速耗时。慢／快轴向侧滚属于当前飞行 GA 内的表现机动，组件拒绝非飞行／暂停／死亡时请求，随机触发由距离和实时状态决定。阶段、形态及技能授予逻辑不变；编译与实际 PIE 侧滚、取消／死亡／退出检查通过。

### 基类 / 属性 / 技能集

| 文件 | 关键内容 |
|---|---|
| `Public/Enemy/EnemyBase.h` | ASC + `UEnemyAttributeSetBase` 挂自身；通用屏幕空间 `EnemyHealthBar` 组件绑定 Health/MaxHealth；`InitGEClass`；`DefaultAbilities`（常驻技能）；**技能集系统**：`PhaseSkillSets`(阶段数组)/`CurrentPhase`/`SetCombatPhase()`/`UseRandomSkill(Target,Range)`/`GrantAbilities()`/virtual `AimAtTarget()`；`CurrentStrength`；`OnDeath()`。FEAT-073 新增 `ReactToProjectileHit` 与可刷新限时 `ApplyMovementSlow`；`SetDesiredMaxWalkSpeed` 将状态基础速度和临时倍率分离。`HitReactionCueTag` 由每类敌人配置，Health 实际扣减后由目标 ASC 执行受击 Cue。默认 `GameplayCue.Character.Enemy.Hit` 对应 Registry 可发现资产 `/Game/Enemy/_Shared/GAS/GameplayCues/GC_Character_Enemy_Hit`，作为所有敌人的公共兜底；同 Tag 不会自动区分角色，需要独立表现时应新增如 `GameplayCue.Character.Phantom.Hit` 并在具体敌人 BP 覆盖。`Tick` 默认关闭。 |
| `Private/Enemy/EnemyBase.cpp` | BeginPlay：InitAbilityActorInfo(self,self) + 应用 InitGE + 初始化/刷新头顶血条 + 授予 DefaultAbilities & 所有阶段技能 + 强度初始化（绑 `OnMidRoundStrengthIncrease`）；`UseRandomSkill`（当前阶段+距离档随机→AimAtTarget→TryActivateAbilityByClass）；`OnDeath` 隐藏血条后默认 Destroy |
| `Public/Enemy/UI/EnemyHealthBarWidgetBase.h` | `AEnemyBase` 共用的轻量原生血条 Widget；黑色边框/暗红底/亮红当前血量，无需敌人 Blueprint 资产 |
| `Public/Enemy/EnemyAttributeSetBase.h` | 继承 `UTheManAttributeSetBase`，怪物专属属性在此扩展（当前为空） |
| `EEnemySkillRange`（EnemyBase.h 内） | 交战距离档枚举：`Near` / `Mid` / `Far` |
| `FEnemyPhaseSkillSet`（EnemyBase.h 内） | 一个阶段的技能集：`NearAbilities` / `MidAbilities` / `FarAbilities`（各 `TArray<TSubclassOf<UGameplayAbility>>`） |

### 人形怪

| 文件 | 关键内容 |
|---|---|
| `Public/Enemy/Humanoid/HumanoidEnemyTypes.h` | `EHumanoidEnemyAIState`：`Patrol` / `Aim` / `SearchRush` / `SearchScan` / `Dead` |
| `Public/Enemy/Humanoid/HumanoidEnemy.h` | 重开 `Tick`；`WeaponMesh`(StaticMesh，挂 `WeaponAttachSocket`=hand_r)；`AimTargetWorld`/`bIsAiming`(public，AIController 写)；`AIState`/`SetAIState`；巡逻路点 `PatrolPoints`；速度参数(PatrolWalkSpeed/CombatWalkSpeed/TurnWalkSpeed)；转身/减速参数；重写 `AimAtTarget` |
| `Private/Enemy/Humanoid/HumanoidEnemy.cpp` | C++ 巡逻：`MoveToNextPatrolPoint`/`OnPatrolMoveCompleted`/`TryTurnOrMove`/`RequestTurn`/`OnTurnComplete`/`ResumeNearestPatrol`；`SetAIState`(Aim:停巡逻+Focus 朝向+加速；回 Patrol:置 bNeedsPatrolResume)；`Tick`(转身旋转 + 接近路点线性减速)；`AimAtTarget`(写 AimTargetWorld)。FEAT-073 起，玩家子弹命中会立即面向玩家、进入 Aim，并写 AI Focus/Blackboard TargetActor；所有状态速度均通过减速倍率入口。 |
| `Public/Enemy/Humanoid/Phantom/Phantom.h` | `APhantom : AHumanoidEnemy`；二阶段透明材质、弹体通道穿透、`ShouldProjectilePassThrough` |
| `Public/Enemy/Components/EnemyMagazineComponent.h` | 通用 20 发弹匣；仅普通自动射击消费，支持空匣判断、Reload 与 AmmoChanged |
| `Public/Enemy/Cover/EnemyCoverPoint.h` | 通用掩体 Actor；StandPoint + 距离/威胁背向/Visibility 遮挡评分选择 |
| `Public/Enemy/Nightmare/NightmareEnemy.h` | `ANightmareEnemy : AEnemyBase`，空壳（非人形，无巡逻/武器逻辑） |

### AI 控制器 / 行为树

| 文件 | 关键内容 |
|---|---|
| `Public/Enemy/Humanoid/HumanoidAIController.h` | `UAIPerceptionComponent`；黑板 key static const `BB_TargetActor`("TargetActor") / `BB_LastKnownPlayerLocation`("LastKnownPlayerLocation")；`BehaviorTree`(蓝图子类指定) |
| `Private/Enemy/Humanoid/HumanoidAIController.cpp` | Sight 感知；发现玩家进入 Aim；丢失时写 LastKnown、清 Target/Focus 并调用公共 `StartLostTargetSearch` |
| `Public/Enemy/BTTask_UseCombatSkill.h` | **通用战斗放招节点**：`Range`(近/中/远) + `TargetActorKey`(默认 TargetActor)；读黑板目标→`AEnemyBase::UseRandomSkill`；不绑定具体技能 |
| `Public/Enemy/Humanoid/BTTask_ResumeNearestPatrol.h` | 丢失目标回巡逻：调 `AHumanoidEnemy::ResumeNearestPatrol`（找最近路点续巡逻） |

### FEAT-080 临时 Phantom 静止测试配置

- 2026-09-05：APhantom新增默认开启的`Phantom|Testing → Stationary Hit Test`。BeginPlay解除并销毁自身AI控制器、清角色计时器、关闭角色Tick与Movement；ReactToProjectileHit跳过父类转向/Focus，Mesh与受击组件仍更新。适用于固定方向受击测试。取消勾选并重启PIE恢复原生AI路径；以下测试树/零速度配置仍需恢复才回正式战斗。

- `/Game/Enemy/Humanoid/Phantom/AI/BT_Phantom_TestIdle`：`Root -> Sequence -> Wait(86400s)`，无移动和攻击节点。
- `/Game/Enemy/Humanoid/Phantom/AI/BP_Phantom_TestIdleAIController`：公共 `BP_HumanoidAIController` 的 Phantom 专用测试副本，运行静止树。
- `BP_Phantom` 当前临时指向该测试 Controller，四项状态速度与 CharacterMovement `MaxWalkSpeed` 均为 0，用作爆炸弹命中/范围逻辑的静止目标。
- 恢复正式 AI 时改回共享 `BP_HumanoidAIController_C`，并恢复 `Patrol/Combat/Turn/SearchRush = 150/300/50/600`、`MaxWalkSpeed=600`。

### 动画（详见 06-animation.md）

| 文件 | 关键内容 |
|---|---|
| `Public/Enemy/Humanoid/HumanoidEnemyAnimInstance.h` | 继承 `UBaseLocomotionAnimInstance`，缓存 `AHumanoidEnemy`；输出 `AIState`/`bIsTurning`/`TurnAngle`/`TurnAnimIndex`(0-6)/`bIsDead`/`bIsPatrolScanning`/`PatrolScanAnimIndex`；停步虚拟减速 `StopDecelerationRate`；左手 Two-Bone IK(`grip_l`)；BBBAimIK 变量(`AimAlpha`/`AimAxis`/`AimSourceLocalTransform`/`bIsAiming`…) |
| `Public/Enemy/Humanoid/AnimNotify_TurnComplete.h` | 挂转身动画末尾 → 回调 `AHumanoidEnemy::OnTurnComplete()` 清 bPendingTurn |

---

## AI 状态机（EHumanoidEnemyAIState）

- `Patrol` 巡逻（未发现玩家）——**移动由 C++ 自驱**（MoveToActor 循环 + 转身），BT 只 Wait。
- `Aim` 锁定追击（发现玩家）——停 C++ 巡逻；`SetFocus` 锁朝向（bUseControllerRotationYaw）；CombatWalkSpeed；**移动+放招由 BT 驱动**。
- `SearchRush` 丢失后以 `SearchRushSpeed` 冲向 LastKnownLocation；到达转 `SearchScan`。
- `SearchScan` 复用 Relaxed Fgt 随机环视，`SearchScanDuration` 后找最近巡逻点恢复；无 Nav/Move 失败时安全回 Patrol。
- `Dead` 死亡。

状态切换入口：`AHumanoidEnemy::SetAIState()`（负责停计时器/StopMovement/改朝向模式/置 bNeedsPatrolResume）。

---

## 巡逻系统（C++ 驱动，FEAT-028/024/029）

- `PatrolPoints`（EditInstanceOnly，关卡实例填 `APatrolPoint`）。`PossessedBy` 绑 `OnRequestFinished`，延迟启动首次 `MoveToNextPatrolPoint`。
- 到点：`OnPatrolMoveCompleted` → 等待(WaitTime)/扫视 → `TryTurnOrMove`（偏角 > 阈值先 `RequestTurn` 转身，`OnTurnComplete` 再续走）。
- Walk→Stop：接近路点 Tick 线性降速 + AnimInstance 虚拟减速，blend space 平滑过渡（FEAT-029）。
- 所有巡逻函数开头 `if (AIState != Patrol) return` 守卫——战斗时不被巡逻干扰。

---

## 感知 → 战斗流（FEAT-032，session40 启用）

```
Sight 感知玩家(1500cm/60°)
  → OnTargetPerceptionUpdated(成功) → BB.TargetActor=玩家 + SetFocus + SetAIState(Aim)
  → AIController::Tick 每帧 AimTargetWorld=玩家位置, bIsAiming=true（供 AimIK + 子弹方向）
  → BT 战斗 Sequence [Decorator: TargetActor IsSet, Abort Both]:
       MoveTo(TargetActor, AcceptanceRadius=交战距离)
       → BTTask_UseCombatSkill(Range) → UseRandomSkill(当前阶段对应档随机一个技能)
       → Wait(出招间隔)
丢失玩家(1800cm)
  → BB.LastKnownPlayerLocation + 清 TargetActor + ClearFocus
  → StartLostTargetSearch → SearchRush(MoveTo LastKnown) → SearchScan(Relaxed 随机环视)
  → SetAIState(Patrol) + ResumeNearestPatrol
```

### 行为树结构（BT_HumanoidEnemy，编辑器）

```
Root → Selector
  ├── Sequence [Decorator: Blackboard TargetActor Is Set, Observe aborts: Both]
  │   ├── Move To              (Key: TargetActor, Acceptance Radius: 交战距离)
  │   ├── BTTask_UseCombatSkill (Range: 近/中/远)
  │   └── Wait                 (出招间隔)
  └── Wait 0.1                 (无目标 → C++ 巡逻自驱)
```

> 距离分近/中/远三支（距离 Decorator）当前**未做**；战斗序列先放一个 BTTask_UseCombatSkill。

---

## 技能集系统（阶段 × 近/中/远，FEAT-035，放 AEnemyBase 通用）

- 二维：`PhaseSkillSets[阶段]` → `FEnemyPhaseSkillSet{Near/Mid/FarAbilities}`。`CurrentPhase` 默认 1，`SetCombatPhase()` 切换（第二阶段换整组）。
- `BeginPlay` 经 `GrantAbilities` 授予所有阶段所有档技能。
- `UseRandomSkill(Target, Range)`：从随机起点轮询当前阶段/距离档能力；空匣等激活条件失败时继续尝试同档其他能力。
- 技能 = `UGA_EnemyShoot` 蓝图子类（技能与子弹绑定，复用子弹管线；不同开火逻辑重写 `SpawnProjectiles`）。详见 `10-gas-abilities.md`。
- 触发不走 GameplayEvent，按类激活（`BTTask_UseCombatSkill` → `UseRandomSkill`），故一敌可多技能各自独立。

---

## 当前状态 / 注意

- ✅ 巡逻(FEAT-028)、转身(FEAT-024)、Walk→Stop(FEAT-029)、左手 IK(FEAT-030) 已验证。
- 🔶 FEAT-032（感知+BT 战斗）：session40 已启用 C++ 感知/Tick/BTTask，**编辑器 BT 战斗序列与 PIE 验证待完成**。
- ✅ FEAT-031（BBBAimIK 脊柱瞄准）：session40 恢复并验证通过，敌人 Aim 时上半身跟随玩家。（aim 取点当前用 GetActorLocation，如需更高改 eyes）
- 🔶 FEAT-035（敌人射击）：C++ 完成，待编辑器配 BGA_EnemyShoot + 武器 Muzzle socket + BP_Phantom PhaseSkillSets。
- ✅ FEAT-058～063：搜索链、通用掩体、弹匣/三连发/扫射/换弹、Phantom 找掩体与二阶段已实现；Phantom 通过 `PhaseSkillSets` 数据注入能力，公共 BT 不依赖 Phantom 类型。
- ✅ FEAT-071：Phantom 子弹 Damage=6；射击散布扩大为3°基础/0.8°逐发/9°上限，移动惩罚2°、恢复2°/s；每波伤害成长10%、最大倍率1.5。只覆盖 Phantom 蓝图，不修改通用 Enemy C++ 默认。
- FEAT-058 session126 曾记录无 NavMesh 直移 fallback，但不代表当前实现。2026-09-17 FEAT-113 核对：当前 MoveToNextPatrolPoint 使用真实 NavMesh 路径请求，没有巡逻直移 fallback；巡逻怪与 PatrolPoints 必须处于可达导航区域。ConfigurePatrolPoints 支持运行时配置，测试在 TestMap 临时生成导航，不保存设施。
- ✅ FEAT-064：Aim 状态移动由 `AHumanoidAIController` 的公共距离环带接管。BT 的 Actor `MoveTo(TargetActor)` 在 Aim 时只作为技能序列门槛并返回 AlreadyAtGoal；真实移动目标为 NavMesh 投影后的战术 Location。默认保持 700±150 cm：近距后撤+侧移、远距收拢+侧移、环带内切向绕行；无 NavMesh 时回退 CharacterMovement 直接移动。实际局部速度继续由 `UBaseLocomotionAnimInstance` 计算 Direction，驱动子 AnimBP 二维 Aim BlendSpace。

### CoreMorph 蓄力地面雷爆更新

尾刺仍由主 BT 请求当前阶段 NearAbilities 中的 `GA_CoreMorphTailStrike`，战斗阶段与形态不耦合。此技能现在锁定目标脚下可用静态地面：2 秒蓄力红圈预警 → 尾尖击地 → GA 结算一次球形 GE 范围伤害 → 收回。目标移动不会拖动本次红圈／伤害中心；刺出途中墙体阻挡不产生雷爆。视效由两个独立 Gameplay Cue 管理，不参与命中判定。全部求解仍位于这只具体头领的 ScorpionCombat／Movement，公共 Boss 层未增加专属逻辑。

### CoreMorph 空中远程决策（2026-09-13）

正式 BT_CoreMorphBoss 的 Manta 分支改为 SimpleParallel：主任务 Flight GA，背景 Selector 按 CanBombard 请求当前阶段 FarPhaseSkill，否则短暂 Wait；WaitForBackground 使飞行结束后等待当前背景技能，再 Reassemble。当前导弹 GA 已按用户要求从 PhaseSkillSets 装配移除，因此远程分支没有可激活的导弹能力；相关 BT／CanBombard 结构保留，待技能重新设计后复用。CanBombard 由原 Target Service 检查具体组件、Attacking／MissileCooldown Tag；远程分支使用 LowerPriority 中止，避免攻击 Tag 中止自身。
Near／FarPhaseSkill 任务比较启动前后的 Active Spec，保存自己新启动的技能句柄；任务中止仅取消自身技能，避免误取消并发飞行。Scorpion 分支保留。当前阶段为 Near=TailStrike、Far 暂空；默认 Flight／Reassemble。闭合路线持续飞行；后续重新设计远程技能时继续沿用正式 BT 入口。

## FEAT-089 friendly Executive drone

Owner-local classes live in Characters/TheExecutive/Drone, not Enemy. ATheExecutiveDroneAIController runs /Game/Characters/TheExecutive/Drone/AI/BT_TheExecutive_Drone with BB_ExecutiveDrone (Leader Object, FollowLocation Vector). UBTTask_TheExecutiveDroneFollow stays active and updates following goals. Movement is handled by UTheExecutiveDroneMovementComponent (acceleration-limited swept movement and slide); Pawn chooses direct owner-relative anchor, visible recorded owner-path points, then local sphere-checked detours. This is local avoidance, not general 3D global pathfinding. Hidden/lobby drone does not run gameplay AI. AI is destroyed with its Pawn; gameplay pause freezes movement.

Behavior tree owns follow decisions only. No drone GA/ASC, attack or damage implementation in this phase. Future Executive GAS integration remains separate work.

## FEAT-090 distance-only follow update

Gameplay no longer rotates its anchor or idle yaw with the player. FollowStartDistance defaults300cm and FollowStopDistance180cm, measured from owner position plus FollowOffset.Z. Start/stop hysteresis preserves a world-space hold point while nearby; when far, radial approach and existing local avoidance bring it closer. Only flight velocity steers gameplay yaw. FollowOffset.X/Y affect initial spawning only; Lobby still uses owner-relative LobbyOffset and its existing hover/full-turn logic. Tuning lives in BP_ExecutiveDrone Drone|Follow.


FEAT-092 (2026-09-16): SpaceStation places two existing BP_CoreMorphBoss instances with InitialForm Manta/Scorpion, each owning its normal single ASC and Controller. Only these map instances override ScorpionCombat.Enabled=true to run the existing main BT; shared BP defaults remain unchanged. Manta follows Station_MantaRoute closed spline at3000cm/s. Scorpion uses existing terrain adaptation and attacks on flat open ground. No general obstacle-routing or new abilities were added.


## FEAT-093 Boss encounter activation (2026-09-16)

- `ABossEncounterTrigger` lives in `Enemy/Boss`, independently placeable with an editable Box `TriggerVolume` and one instance `TargetBoss`. Only player-controlled Pawn overlaps activate. Initial overlap / delayed possession / failed opening-action retries are checked every 0.2 seconds until success; leaving does not reset the boss. A successful trigger disables its collision and tick.
- `ABossEnemyBase::ActivateEncounter(APawn*)` is the guarded BlueprintCallable entry: requires authority, completed BeginPlay, living boss, player Pawn and no prior/reentrant activation. `OnEncounterActivated` is a bool-returning BlueprintNativeEvent; C++ overrides `_Implementation`, Blueprint overrides the event. False leaves activation available for retry. Successful activation persists for the Actor lifetime.
- `WaitForEncounterTrigger` opts a boss into dormant startup; default false preserves existing placements. `IsEncounterActivated` reports the explicit one-shot event; `CanRunEncounter` additionally permits legacy automatic startup. Concrete bosses must honor CanRunEncounter in their AI startup and combat updates.
- CoreMorph honors the gate in its controller and ScorpionCombat. `ReassembleOnActivation` defaults false; true requests existing Reassemble GA if currently Manta, then enables combat and records the activating player. The opening action stops the old tree before requesting the GA. A new tree cannot start during transformation; after completion the usual Scorpion branch takes over. Form details remain outside BossEnemyBase. A failed GA request does not consume activation.
- Setup: place BossEncounterTrigger, adjust TriggerVolume extents, assign TargetBoss; enable WaitForEncounterTrigger on that boss and optionally ReassembleOnActivation. To extend behavior override OnEncounterActivated and return success only after accepting the opening action. Do not bypass the guarded ActivateEncounter entry.
- Humanoid perception, EnemyBase and GA implementations are unchanged. Per user follow-up, the two SpaceStation Boss instances were removed and cold verified; the Boss Blueprint and existing flight route remain. New placements opt in via WaitForEncounterTrigger. Tests use unsaved TestMap fixtures and external scripts in D:/UnrealWork/BossEncounter.


## FEAT-094 trigger Blueprint and beacon (2026-09-16)

BossEncounterTrigger now owns cosmetic BeaconColumn/BeaconHalo/BeaconRing static meshes (engine Cylinder/Plane), noncolliding and non-shadowing. Blueprint `/Game/Enemy/Boss/Encounter/Blueprint/BP_BossEncounterTrigger` configures dedicated procedural additive materials in `Encounter/Effects/Materials`. Details expose color, height (200m), column/ring radii, intensity, ShowBeacon and HideBeaconAfterActivation. Construction/BeginPlay update dynamic materials; accepted activation extinguishes the beacon by default. Visuals do not drive combat.

SpaceStation current placement supersedes FEAT-093 removal: one Station_Manta, dormant until linked Station_BossEncounterBeacon overlap, ReassembleOnActivation=true; Station_BossLanding supplies the previously verified open-ground landing location. No separately placed Scorpion. Existing route retained. Build, actual PIE activation/morph/AI, cold configuration and rendered beacon verified; see FEAT-094 archive and D:/UnrealWork/BossBeacon.


## FEAT-095 ambient Manta patrol (2026-09-16)

CoreMorph adds `Boss|Encounter.PatrolBeforeEncounter` (default false, editable when WaitForEncounterTrigger). After GAS/form initialization, a gated Manta with assigned FlightRoute starts existing Flight GA independently of the combat tree. Closed splines loop, open splines hold at endpoint; neither auto-transforms. Missing/invalid route logs once and holds. Combat remains gated. On accepted beacon, existing Reassemble GA captures live flight pose then cancels Flight; Scorpion combat tree starts after morph. Failed entry preserves patrol.

SpaceStation Station_Manta enables it, references existing Station_MantaRoute (8 closed points,110005.73cm,3000cm/s), and starts at spline first point. Beacon/landing unchanged. Full-loop, live-flight transition, failed-entry retry and active-patrol PIE exit verified; see FEAT-095. No common/dynamic tree refactor this batch. Recommended future family-level main tree separates NonCombat, EntryTransition, Combat; skill-set-only changes continue using PhaseSkillSets.

## FEAT-096 encounter climb and dive (2026-09-16)

SpaceStation Station_Manta now enables `Boss|Encounter.ClimbAndDiveOnActivation` alongside PatrolBeforeEncounter, WaitForEncounterTrigger and ReassembleOnActivation. This opt-in defaults false elsewhere. It supersedes the immediate transformation described above: beacon -> native `GA_CoreMorphEncounterEntry` -> source orbit/climb/crest/intact dive -> existing Reassemble GA -> Scorpion combat.

Entry cancels ambient Flight and any missile ability, retains FlightMotion/body/tail history, and starts FlightComponent.StartEncounterApproach at source orbit t3.55 with live heading/position and a1.5s velocity/clock blend. It reuses unchanged CoreMorphFlightPath mathematics, aiming the dive at Reassembly.GetLandingPoint (LandingReference plus ground probe). No new spline asset is needed for the entrance. OnFlightFinished at source release t13.4 starts reassembly. Entry owns cancellation and its completion delegate; controller holds combat until transformation finishes, and holds a cancelled/failed entrance instead of restarting flight. DefaultAbilities includes Flight, EncounterEntry, Reassemble; form/phase/health remain shared.

## FEAT-098 player flyby before ascent

This supersedes FEAT-096's direct patrol-to-orbit join. OnEncounterActivated snapshots the activating Pawn's location before starting Entry GA. Flight.StartEncounterApproach takes that snapshot plus the saved landing point: a cubic Hermite approach preserves live position/velocity and passes above the snapshot at EncounterFlybyHeight (default1600cm, editable on Flight). The endpoint matches source orbit t3.55 position and11215cm/s tangent, preserving scale; duration adapts to distance (3..20s). The simulation splits the exact join substep, retains Motion/tail history, and then advances the unchanged source orbit/climb/nose-up crest/dive math until reassembly release. Later player movement cannot steer the entrance. EncounterFlybyPoint exposes the captured destination. No new spline or saved map edit is needed. Combat/HUD/GA cleanup retain FEAT-096/097 behavior.


## FEAT-105 current Phantom native rig (2026-09-17)

This supersedes FEAT-084 mesh/animation configuration and FEAT-080 stationary target configuration. BP_Phantom uses `/Game/Enemy/Humanoid/Phantom/Meshes/SK_Phantom_Native`, its native Skeleton/PhysicsAsset, and `Animations/Logic/ABP_Phantom_Native`. Mesh scale1, relative Z=-90/Yaw=-90. The 31 finalized sequences live in `Phantom/Animations/Native`; two BlendSpaces in `Phantom/Animations/BlendSpaces`. OriginalRifle still owns the live M4_Rifle_01 mesh and materials; its unreferenced animation prototypes were removed by FEAT292. Final animations were retargeted in TMIIR and grip-corrected in external Blender only; never retarget inside TheManTest.

Weapon attaches with identity transform to hand_r_wepSocket on hand_r_wep. Native AimSocket and measured rifle Muzzle socket initialize existing AimIK. Both arm poses/support contact are baked into final clips; reload retains release motion. Four native BlastRifle sequences drive the existing reaction component with horizontal root displacement. Burst/SuppressiveFire/Reload ability references use native clips.

BP_Phantom disables StationaryHitTest, restores shared BP_HumanoidAIController, and uses Patrol150/Combat300/Turn50/SearchRush600 cm/s. SpaceStation saves Phantom_Patrol and four Phantom_Patrol_01..04 points in Gameplay/PhantomPatrol, around X1800..2400/Y300..900, with Nav_PhantomPatrol. Existing PlayerStart is preserved. Temporary validation cameras/lights must not be saved.


## 2026-09-17 follow-up integration

FEAT113: BP_Phantom_WeaponTest duplicates productionPhantom with bPatrolWeaponTest=true, bStationaryHitTest=false, plain native AIController and empty offensive ability sets. Existing Humanoid C++ patrol follows assigned PatrolPoints; emptyarray idles. Owner flag skips damage-induced aggro and forces cloakfalse, retaining damage/slow/death. OriginalPhantom flagfalse unchanged. No permanent map fixtures.

## Friendly Executive turret AI (FEAT-140)
This is a friendly stationary APawn + ASC, not AEnemyBase. All files/assets belong to Characters/TheExecutive/Drone. `BT_TheExecutive_DroneTurret` is a Sequence with0.2s `BTService_TheExecutiveTurretTarget`, `BTTask_TheExecutiveTurretFire`, then0.05s Wait. Blackboard: CombatReady(bool),TargetActor(object),AimPoint(vector). Task delegates attack authorization to the turret/GAS; targeting service maintains nearest visible living enemy and retains valid current target. Pawn Tick performs continuous skeletal aiming without moving its base.
`TheExecutiveTurretAssetLibrary::InitializeCombatTree` builds editable graph nodes with explicit NodeInstance/ClassData and locked graph updates, then verifies both task references and target service survived graph reconstruction. The initial generic SpawnMissingNodes graph lost task instances after save; do not revert to that authoring path. Final cold checks must find fire/wait/service instances, followed by actual autonomous PIE shooting.


## FEAT-153 lower Manta entrance (2026-09-24)

Flight.EncounterClimbHeightScale defaults0.25 (editable0.1..1). Separate private EncounterPathFrame compresses entrance path Z around EncounterFlybyPoint, and inverse-transforms the actual landing point before PrepareDive. Both source path sampling and approach endpoint velocity use this frame. ChoreographyFrame retains original body/piece scale; do not compress it to tune altitude. Patrol/reference flight and reassembly timing remain unchanged. This supersedes FEAT098 unchanged-height presentation: actual station climb is now114.88m instead of about460m. Full PIE flyby->crest->dive->reassembly->Scorpion combat verified; user aesthetic feedback pending.


## FEAT-154 Executive terrain placement (2026-09-24)

Turret.CanPlaceAt validates9 static, near-horizontal footprint samples with8cm height tolerance and accepts different supporting components (Landscape seams or adjacent floors). Optional SupportComponents output is populated only after successful clearance validation. Volume clearance and drone descent ignore this validated support set, not arbitrary ground actors. Drone45cm footprint support also accepts adjacent static components while keeping normal/plane bounds. Flight path, obstacle/edge checks and held E + LMB input retained. SpaceStation actual full deploy tested.


## FEAT232 Nightmare Medusa
Medusa is an independent ANightmareEnemy subclass, temporarily reusing Phantom visual/idle assets only. No Phantom AI/weapon/attack inheritance. Configurable proximity activates its single GAS nightmare presentation; attacks explicitly deferred. Hidden body remains collidable; player projectile hits trigger0.8s scan-gold reveal, refreshing on repeat hits. Death/destruction cancels darkness and restores shared runtime scene state. See FEAT-232 archive for verification.


## FEAT233 Enemy trigger visualization
Enemy/_Shared/Debug/UEnemyTriggerDebugSubsystem is a Game/PIE world subsystem, not created in Shipping. Default all categories off; category state survives panel/character changes within the world and resets on travel/end PIE. APhantom (including derived variants) uses actual possessed controller sight config/location/direction; view-cone boundary lies on SightRadius sphere. AMedusa uses live ActivationRadius (gray when automatic activation disabled). ABossEnemyBase shows associated ABossEncounterTrigger component boxes using scaled extent and rotation only when bWaitForEncounterTrigger; consumed boxes dim. No spatial trigger yields no fabricated radius. Dead/destroyed enemies excluded. Dedicated transient ULineBatchComponent is explicitly flushed/rebuilt every tick, also while paused, and destroyed on subsystem shutdown; other debug drawers untouched. All visualization settings are read-only to gameplay.

## Deferred activation architecture (2026-09-30; NOT IMPLEMENTED)

User accepted a shared activation/engagement component with configurable sight/proximity/trigger/hit conditions, common idle-patrol / optional entry / combat / disengage lifecycle, and enemy-specific behavior subtrees. Explicitly deferred for this batch. Inactive enemies may still patrol; skill lifetime is independent of engagement state. Do not describe this as current architecture or start the refactor as a prerequisite for Sevarog. Full decision and separate Sevarog intent: `../archive/FEAT-234-enemy-activation-lifecycle.md`.


## FEAT235 current combat contract (implementation/validation in progress)

Use the existing BTTask_UseCombatSkill for all Near/Mid/Far attacks. Display name is Use Combat Skill. PostSkillDelay is configurable >=0, default3s preserving old enemies; Medusa uses.15s. Task tracks and waits/cancels only the ability activation it initiated, rather than every active GAS ability. FEnemyPhaseSkillSet adds BackgroundAbilities; all four pools are granted once at BeginPlay. Background activation skips AimAtTarget.

MedusaAIController runs BT_Medusa with BB_Medusa. Root decision service acquires living player within ActivationRadius and line of sight, retaining briefly within1.4x radius; no shared activation refactor (FEAT234 remains deferred). Selector: retreat after melee round, Near melee, Mid dash, Far spell, approach, wait. Custom decorators choose actions; attack tasks are the common UseCombatSkill. Movement Task handles approach/retreat using navigation and slow-aware desired speed. A maximum two-hit melee round requests retreat with retry fallback. Spell location is locked on cast; dash sweeps collision along fixed direction. Nightmare runs separately through Background pool on random cooldown.

Dev Panel remains category-based. Each row independently controls trigger geometry and through-wall position labels; switches apply to current/future instances, are world-local and never change gameplay activation. EnemyPositionDebugWidget projects names/distances; dead enemies are excluded.

Death now routes through final EnemyBase.OnDeath and additive OnDeathCleanup plus configurable DeathPresentation; earlier unconditional-ragdoll/immediate-Destroy descriptions are historical.

FEAT235 verified combat tuning: Medusa Near/Mid/Far pools use BGA_Medusa_Melee/Dash/Spell via the existing BTTask_UseCombatSkill (display name Use Combat Skill). Task default recovery3s remains for old trees; Medusa nodes use.15s. Background pool carries independent Nightmare, randomized12-25s after each completion. Ranged Subjugate locks a ground point at cast start, hits at.7s in250cm radius; it is not a homing projectile and has no added warning. Dash sweeps a fixed direction, melee hits at.45s, two swings request retreat to1000cm; failed retreat retries after2s while close combat remains available. Original successful melee impact FX plays only on a valid hit at settlement. Base direct damage12/10 is multiplied once by GetDamageMultiplier at settlement. FEAT234 shared activation lifecycle remains deferred.

FEAT237 Medusa now turns smoothly through CharacterMovement controller-desired yaw (180deg/s) and acceleration-based navigation. It waits within15deg facing before choosing/starting an attack, instead of snapping on activation. Active attacks lock facing; completion/cancellation re-enables smooth facing. Original two-melee retreat uses backward directional animation while keeping focus on the player; AI path completion permits motor deceleration. ABP_Medusa parent is MedusaAnimInstance, editable directional/rotation-rate BlendSpaces under Medusa/Animations/Logic. GAS phase pools, common Task, damage and random nightmare unchanged. Actual25second cycle measured capped yaw and working Spell/Dash/Melee/retreat; see FEAT237 archive.


FEAT238: BTService_MedusaDecision asks world-owned MedusaIntroSubsystem to present the first valid activation before setting Combat.Target/attack decisions. Once per combat world across all Medusa instances, including replacement spawns; normal new game world resets it. Shared activation FEAT234 remains deferred. First acquisition requires800cm and LOS; existing retention1.4x radius/5s is unchanged. Intro pauses world/round timers until return fade clears, then existing common GAS task resumes.


FEAT239 supersedes automatic FEAT238 intro: BP_Medusa.bPlayIntroduction=false while user hand-edits /Game/Enemy/Nightmare/Medusa/Sequences/LS_Medusa_Intro. This6s30fps LevelSequence contains spawnable native preview mesh/cine camera,editable Transform/LevelStart/CameraCuts/Fade tracks. Preview in TestMap; no persistent map actors. Fullframe28mm lens,DOF disabled,motionblur0,no aspect lock/YFOV framing. Draft is not bound to gameplay activation. Native800cm range and all GAS remain. Automatic replacement handoff is a future user step after shot editing.


FEAT240 supersedes FEAT239 draft-only behavior: BP_Medusa.bPlayIntroduction=true and IntroSequence references LS_Medusa_Intro. Existing800cm/LOS decision gate plays it once per combat world. Native MedusaIntroSubsystem uses LevelSequencePlayer and DefaultLevelSequenceInstanceData.TransformOrigin: first authored skeletal Transform inverse * actual Enemy mesh world transform. Handles UE5.7 custom spawn bindings via SkeletalAnimation-track ownership. Sequence keys/lens/animation/duration unchanged; real gameplay actor stays stationary/hidden while authored visual actor plays. World pause freezes combat/countdown; real-time subsystem seeks sequence and refreshes skeletal pose. Paused Fade renderer workaround samples authored active Fade section value/color/audio at the same time.0.3s entry and0.35s return bridge around authored shot. Stop restores state/destroys spawned actors,view,input,visibility,fade and owned pause; death/EndPlay/unpossess/travel cancel. Legacy IntroAnimation property is compatibility-only; edit IntroSequence for playback. No shared activation redesign.


FEAT241: native/BP Medusa ActivationRadius1200cm supersedes800cm. Intro visibility track hides preview frames0..12; main LevelStart12..162 preserved; separate fixed-time last-frame hold162..180 prevents idle reset during fade. Camera/mesh transform keys preserved. Death_front is only.966667s and final tilted pose is unsuitable for corpse hold; current death behavior unchanged pending separate presentation decision.


## FEAT242 Medusa original spatial voices
MedusaCombatComponent exposes SpellSound/DashSound/MeleeSound/DeathSound under Combat|Audio. BP_Medusa assigns owner Audio/SCue_Medusa_{Spell,Dash,Melee,Death}. Accepted GAS BeginAction plays one attached head voice; normal FinishAction preserves voice tail, cancellation/new action/death stops attack voice, EndPlay stops all. Death cleanup plays separate DeathAudio once; shared death presentation/body mode unchanged. Source original Sevarog Effort Ability_E/Ability_RMB/Swing/Death respectively, three native waves each. No synthesized sound, dialogue, added warning, or DarkVeil timing changes.


## FEAT243 Void Gazer wall crawl
VoidGazer first stage is planar static vertical wall locomotion only. AttachToWall(Point,OutwardNormal) validates geometry/clearance; local X is projected world-up, local Z wall normal. SetWallInput(X=right,Y=up) controls85cm/s travel; StopWallCrawl stops. AutoAttach traces actor forward up to300cm at BeginPlay, InitialWallInput default zero. Surface probes plus capsule sweeps stop on unsupported edges/obstacles; no gravity, navigation, combat or chase yet. Existing enemy slow multiplier applies once; death and missing support stop movement. TestMap fixtures stay transient.


## FEAT244 Void Gazer automatic glance

FEAT247 adds the higher-priority FindLeap -> GAS Leap sequence to BT_VoidGazer. Same-wall look/crawl runs between transfers; LookAtActor refuses requests while leap is active. Static wall destination/trajectory validation and lifecycle are documented in15-enemy-movement.md. BP_VoidGazer grants BGA_VoidGazer_WallLeap through existing DefaultAbilities; the controller cancels it on unpossess. This is automatic patrol movement, not chase/attack behavior, and does not implement deferred FEAT234 activation.
Independent native wall crawler optionally checks local player every LookInterval6s after previous glance ends, within LookRange1200cm and visibility trace. Target must be on outward side of wall. Selects Left/Right by wall tangent dot; central abs(dot)<.35 selects Back(ledge Lean_Out). Entry/exit follow clip durations, LookHoldTime1.6s, then resumes queued crawl input. No chase/attacks/navigation added. bAutoLookAtPlayer=true on new VoidGazer; disable for externally driven presentation. Original Medusa/DarkVeil unchanged.


## FEAT246 movement responsibility update
See `15-enemy-movement.md` for current selection/request/execution ownership. Ground queries are shared by Phantom/Medusa; Void mechanics move into WallCrawlMovementComponent with tree scheduling. Existing animation assets, retargeting and CoreMorph flight presentation remain unchanged. Implementation verification is tracked in FEAT246 archive. This supersedes earlier actor-Tick random roaming and Controller-Tick Phantom combat requests.

FEAT246 user correction: FlyingBug2 is retired. Its owner assets/native class/dedicated tests are removed; historical FlyingBug sections above are archival, not current setup instructions. Do not restore or continue FEAT076.


FEAT248 Void combat now uses the same PhaseSkillSets and BTTask_UseCombatSkill as other enemies. Root BTService_VoidTarget writes living player TargetActor; explicit BTDecorator_VoidSkillRange gates Near ground melee by distance/height/cooldown and Mid wall slam by surface/range/cooldown. Near task selects one Near ability (internally randomizes four animation clips); Mid sequence plans safe ground point then runs common skill task. Both configured phases have one Near Melee and one Mid GroundSlam. No landing combo, dedicated melee Task or component/controller automatic attack scheduler. Common Task waits for owned GAS ability and cancels it on abort. Existing look/crawl/wall patrol branches remain lower priority.


FEAT249 historical path: Void crawl selection used BTTask_FindWallLocation -> FEnemyWallPointQuery -> wall capability validation, then VoidWallAction Move. FEAT284 now replaces those saved BT node pairs with BTTask_WallMovementPreset; FindPoint remains a compatibility enum/action. Leap planning/execution and combat priorities are unchanged. See arch/15-enemy-movement.md.


## FEAT257 Void tentacle GAS integration (verified 2026-10-07)
Supersedes the earlier presentation-only scheduling boundary. GA_VoidTentacle is InstancedPerActor/ServerOnly and owns the existing component from windup through retraction; cancellation calls CancelPresentation and releases owned player controls/UI/strands. Cooldown uses GE_VoidTentacleCooldown and native Cooldown.VoidGazer.Tentacle tag, applied after accepted activation ends or is cancelled; editable ability CooldownSeconds defaults8. Component keeps existing4s Hook/450cm Whip/shake0 and single-multiplier damage. Read-only CanStartPresentation validates owner/controller, live player, reach, LOS, action exclusion and captured-victim ownership.
EEnemySkillRange appends AnyDistance after Background without renumbering existing entries. FEnemyPhaseSkillSet.AnyDistanceAbilities is granted with existing class deduplication and selected by UseRandomSkill. Its selected target is exposed through GetSkillTarget; the ability uses this target rather than an implicit local player. AnyDistance bypasses distance-group partition only, not skill reach/LOS/cooldown. BP_VoidGazer configures BGA_VoidGazer_Tentacle under GAS/Abilities in both phases. BT retains close ground melee first, then common UseCombatSkill(AnyDistance), then prior slam/transfer/look/crawl branches; failed activation falls through immediately. The shared task waits for the owned GAS handle and cancels it on abort. Full patrol/combat hierarchy and ground-to-wall re-entry were outside FEAT257; FEAT260 below supersedes this.


### FEAT259 Void action cadence and ground movement
Void BT now follows successful Near melee / AnyDistance grab with a short reposition selector (wall crawl or grounded movement); slam follows with ground movement, wall transfer with reposition. Existing priority/GAS task ownership and explicit look branch remain. During a selected wall crawl, the move task no longer starts extra look actions. Short180cm crawl completes before another attack cycle. UBTTask_FindVoidGroundLocation is a Void policy adapter to FEnemyGroundPointQuery: approach when farther than MeleeRange+120, otherwise bounded160cm strafe around .85*MeleeRange with40cm tolerance. It clears failed goals and rejects wall/air/skill states. UBTTask_VoidGroundMove extends the shared request-owned executor, uses1.2s replanning, faces target at180deg/s, yields when melee becomes valid and aborts if an exclusive action starts. Uses Void SetGroundMoveSpeed -> enemy slow-aware desired-speed API (200approach/120strafe). FEAT260 below supersedes the strafe policy, movement speed and ground-return limitation.


### FEAT260 current Void behavior (2026-10-07)

FEAT260: Void tentacle is in FarAbilities for both phases; BT common UseCombatSkill(Far) and GA_VoidTentacle MinimumDistance800cm reject closer activation, existing component1400cm root-based reach/LOS/cooldown remain. Already-acquired grip continues if target comes closer. Root highest-priority ground-return sequence calls VoidWallAction FindWallReturn then existing GAS leap and reposition. Ground return waits4s after a successful slam, probes valid nearby static walls and uses full supported/collision-checked trajectory. Ground movement policy now only approaches, stops within0.9*MeleeRange (or when melee is available), faces velocity and moves140cm/s; random alternating strafe removed. Shared AnyDistance support remains for other consumers; FEAT257/259 AnyDistance assignment is historical.

Historical FEAT268 ground return used request-owned MoveTo before turning and planning the GAS leap. The 2026-10-15 follow-up in15-enemy-movement.md replaces that walk with one validated ground hop when a direct wall leap cannot be planned. FindWallReturn still has a12s timeout; no return NavMesh request exists.

FEAT268 precise return approach: MoveTo acceptance5cm, excludes agent radius; arrival guard15cm and grounded facing threshold1deg before PlanTo. This prevents loose navigation arrival plus old5deg facing tolerance from repeatedly failing stricter launch admission. Request ownership/cancellation remains unchanged.


## Current shared patrol/alert turn lifecycle (2026-10-12)
Both stances use AHumanoidEnemy.RequestTurn -> pending -> OnTurnComplete. Alert turn is stationary-only/grounded/valid combat target with all8compatible configured assets; controller facing yields to authored yaw. bDeferTurningDuringHitReaction now belongs to humanoid and bypasses the base projectile immediate yaw snap while still notifying normal Hit engagement. Reaction completion permits turning; repeated hits interrupt it. Alert completion never calls MoveToNextPatrolPoint. Patrol retains existing relaxed rotation/movement behavior and can resume a reaction-interrupted pending turn. Test-soldier preset remains noBT/noauto-navigation/initialPatrol/Hit-only retainedAim; do not restore old PatrolWeaponDummy no-engagement assertions.


## FEAT-280 CoreMorph AI/GA boundary (2026-10-12)
CoreMorph BT movement tasks now call ScorpionMovement; BT service reads attack readiness from the GA classes and targets the motor. Attack state is owned solely by the active GA, not a Combat component. Movement yields if an attack owns the Attacking tag. ScorpionRecordingGameMode and ScorpionExecution regression use motor movement state and GA strike count. BP_CoreMorphBoss moves its saved BT to ScorpionMovement. SpaceStation Station_Manta external package F/4W/68VCV3P9SEAQZQQS860JTO migrates its enabled override (original author.py evidence), retaining position, initial Manta and existing climb/dive/reassemble opening; only that actor package resaved.

## FEAT-284 movement preset authoring (2026-10-14)

Historical FEAT-284 migration: BT tasks schedule movement and own only their request token. Ground consumers use `BTTask_GroundMovementPreset` with `RandomPoint`, `KeepPlayerDistance` or `FollowActor`; the ground component validates NavMesh and issues `AIController::MoveTo`. At this stage Void wall reposition branches in both saved Ambient and Combat trees used `BTTask_WallMovementPreset` with `RandomPoint`; later Combat moved to PhaseSkillSets selected GA while Ambient retained its presets. The wall component validates support and clearance and executes crawl; wall leaps and ground return remain separate. CoreMorph Ambient's `BTTask_CoreMorphAction` exposes `AmbientFlightPreset`: saved default `AuthoredRoute` preserves spline/reference choreography, while `RandomPoint` uses a bounded 3D swept route and the existing flight GA/component. Air collision reports failure; no route around obstacles is implied. All three use `EEnemyMovementRequestStatus` and token ownership; BT abort never cancels a newer request. Keep creature-specific combat and encounter policy in the respective BT/GA rather than the shared request state.

Historical 2026-10-15 intermediate routing (superseded by the current section above): saved BT_VoidGazer had one `BTTask_UseCombatSkill` with an earlier Near <=200cm threshold. Both BP_VoidGazer phase sets repeated wall actions in each distance pool. This paragraph records the migration away from fixed-priority Combat branches; do not reuse its old ranges or action list.

## Void current attack and movement selection (2026-10-15)

The saved combat selector now uses Near <=500cm, Mid <=800cm and Far >800cm. Near ground chase and retreat are random entries; wall return stays in all ground ranges. Combat WallRoam uses the Void actor settings 250..450cm with 24 point attempts. The saved Ambient subtree has three independent WallMovementPreset nodes with the same 250..450cm and 24 attempts. Invalid wall support or path candidates fail selection and let the skill task retry after its normal wait.
