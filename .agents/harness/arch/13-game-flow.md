# 游戏流程（回合、死亡重生与大厅）

## FEAT-300 默认地图与死亡重生（2026-10-20）

- 项目运行和编辑器启动地图均为 `/Game/Maps/Lvl_MainGame`，并纳入 `MapsToCook`。
- `UTheManGameInstance::HandlePlayerDeath` 保留选中角色和回合数，重载 `RespawnMapName`（默认 `Lvl_MainGame`）。目标世界到达时 `OnWorldChanged` 解除死亡切图锁，支持再次死亡重生。
- 显式 `ReturnToMainMenu` 和最终 `HandleGameOver` 仍前往 `LobbyMap`；大厅选角进入地图仍由 `TestMapName` 控制。下方 FEAT-037 记录的是旧死亡流程。

FEAT-102（2026-09-17）：暂停菜单 ReturnToMainMenu 经 GameInstance 的独立同名入口采集现有世界持久化快照、恢复暂停状态并 OpenLevel(LobbyMapName)，不调用死亡处理，不改变 CarriedRoundNumber 或 GameOver。复用切关守卫；下次 START GAME 仍由 SelectCharacterAndStart 解除守卫。大厅主菜单现在为 START / SETTINGS（仍禁用）/ EXIT；新增 Button_Exit 仅在主菜单显示，调用标准 UKismetSystemLibrary::QuitGame，PIE 中只结束游戏会话。布局通过 TheManLobbyAssetLibrary.AddExitButton 添加，保留原两行位置与样式。

FEAT-085（2026-09-16）执行官已接入正式大厅展示选择：Character 打开独立角色详情页，文字按钮选择维修工或执行官，只显示名字和介绍；选择与 Back 都保持 Relax／远景。Back 返回主菜单，再点 Weapon 对当前角色进入 Rifle／近景；执行官使用自带狙击枪并显示对应文字，隐藏维修工三枪选择器。角色实例与各自枪索引由 CharacterSelectPlayerController 缓存，所有 UI／姿态均取当前 Actor。主菜单初始仍展示地图原维修工；无需替换地图，执行官在选择时生成。该流程不启动游戏、不改变原战斗 Pawn／GameInstance 选角入口。执行官已有 Relax 和 Ready 成品，Standing 非本批范围。

> 地图现状（2026-09-14）：FEAT-045选角场景已恢复并正式保存为`/Game/Maps/Lobby/LobbyMap`，原同名弃用空地图已删除。326个大厅地图专属依赖已从素材包路径`/Game/Maps/SciFiIndustrialBase`迁至`/Game/Maps/Lobby/{Blueprint,Meshes,Materials,Textures,Audio}`，预计算数据仍为`/Game/Maps/Lobby/LobbyMap_BuiltData`。`/Game/Environment`保留给游戏环境中的交互与场景功能，不接收地图专属依赖。`UTheManGameInstance::LobbyMapName`原本就是`LobbyMap`，因此死亡返回大厅流程直接进入正式选角场景。

## 大厅人物展示资源（FEAT-082）

用户新增大厅摆放用`/Game/Characters/MaintenanceWorker/Lobby/Blueprint/BP_MaintenanceWorker_Lobby`，继承AMaintenanceWorkerLobbyCharacter→ALobbyCharacterBase→AActor。默认DisplayPose=Standing空手站立，StandingIdleIndex=0；UI用SetStandingIdleIndex(0/1)选择两种站立待机，也可用SetDisplayPose选择Standing／Relaxed／Rifle。大厅按钮的标准持枪接口是`SetWeaponReady(false/true)`：false进入Relaxed放松持枪，true进入Rifle举枪，`IsWeaponReady`读取状态。Standing隐藏武器。它只负责人物和展示武器，不创建Pawn／ASC／游戏装备；模型与五条动画在TMIIR完成适配，成品归MaintenanceWorker/Lobby。维修工当前以WeaponPresentations各项RelaxedAnimations为准；全局Relaxed Animation仅为未配置武器列表的通用后备。维修工展示通过WeaponPresentations读取正式BP_RepairGun／BP_ExplosionGun／BP_ElectricGun的显示配置，DisplayWeaponIndex=0／1／2或SetDisplayWeaponIndex选择。默认0维修枪使用正式骨骼枪模和三槽材质，另两枪使用各自静态模型。每把枪分别配置右手挂点和独立修正动画，运行时通过LobbyPoseBlendAnimInstance混合成品，已移除大厅IK。RelaxedIdleIndex／SetRelaxedIdleIndex选择每枪两种放松版本；Standing继续使用原空手动画并隐藏两种枪械组件。不再用Rifle_01替代正式武器，正式武器BP和原动画不变。正式LobbyMap在角色焦点原点放置一个`MaintenanceWorker_LobbyDisplay`实例，Yaw=180朝向远景，初始为Relaxed。两台原有镜头瞄准人物并按实际距离对焦。

大厅测试复用`IA_Test`和键盘上方`1`。`IMC_CharacterSelect`将One映射到IA_Test，`BP_CharacterSelectPlayerController.TestAction`引用该Action。FEAT-118：仅WITH_EDITOR PIE中，`HandleTestInput`将当前展示选角ID传给`GameInstance.EnterTestMap`，不再切换Relaxed／Rifle。游戏内同键根据当前Pawn花名册行保留英雄。EnterTestMap固定打开`/Game/Maps/Test/TestMap`，已在TestMap时忽略；切图前保存世界快照、解除暂停并清切关守卫，不更改正常START GAME的SpaceStation目标或携带回合值。

- Enemy死亡与玩家流程独立：EnemyBase.OnDeath现在启动布娃娃并停止AI/技能，CorpseLifetime默认5游戏秒后销毁，详情见arch07/09/11；不会触发玩家回大厅。Enemy在死亡瞬间解除GameState波次/阶段订阅，尸体不继续升级。

**何时读取：** 修改回合/倒计时/阶段升级逻辑、死亡处理、关卡切换、选角色流程、跨关卡持久数据（GameInstance）时。

> 涉及功能：FEAT-022（回合系统）/ FEAT-036（半场二阶段 + 强度伤害递增）/ FEAT-037（死亡→大厅→选角色→重开）。

---

## 涉及文件

| 文件 | 关键内容 |
|---|---|
| `Public/Core/TheManGameStateBase.h` / `.cpp` | 回合倒计时驱动 + 半场二阶段广播 + 强度波广播；`AdvanceRound()` / `DebugSkipTime()` |
| `Public/Core/TheManGameInstance.h` / `.cpp` | 跨关卡持久：`SelectedCharacterID` + `CarriedRoundNumber`；`SelectCharacterAndStart()` / `HandlePlayerDeath()` |
| `Public/Core/TheManGameModeBase.h` / `.cpp` | 测试地图 GameMode：`GetDefaultPawnClassForController` 据选定 ID 在花名册查类生成角色；`CharacterRosterTable` |
| `Public/Core/TheManLobbyGameMode.h` / `.cpp` | 大厅专用 GameMode：`DefaultPawnClass=nullptr`；BeginPlay 建选角色 UI + 显鼠标 + UI 输入模式 |
| `Public/UI/CharacterSelectWidgetBase.h` / `.cpp` | 选角色 UI 基类：BindWidget 三按钮 + 自动绑点击 → `SelectCharacter(ID)` |
| `Private/Core/TheManPlayerController.cpp` | BeginPlay 末尾重置 `FInputModeGameOnly` + 隐藏鼠标（覆盖大厅残留的 UI 输入模式） |
| `Private/Characters/CharacterBase/FPSCharacterBase.cpp` | `OnDeath()` → `GI->HandlePlayerDeath(RoundNumber)`（不再原地复活） |

---

## 回合系统（GameState，FEAT-022/036）

`ATheManGameStateBase` 每帧 `Tick → AdvanceRound(DeltaSeconds)`：

- **倒计时**：`TimeRemaining` 递减（钳 0 不为负）；归零 → `OnCountdownExpired()`（= 玩家死亡，见下）。
- **强度波（细粒度递增）**：每过 `StrengthIncreaseInterval`（默认 150s=2.5min）广播一次 `OnMidRoundStrengthIncrease` + `ElapsedStrengthWaves++`。敌人订阅后伤害倍率累加（见 `11-enemy-ai.md`）。
- **半场二阶段（粗粒度）**：`TimeRemaining ≤ CurrentRoundDuration × Phase2TriggerRemainingFraction`（默认 0.5）时广播一次 `OnCombatPhaseChanged(2)`（`bPhase2Triggered` 守卫）。敌人据此 `SetCombatPhase(2)` 换技能集——**阶段只换技能集，不碰伤害**。
- `StartNewRound()`：`RoundNumber++`，按 `BaseCountdownDuration - CountdownReductionPerRound×(Round-1)`（钳 `MinCountdownDuration`）算时长，重置波数/阶段标记。
- `BeginPlay`：先从 GameInstance 读 `CarriedRoundNumber` 赋 `RoundNumber`，再 `StartNewRound()`（+1 衔接），实现回合数跨关卡延续。

**默认值定稿**：`BaseCountdownDuration=600`(10min) / `StrengthIncreaseInterval=150`(2.5min) / `Phase2TriggerRemainingFraction=0.5`。

**调试**：`DebugSkipTime(Seconds=150)`（BlueprintCallable，复用 `AdvanceRound`，减过头钳 0 触发回合结束）；Controller `DebugSkipTimeAction`(IA) 绑键调用。屏幕显示剩余时间「分:秒」。

**显示语言**：项目是英语游戏。倒计时格式为 `[Round N | Phase N] Time Remaining: M:SS.s`；所有玩家可见 UI、HUD 和屏幕 Debug 文本必须使用英语。中文只允许用于不会渲染到屏幕的源码注释、harness 文档和开发者日志。

事件（BlueprintNativeEvent，蓝图覆写须调 Parent）：`OnRoundStarted(Round, Duration)` / `OnCountdownExpired()`。

---

## 死亡 → 大厅 → 选角色 → 重开（FEAT-037）

```
死亡两源：
  ① 被打死：Health≤0 → AttributeSet → AFPSCharacterBase::OnDeath()
  ② 时间到：TimeRemaining≤0 → GameState::OnCountdownExpired()
        ↓ 都 funnel 到
UTheManGameInstance::HandlePlayerDeath(当前RoundNumber)
  - bPendingTransition 守卫（防同帧两源重复切换）
  - CarriedRoundNumber = 当前回合
  - UWorldPersistenceSubsystem.CaptureWorldState（FEAT-079；下一次 TestMap 在 Actor BeginPlay 前恢复）
  - OpenLevel(LobbyMapName="LobbyMap")
        ↓
LobbyMap（GameMode=BP_LobbyGameMode : ATheManLobbyGameMode）
  - DefaultPawnClass=nullptr（大厅不生成战斗角色）
  - BeginPlay：CreateWidget(LobbyWidgetClass=WBP_CharacterSelect) + AddToViewport
              + bShowMouseCursor=true + SetInputMode(UIOnly)
        ↓ 玩家点按钮
UCharacterSelectWidgetBase::SelectCharacter(ID)
  → GI->SelectCharacterAndStart(ID)
      - SelectedCharacterID = ID；bPendingTransition=false（解除守卫）
      - OpenLevel(TestMapName="TestMap")
        ↓
TestMap（GameMode=BP_TheManGameMode : ATheManGameModeBase）
  - GetDefaultPawnClassForController：读 GI.SelectedCharacterID → CharacterRosterTable 查行 → 返回 CharacterClass
        → 引擎在 PlayerStart 生成该角色并 Possess（GAS/装备照常 PossessedBy/BeginPlay 初始化）
  - GameState.BeginPlay：RoundNumber=CarriedRoundNumber → StartNewRound()（回合+1，敌人初始强度+1）
  - PlayerController.BeginPlay：AddMappingContext + SetInputMode(GameOnly) + 隐藏鼠标
```

### 跨关卡数据（GameInstance 是唯一能存活过 OpenLevel 的对象）
- `SelectedCharacterID`：选了谁 → 测试地图 GameMode 据此生成角色。
- `CarriedRoundNumber`：死亡写入当前回合 → 测试地图 GameState 读取后 +1。**回合内的波次增强/伤害倍率不携带，每回合重置**（对应"前面增强消失、只留逐回合 +1 的初始增强"）。
- FEAT-079 起，Actor 世界状态由 `UWorldPersistenceSubsystem` 在同一次游戏进程内保存；详见 `14-world-persistence.md`。普通返回大厅不清空，关闭游戏后内存状态自然消失。

### 关键约束 / 坑
- **测试地图 GameMode 必须填 `CharacterRosterTable=DT_CharacterRoster`**，且 World Settings GameMode Override 指向它，否则 `GetDefaultPawnClassForController` 查不到 → 回退默认 Pawn（角色错）。
- **大厅必须用 `ATheManLobbyGameMode`**（非测试地图 GameMode），否则会在菜单里生成战斗角色。
- **输入模式跨关卡持久**（`SetInputMode` 改的是持久的 `UGameViewportClient`）：大厅设 UIOnly，进游戏关卡靠 `ATheManPlayerController::BeginPlay` 重置 GameOnly，否则角色动不了（BUG-037-001）。
- 选角色 UI 不拖进关卡——填在 `BP_LobbyGameMode.LobbyWidgetClass`，由大厅 GameMode BeginPlay 自动创建。
- 按钮↔角色对应 = `WBP_CharacterSelect` 的 `Character1/2/3ID`（默认 Infiltrator/MaintenanceWorker/TheExecutive），**不按 DT 行顺序**（DataTable 底层 TMap 遍历顺序不保证）。

### 临时调试（查清后删）
- `GetDefaultPawnClassForController` 内 `[PawnSelect]` 屏幕/Log 诊断（逐步定位生成失败原因）。

---

## 目录约定
- UI 类（UMG widget）放 `Source/.../UI/`。
- 框架级（GameMode / GameState / GameInstance / PlayerController / PlayerState）放 `Core/`。

## FEAT-082 正式大厅展示UI（2026-09-14）

当前LobbyMap实际GameMode为BP_CharacterSelectGameMode（上文BP_LobbyGameMode描述为旧流程）。该GM创建WBP_LobbyPresentation，左侧纵向CHARACTER／WEAPON只切换展示镜头，不启动游戏或改持枪动画。Controller明确设置远／近相机，背景点击不切换。既有FarCamera展示角色整体、NearCamera展示枪与双手，0.7秒同步镜头过渡；鼠标幅度5／3cm，近景减半。人物放置和独立三枪动画保持。


FEAT-082按钮姿势联动（2026-09-14）：SetWeaponPresentationView现在同步大厅展示Actor和镜头。true=Rifle/近景，false=从当前枪非空RelaxedAnimations均匀随机选一条并进入Relaxed/远景，每次调用重新选，允许重复。UI不再保留选中高亮，仅Hovered高亮。角色姿势与枪挂点现在使用0.5秒混合；相机仍独立使用既有0.7秒平滑。此说明覆盖之前“按钮仅切相机”的行为记录。


2026-09-14：运行时新增ULobbyPoseBlendAnimInstance（继承UAnimSingleNodeInstance），从当前显示的局部骨骼姿势快照向继续播放的目标成品动画混合；ALobbyCharacterBase.PoseBlendDuration默认0.5秒，SmoothStep进度同步静态/骨骼枪挂点。反向切换重新捕获当前姿势，重复相同状态不重启。初次显示及换枪立即应用；编辑器静态预览保留OverrideAnimationData/PlayAnimation。没有IK，既有每枪独立成品不变。


大厅灯光（2026-09-14）：用户参考16:40:13红白侧光人物与16:39:39暗背景大厅截图，要求调暗整个Lobby、顶部大面积白光向下、画面左红右白。checkpoint7baac17保存此前0.5秒混合。现有地图内灯光强度降至原2.5%，PostProcessVolume固定EV100=3、补偿0；新增LobbyLighting文件夹三盏可直接调节的Movable RectLight：Lobby_Top_SoftWhite 250lm/180×140cm，Lobby_Left_Red 750lm/75×190cm，Lobby_Right_White 500lm/85×190cm，衰减半径均520cm、间接照明0.1。侧灯仅Lighting Channel1，地图人物与两个枪显示组件启用Channel0+1；顶灯Channel0，保留暗淡落地光区。左右以实际镜头画面为准（红灯世界+X，白灯-X）。仅修改LobbyMap，不改蓝图、材质、正式地图或C++，所有原Actor变换逐项相等。


2026-09-14武器详情：WBP_LobbyPresentation中的PresentationMenu与WeaponDetailsPanel按近/远视图互斥显示；详情含Text_WeaponName、Text_WeaponDescription、Button_Back。BACK调用原ShowCharacter，恢复远景及每枪随机Relax。ULobbyPresentationWidgetBase缓存展示Actor，NativeTick仅检查视图/枪索引变化，变化才更新UMG；详情打开时换枪同步刷新。文案归BP_MaintenanceWorker_Lobby的Weapon Presentations各项DisplayName/Description（FText，多行介绍），无需改UI代码；空文本实例按WeaponClass回退角色BP默认值。原有深色/Roboto/金色hover风格保持，Back初始/点击不常驻高亮。


2026-09-14最终简洁选择器取代此前详情卡：WeaponDetailsPanel透明布局，WEAPON标题下三张112×112枪械按钮，按钮中实际模型侧视PNG100×50；选中金边和18×18角标。Button_RepairGun/ExplosionGun/ElectricGun选择索引0/1/2并保持Rifle/近景，同步名称/介绍/实际模型/独立动画；重复当前枪不重启。BACK独立放根Canvas左下角，返回远景/随机Relax并保留枪种。无大底板、02/03或左右翻页。FLobbyWeaponPresentation.Thumbnail引用UI/Lobby/Textures/T_Lobby_{RepairGun,ExplosionGun,ElectricGun}，非AI轮廓；三枪来自当前真实资产在外部Blender工程LobbyWeaponIcons统一渲染。文案、Thumbnail仍在维修工大厅BP Weapon Presentations配置。主菜单仅hover高亮，缩略图选中角标是用户确认的独立选择态。


大厅三点光现状（2026-09-14，覆盖旧灯配置说明）：LobbyLighting/Previous_Disabled归档关闭Lobby_Top_SoftWhite、Lobby_Left_Red、Lobby_Right_White、SpotLight3、SpotLight4。关闭方式是LightComponent Visibility=false，旧灯位置/强度/颜色保留。LobbyLighting/ThreePoint新增三盏Movable RectLight：Lobby_Key_SoftWhite白色主光650lm、160×180cm、位置(-155,-185,295)；Lobby_Fill_SoftWhite白色补光220lm、160×200cm、位置(170,-150,175)；Lobby_Rim_Red红色轮廓光220lm、100×200cm、位置(160,130,235)。衰减半径均650cm，间接强度0，仅Lighting Channel1。人物和两枪组件原Channel0+1保持；曝光、环境灯、相机、人物位置保持。调亮度在对应灯Details的Intensity，调覆盖范围在Attenuation Radius，柔和程度在Source Width/Height。宽面光覆盖Relax/Rifle，不存在运行时灯位切换或跟随骨骼。

## FEAT-087（2026-09-16）：大厅入口与游戏启动

- WBP_LobbyPresentation 主菜单改为 START / SETTINGS；SETTINGS 按用户要求保留禁用。
- START 打开角色详情：SELECT YOUR CHARACTER、角色名与右箭头、TAB NEXT CHARACTER、介绍、VIEW WEAPONS。NextCharacter 按 Controller.CharacterPresentations 循环，键盘 TAB 在 Widget NativeOnPreviewKeyDown 消费，长按重复不连跳。
- VIEW WEAPONS 对当前展示 Actor 进入原武器页/举枪近景；武器页 BACK 回当前角色详情/Relax，角色页 BACK 回菜单。Escape 与 BACK 相同。
- 左侧 START GAME 调用既有 GameInstance.SelectCharacterAndStart(Current CharacterID)，目标 /Game/Maps/Test/TestMap；TestMap GameMode 仍通过 DT_CharacterRoster 查对应战斗 BP。按钮有重复启动和 GameOver 守卫。
- UI 由原 Editor 辅助库 RefineCharacterNavigation 显式写入 UMG；运行时只管理状态、文本与动作。不导入生成效果图做 UI。
- 大厅当前只有维修工和执行官展示。潜伏者仅存在战斗花名册，不自动增加未制作的大厅展示。


FEAT-092 (2026-09-16, supersedes the earlier TestMap target): BP_TheManGameInstance.TestMapName = /Game/Maps/SpaceStation/SpaceStation. Existing Lobby START GAME/SelectCharacterAndStart, SelectedCharacterID and roster/GameMode selection remain. New map has Manta/Scorpion; Executive spawns its companion, Worker does not. Packaging explicitly includes LobbyMap,TestMap,SpaceStation.

## FEAT-111 Lobby weapon icons (2026-09-17)
Executive now uses one FLobbyWeaponPresentation entry with BP_ExecutiveSlowRifle and /Game/Characters/TheExecutive/Lobby/Textures/T_TheExecutive_Lobby_ExecutiveSlowRifle. Both original relaxed/ready animations, materials and mesh-space transforms are preserved by compensating the gameplay weapon component transform in the presentation attachments. The direct fallback fields remain historical; edit the presentation entry for current weapon details. Infiltrator icon /Game/Characters/Infiltrator/Lobby/Textures/T_Infiltrator_Lobby_LaserRifle is imported for FEAT109 binding. Both512x256 transparent silver exact-model renders reuse the Worker external Blender pipeline and effective UI texture settings. Widget RefreshPresentation collapses invalid button parent SizeBoxes, restoring them for valid entries; single-gun characters show one selected tile, Worker keeps all three. Existing112x112 tiles/100x50 icon sizing/gold selection style unchanged. Build,15actual Lobby checks/16physical inputs and cold read passed for Executive/Worker; final Infiltrator check follows FEAT109. No Widget asset/map/camera/light edits in FEAT111.

## FEAT-109 Infiltrator Lobby integration (2026-09-17)
Supersedes the prior two-character Lobby limitation. BP_CharacterSelectPlayerController.CharacterPresentations now has MaintenanceWorker,TheExecutive,Infiltrator in that order; existing two entries preserved. New /Game/Characters/Infiltrator/Lobby/Blueprint/BP_Infiltrator_Lobby derives ALobbyCharacterBase and uses existing0.5s ULobbyPoseBlendAnimInstance. Native O7665 mesh/skeleton,4materials15textures and5final clips live under that owner. Two standing clips are unarmed; two relaxed variants and RifleIdle use exact BP_LaserRifle plus FEAT111 T_Lobby_LaserRifle. WeaponAttachSocket Bip01-R-Hand; separate final relaxed/ready mounts compensate actual source weapon component transform and scale1.1. Final attachment/evidence in Task03/attachment-transforms.json. All retargeting/grip bake occurred externally in TMIIR/Blender; target has no source Worker rig or retarget work. Character English text accurately describes E Scan toggle and Q camera-forward max3.5m obstructed Blink. This is Lobby presentation only; Infiltrator gameplay mesh/loadout unchanged. Current playable LaserRifle remains Worker fourth slot from FEAT107. All-three actual PIE28checks and cold compile/reference/protected-package checks passed; physical13checks separate from final reflected-handler run. No map/camera/light edits.


FEAT-135: God Mode is pawn-local, never persisted in GameInstance or WorldPersistenceSubsystem. Only ATheManPlayerController-controlled combat characters can activate. Unpossess/EndPlay remove the local-player God Mode mapping, PlayerState tag and temporary flight state during level travel. Lobby remains on its separate controller/input path; new combat pawns default OFF.


FEAT-136 (2026-09-19): START GAME is now directly under the character selection details alongside VIEW WEAPONS. It uses the existing CharacterPresentations selected ID and SelectCharacterAndStart/SpaceStation flow, with double-start guard; weapon view no longer contains or permits StartGame. Settings can be opened from the lobby main menu and closed with BACK/Escape; presentation scene, actors and cameras unchanged. Runtime verifies all three actual playable pawns via character-page starts and returns.


## FEAT-138 Infiltrator two-weapon lobby

BP_Infiltrator_Lobby.WeaponPresentations has two entries: BP_InfiltratorLaserRifle (LASER RIFLE) and BP_InfiltratorScopedRifle (SCOPED RIFLE). Each references its own two relaxed clips, ready clip and right-hand mount. Existing WBP_LobbyPresentation slots0/1 bind matching512x256 transparent T_Infiltrator_Lobby_LaserRifle / ScopedRifle icons; slot2 is hidden. Model-rendered silver icon style matches existing characters. The shared widget/map/cameras and gameplay initial equipment remain unchanged. Grip correction affects lobby clips and opt-in transition contact only (see arch06); do not rerun earlier initial-pose author.py/animate.py without preserving GripCorrection results.


FEAT-139: fresh standalone/game startup plays native9s USC/title movie while default LobbyMap loads. Does not add a title menu, map or UMG layer; normal PIE and later returns to Lobby do not replay. Title zoom1->1.035 is baked continuously; START/Settings/Tab/Escape flow unchanged. Source and reproduction: SourceArt/Opening and Scripts/Opening/build_movie.py.


## Lobby UIOnly focus fix (2026-09-20)
CharacterSelectPlayerController uses UIOnly; CharacterSelectGameMode focuses the created widget after AddToViewport. LobbyPresentationWidgetBase makes its full-screen root hit-testable at NativeConstruct and handles unhandled background mouse/double-clicks, retaining focus in lobby (SettingsPanel while open). Buttons/sliders handle their own events first. Tab/Escape remain UI events; no viewport Tab fallback. Editor-only NativeOnKeyDown bubbles after child controls and bridges existing TestAction mapped keys through HandleLobbyTestKey to original HandleTestInput; no hardcoded/new shortcut or new IA. Combat TheManPlayerController already restores GameOnly/cursor on BeginPlay.


## Scorpion recording map (FEAT-142, 2026-09-21)
User explicitly authorized a separate empty recording map, superseding TestMap-only policy for this map. `/Game/Maps/ScorpionRecording/ScorpionRecording` uses `AScorpionRecordingGameMode` under `Maps/ScorpionRecording`. DefaultPawnClass/HUDClass are null; engine PlayerController views a fixed elevated CameraActor. Mode spawns one production BP_CoreMorphBoss as Scorpion and an invisible TargetPoint; production BT/motor/tail GAS operates normally. After two strikes the target cycles through four ground locations. No player character, weapon or round system; existing gameplay/lobby/TestMap inputs and routes unchanged. Open the map and Play to repeat the recording. Map is not added to shipping cook maps.


## Lobby scene cleanup (FEAT-143, 2026-09-22)
User-approved fixed-camera scene reduction: LobbyMap has 787 actors versus 1985 before. Both far/near camera actors and switcher/parallax retained; outer structural shell retained for shading. Removed distant out-of-view scene instances, unused cameras/disabled lights/groups and all imported door interaction actors; one retained door appearance is nine native static mesh pieces. All 709 remaining scenery actors have collision disabled, primitive collision/overlap disabled; no scenery actor tick or door timelines in PIE. Seven fans and two reactor rotators retain visual rotation only. BP_Fan and BP_Gravity_Reactor_Rotator (Lobby-only consumers) no longer own AudioComponents; all 13 AmbientSound actors removed. Lobby has zero scene AudioComponents and no direct Audio package dependencies from map/these BPs. Original unreferenced source assets remain in Content; cleanup removes their map loading path, not unrelated asset files. Camera/UI/character/game-travel code unchanged. Evidence/backup: D:/UnrealWork/LobbyCleanup; archive FEAT-143.

## Lobby presentation preparation (FEAT-141, 2026-09-22)
CharacterSelectPlayerController precreates all configured lobby display actors behind the existing loading overlay. It sequentially renders each selectable weapon's relaxed variants and ready pose, requesting mesh/texture residency and component PSO precaching. Readiness uses the engine streaming manager (including current quality LOD bias and optional-mip availability), viewport draw counts and a render fence. Authored initial character, weapon, idle, pose and blend duration are restored before revealing the lobby; subsequent character selection reuses the same actor instances. No map fixtures or asset edits.

GamePresentationSubsystem gates Lobby readiness on ArePresentationsPrepared for return travel and also covers first startup/direct PIE entry, which bypasses RequestTravel. The native opening movie remains unchanged. Lobby render resources stay resident through controller-owned requests renewed every ten seconds; EndPlay releases the requests/references. A 30-second active-focus timeout reports degraded preparation rather than trapping the menu; background time is excluded because the editor may suspend rendering/streaming. bPresentationPreparationFailed and GetPendingPresentationResources provide verification state. Existing world streaming, fade and input restoration remain in use.

Lobby residency budget follow-up: retaining all display textures exceeded the original 800 MB editor / 1000 MB standalone pool. The controller now reserves original pool + measured presentation texture bytes (rounded to 64 MiB) + 128 MiB during lobby use, capped at half reported device working memory (never below the original pool). Automatic/unlimited pools are left as configured. The original positive pool value is restored at EndPlay only if no later user/scalability change has replaced our value. Nothing is saved to config. Runtime RHI dependency supplies device memory statistics. Final standalone: 1021 MiB of presentation textures; pool 1000 ->2152 ->1000 MB, preparation 5.521s; no texture pool overflow in final captures.

## SpaceStation exterior streaming (FEAT-144, 2026-09-23)

`/Game/Maps/SpaceStation/SpaceStation` now uses native World Partition with the spatial hash:256m runtime cells,768m loading range, block on slow streaming. Landscape has256proxies (2x2components each);250proxies and22rock foliage actors stream. Six landscape proxies intersecting X[-10000,10000],Y[-25000,10000]cm stay resident to support persistent station NPCs/navigation and initial boss area. All existing station geometry, doors, enemies, encounters, routes, lighting and navigation remain non-spatial. Do not stream these gameplay actors without adding state capture and ensuring their ground remains loaded: first runtime test exposed falling weapon-test NPCs when their terrain unloaded. Authored actor classes/positions preserved; no retargeting, gameplay tuning or lighting edits.

Existing GamePresentationSubsystem already waits native WorldPartition streaming before revealing travel. PIE: near208/1024terrain components, far184;36nearproxies unload and30newproxies load;49gameplay instances persist; return restores same terrain and floor collision; lobby return/START successful. No general FPS or packaged performance claim.

Conversion-only helper `Maps/SpaceStation/Editor/StationWorldPartitionConvertCommandlet` delegates to the engine commandlet while retaining original embedded textures and fully loading existing foliage destinations. This works around observed UE5.7 package-trailer and partially-loaded-package save failures. Implementation guarded WITH_EDITOR, existing UnrealEd dependency only. Explicit invocation: `-run=StationWorldPartitionConvertCommandlet /Game/Maps/SpaceStation/SpaceStation -AllowCommandletRendering -RunningFromUnrealEd -SCCProvider=None -unattended`. Map-local SpaceStation.ini records conversion parameters; do not rerun conversion on the already-partitioned map. Original backup and evidence D:/UnrealWork/StationOptimization. No final commit/push/package or version increment.

FEAT-145: lobby SETTINGS includes GRAPHICS QUALITY (LOW/MEDIUM/HIGH), draft until APPLY, saved across sessions; closing discards uncommitted selection. Defaults preview HIGH. Existing audio preview and15-second display confirmation retained; pending display changes delay saving quality until KEEP. Quality presets retain100%render scale and compare all11 quality groups explicitly (engine overall level incorrectly calls these custom due to engine render-scale presets). Applying quality refreshes lobby texture reservation and records the new base pool for travel restoration. No combat pause/settings entry added.

### FEAT-141 scene transitions (2026-09-23)
GamePresentationSubsystem separates black background and title/ring opacity. RequestTravel fades current scene to black0.7s then reveals loading content0.6s using smoothstep; waits two fully covered viewport draws before OpenLevel. MoviePlayer uses the same start clock for seamless handoff. Initial lobby preparation begins fully covered, fading content in0.6s. Once native readiness/render fence pass and entrance completes, fade loading content out0.6s over black, then reveal destination0.7s; input restoration follows completion. This supersedes the previous0.25s whole-overlay fade and abrupt loading entrance. Original startup movie remains unchanged. Durations central in private subsystem cpp namespace; applies all existing RequestTravel routes.

2026-09-24 supersedes the paragraph above for Lobby: Opening.mp4 is now the original4s USC section only. GamePresentationSubsystem owns the second The Man I Invented logo card (same original title texture, gentle3.5% zoom, minimum4s and readiness hold) while lobby resources/rendering prepare. Title fades out then lobby fades in without any Loading indicator. Initial PIE uses a plain cover rather than replaying startup logos. Returning to LobbyMap fades to black0.7s, waits readiness, then reveals0.7s; no title/spinner/Loading text. Combat destinations retain0.7+0.6s entry and0.6+0.7s exit. Countdown now uses independent Slate text Time Remaining: M:SS.s; old Round/Phase diagnostic prefix and all other screen-log messages/FPS are hidden. Countdown gameplay and developer file logging unchanged. IsPresentationBlocking is the readiness/input guard; IsLoadingScreenVisible is true only when combat Loading artwork is active.

2026-09-24 user correction: retain FPS, Round and countdown. Earlier hiding of FPS/Round was an interpretation error, now corrected. Independent Slate shows FPS in lobby and combat; combat adds Round N | Time Remaining: M:SS.s. Other engine/BP screen diagnostic logs remain hidden, normal HUD and Dev Panel retained. build-hud.log Succeeded; focused actual PIE screenshot PIE-HUD00000.png shows FPS/Round/countdown with SCREEN_LOG_MUST_BE_HIDDEN probe absent; hud.json confirms round1 and decreasing time. Loading changes retained and prior standalone/PIE travel results remain valid. Final editor on TestMap, PIE ended. No packaging/version increment/final commit.


## FEAT-152 Dev Panel test actions (2026-09-24)
User supersedes the old keyboard0/1 test routes: time skip and test-map travel now exclusively use WBP_DebugMenu buttons Button_SkipTime / Button_TestMap (TESTING section). Tab still opens/closes combat Dev Panel. Labels: Skip Time +150s / Enter Test Map. UDebugMenuWidgetBase routes to ATheManPlayerController.SkipDebugTime / EnterDebugTestMap; both close the panel/resume before invoking existing round/travel logic. Test travel retains current pawn roster ID; already-TestMap is a no-op after closing UI. UE_BUILD_SHIPPING compiles out both implementations and panel access, consistent with FEAT151.
No runtime binding for IA_SkipTime or IA_Test, no Lobby UI key bridge or Lobby test callback. Removed their mappings from IMC_Default and IMC_CharacterSelect; keep serialized IA references/assets for compatibility/reserved future authoring. Normal lobby Tab selection and gameplay inputs unchanged. Editor helper AddTestActionButtons appends editable UMG controls without rebuilding existing layout.


FEAT238: MedusaIntroSubsystem owns first-activation presentation per world. Fixed camera retains player position/FOV, changes aim only under full black.0.4s fade to black, reveal original5s intro, fade black at its end, restore player view under black,0.4s reveal then resume combat. No dolly/zoom/orbit. Preserve viewport widget visibility, player/attached actor visibility and input ignore stack; finish/death/EndPlay/unpossess/view replacement clears fade, view and owned pause. Loading/preexisting pause/fade defers start. Native countdown/FPS Slate remains present. No global activation-framework refactor.


FEAT239 supersedes automatic FEAT238 intro: BP_Medusa.bPlayIntroduction=false while user hand-edits /Game/Enemy/Nightmare/Medusa/Sequences/LS_Medusa_Intro. This6s30fps LevelSequence contains spawnable native preview mesh/cine camera,editable Transform/LevelStart/CameraCuts/Fade tracks. Preview in TestMap; no persistent map actors. Fullframe28mm lens,DOF disabled,motionblur0,no aspect lock/YFOV framing. Draft is not bound to gameplay activation. Native800cm range and all GAS remain. Automatic replacement handoff is a future user step after shot editing.


FEAT240 supersedes FEAT239 draft-only behavior: BP_Medusa.bPlayIntroduction=true and IntroSequence references LS_Medusa_Intro. Existing800cm/LOS decision gate plays it once per combat world. Native MedusaIntroSubsystem uses LevelSequencePlayer and DefaultLevelSequenceInstanceData.TransformOrigin: first authored skeletal Transform inverse * actual Enemy mesh world transform. Handles UE5.7 custom spawn bindings via SkeletalAnimation-track ownership. Sequence keys/lens/animation/duration unchanged; real gameplay actor stays stationary/hidden while authored visual actor plays. World pause freezes combat/countdown; real-time subsystem seeks sequence and refreshes skeletal pose. Paused Fade renderer workaround samples authored active Fade section value/color/audio at the same time.0.3s entry and0.35s return bridge around authored shot. Stop restores state/destroys spawned actors,view,input,visibility,fade and owned pause; death/EndPlay/unpossess/travel cancel. Legacy IntroAnimation property is compatibility-only; edit IntroSequence for playback. No shared activation redesign.


### FEAT-256: Dev map travel (2026-10-07)
GameInstance.EnterDebugMap accepts only the five Dev Panel map package paths (TestMap, SpaceStation, LobbyMap, ScorpionRecording, Lvl_MainGame), validates availability, ignores same-map/loading requests, preserves character choice and captures persistence, unpauses then uses GamePresentationSubsystem.RequestTravel. Shipping compile excludes this debug travel. Existing EnterTestMap delegates to this route. Recording map readiness is explicitly keyed to AScorpionRecordingGameMode and requires an ACameraActor view target instead of a possessed pawn. Lobby preparation remains unchanged. Dev Panel remains the existing combat Tab panel; Lobby and camera-only Recording retain their own controller/input behavior.

2026-10-14 Dev Panel update: `WBP_DebugMenu` no longer contains the INFILTRATOR Blink variant selection section. `UDebugMenuWidgetBase` no longer binds those six buttons. Other Dev Panel controls and Tab open/close remain.

## FEAT270 startup loading feedback and diagnostics (2026-10-08)

Initial standalone title now reuses the existing gold rotating arcs and LOADING... while preserving its logo zoom, minimum4s and readiness/fade gates. USC movie remains4s. Direct PIE and return-to-lobby retain plain preparation covers; combat loading unchanged. CharacterSelectPlayerController emits bounded5s diagnostics after slow preparation begins, identifying each pending render asset (path, active IO/init, resident/requested/max LODs, bias) or pending PSO component. This adds observability, not a timeout fix. Historical v0.8 job0/pending2 30s stalls have not reproduced in current isolated tests; do not claim them resolved. See FEAT270 archive.

2026-10-11 user correction: explicit RequestTravel(LobbyMap), including Pause ReturnToMainMenu, now shows the same loading title/spinner/details as other map travel. Supersedes prior plain black return fade; startup BeginLobbyPreparation remains separate and unchanged.


FEAT274 LobbyPresentation adds optional Button/Image/Corner_DisintegrationGun and SelectDisintegrationGun index3; Worker has4presentation entries, other character slots still collapse by roster length. RefineWeaponDetails adds the fourth designer-authored tile and registers widget variable GUIDs before compilation. Thumbnail is Weapons/DisintegrationGun/UI/Textures/T_DisintegrationGun_Lobby; source image generated from actual weapon reference in existing neutral gray/white transparent style.
