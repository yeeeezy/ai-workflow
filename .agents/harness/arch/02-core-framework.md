# 核心框架（Core）

## FEAT-300 地图入口与重生（2026-10-20）

`UTheManGameInstance` 的死亡切图目的地是可配置的 `RespawnMapName`，默认 `/Game/Maps/Lvl_MainGame`。每次到达该世界时解除死亡切图锁，以允许后续重生。启动地图与编辑器启动地图也设为 `Lvl_MainGame`；显式返回大厅及游戏结束仍使用 `LobbyMapName`。

FEAT-102（2026-09-17）：战斗 Controller 的 PauseAction 引用 `/Game/Core/Input/Actions/IA_Pause`，在 IMC_Default 映射 Escape。TogglePauseMenu 创建 `/Game/UI/Pause/WBP_PauseMenu`，暂停世界、隐藏战斗 HUD、设置 UIOnly 和鼠标；ResumeGame 解除暂停、移除菜单、清按键并恢复 GameOnly/HUD。UMG 基类 UPauseMenuWidgetBase 在 UI 焦点路径处理 Escape 恢复，按钮也调用同一恢复入口。OnUnPossess/EndPlay 清理菜单。UI 布局在可编辑 WidgetBlueprint，不运行时拼装；没有左下角 ESC 提示。UE5.7 的 IMC 正式映射在 DefaultKeyMappings.Mappings，旧 Mappings 字段已弃用。

FEAT-085（2026-09-16）大厅展示选角：CharacterSelectPlayerController.CharacterPresentations 在 BP 中配置维修工和执行官的 ID、英文名字／介绍、展示类。GetDisplayCharacter 返回当前可见展示对象，SelectPresentationCharacter 缓存各角色实例、保留各自枪索引、切换可见性并进入 Relax／远景。UI、Weapon 和预留1键都使用当前对象，不再遍历取首个。该入口只选择大厅展示，不调用旧 SelectCharacterAndStart 或切换战斗 Pawn。

**何时读取：** 修改输入处理、角色切换流程、GameMode / PlayerState 初始化时。

> 回合系统 / 死亡 / 大厅选角色 / 关卡切换 / GameInstance 跨关卡持久 → 见 `13-game-flow.md`。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Core/TheManGameModeBase.h` | 测试地图 GameMode：`CharacterRosterTable` + override `GetDefaultPawnClassForController`（据 GameInstance 选定 ID 生成角色，详见 13） |
| `Source/TheManTest/Private/Core/TheManGameModeBase.cpp` | 含 `[PawnSelect]` 临时诊断（查清后删） |
| `Source/TheManTest/Public/Core/TheManLobbyGameMode.h` / `.cpp` | 大厅 GameMode：`DefaultPawnClass=nullptr`；BeginPlay 建选角色 UI + UI 输入模式（详见 13） |
| `Source/TheManTest/Public/Core/CharacterSelectGameMode.h` / `.cpp` | 新选角场景专用 GameMode：不生成 Pawn；默认使用 `ACharacterSelectPlayerController`；可选创建新选角 UI；不依赖旧 LobbyMap / 旧 WBP |
| `Source/TheManTest/Public/Core/CharacterSelectPlayerController.h` / `.cpp` | 新选角场景专用 Controller：GameAndUI 输入模式 + Enhanced Input IMC；UI通过SetWeaponPresentationView明确切角色／武器镜头，IsWeaponPresentationView读取状态；不再绑定背景点击切镜头。原SetPointerOverUI接口保留兼容 |
| `Source/TheManTest/Public/Core/CharacterSelectCameraSwitcher.h` / `.cpp` | 新选角场景摄像机控制：引用远/近 Cine Camera 目标点；运行时自动生成内部 `ACineCameraActor` Rig 作为 ViewTarget；Rig以BlendTime（0.7秒）smoothstep同步位置、四元数旋转、焦距、光圈、手动对焦距离；同目标忽略，反向切换从当前画面开始。鼠标视差5／3cm、近景乘0.5，切换期间暂停视差；无大推进／弹簧速度累加 |
| `Source/TheManTest/Public/Core/TheManGameInstance.h` / `.cpp` | 跨关卡持久容器：`SelectedCharacterID` / `CarriedRoundNumber`；`SelectCharacterAndStart` / `HandlePlayerDeath`（详见 13） |
| `Source/TheManTest/Public/Core/TheManPlayerController.h` | 增强输入绑定、`SwitchCharacter(FName)`、`DT_CharacterRoster` 指针；`PrimaryFireAction` / `SecondaryFireAction` / `ReloadAction`；`DebugSkipTimeAction`(调试快进)；本地 `CombatHUDWidget` 生命周期与 Equipment/Firearm 委托绑定 |
| `Source/TheManTest/Private/Core/TheManPlayerController.cpp` | 输入回调；BeginPlay 加 IMC + **重置 GameOnly 输入模式**（覆盖大厅 UIOnly 残留，详见 13 BUG-037-001）；`HandleDebugSkipTime`；本地创建 `UCombatHUDWidgetBase`，在 Possess/UnPossess/切枪时解绑重绑装备、弹药和 PlayerState ASC 血量委托，不做 UI Tick |
| `Source/TheManTest/Public/Core/TheManPlayerState.h` | ASC 和 AttributeSet 的声明（GAS 所有者） |
| `Source/TheManTest/Private/Core/TheManPlayerState.cpp` | ASC / AttributeSet 构造 |
| `Source/TheManTest/Public/Core/TheManCharacterTypes.h` | `FCharacterType` 结构体（DataTable 行类型，含角色类引用、图标、描述） |
| `Source/TheManTest/Public/UI/CharacterSelectWidgetBase.h` / `.cpp` | 选角色 UI 基类（BindWidget 三按钮自动绑点击，详见 13） |
| `Source/TheManTest/Public/UI/Combat/CombatHUDWidgetBase.h` / `.cpp` | 原生战斗 HUD：视口中心半径46.08px、线宽2px的80段空心圆；底部显示当前血量、大号当前子弹数和小号备用弹夹数，不显示容量；Slate Paint 绘制、Hit Test Invisible、事件驱动更新 |
| `Source/TheManTest/Public/Core/TheManGameStateBase.h` / `.cpp` | 回合倒计时 + 半场二阶段 + 强度波 + DebugSkipTime（详见 13）；`bIsPowered` 保存当前地图总电闸状态，蓝图可用 `IsPowered` / `SetPowerOn` |

## 测试地图



- 2026-09-13 用户清理：额外7张测试地图（CoreMorph×5、GASPTest、VFXTestMap）及两份地图专属材质已删除。只保留TestMap作为测试地图；LobbyMap正式流程保持。TestMap删除58个后加测试Actor，基础环境与PlayerStart保持。旧VFX房自动化改用TestMap，临时对象不保存回关卡。

## 统一预留测试入口

IMC_Default 的 IA_Test = One（键盘1）。BP_TheManPlayerController 的 TestSwitchCharacterAction 保留旧字段名／IA引用，Started 调用 HandleTestInput。FEAT-118：入口查当前 Pawn 对应 DT_CharacterRoster 行并调用 GameInstance.EnterTestMap，进入 /Game/Maps/Test/TestMap；已在 TestMap 时忽略。旧 Manta 临时生成、弱引用及清理入口移除。大厅 Controller 的同键入口在 WITH_EDITOR 下传当前展示选角 ID；正常 START GAME 仍使用 SpaceStation。没有新增按键或地图。

## FEAT-082 展示导航

正式LobbyMap使用BP_CharacterSelectGameMode，CharacterSelectWidgetClass引用/Game/UI/Lobby/WBP_LobbyPresentation（原为空）。父类ULobbyPresentationWidgetBase绑定Button_Character和Button_Weapon、维护选中描边，UMG资产持有布局／样式／英文文字；它不调用SelectCharacterAndStart。原WBP_CharacterSelect和回合选角流程独立。Editor辅助TheManLobbyAssetLibrary仅初始化空WidgetBlueprint，不覆盖已有布局。

- FEAT-082相机一致性：Switcher每帧读取绑定CameraComponent世界变换（含组件相对偏移），同步取景约束／过扫描／后处理；场景相机是最终取景来源，不能再用一次性Actor位置缓存。鼠标居中、切换结束、相同视口尺寸时与场景相机视图一致。


FEAT-082按钮姿势联动（2026-09-14）：SetWeaponPresentationView现在同步大厅展示Actor和镜头。true=Rifle/近景，false=从当前枪非空RelaxedAnimations均匀随机选一条并进入Relaxed/远景，每次调用重新选，允许重复。UI不再保留选中高亮，仅Hovered高亮。角色仍SingleNode硬切，没有动画混合；既有0.7秒平滑仅用于相机。此说明覆盖之前“按钮仅切相机”的行为记录。

FEAT-087（2026-09-16）：ULobbyPresentationWidgetBase 将主菜单 START、角色箭头/TAB、VIEW WEAPONS 与武器页 START GAME 串联。GoBack 从武器回角色，从角色回主菜单。StartGame 复用 TheManGameInstance.SelectCharacterAndStart 并传当前 CharacterPresentations.CharacterID。旧 Button_Character/Button_Weapon 绑定名保留以兼容资产；前者显示 START，后者显示 SETTINGS 且禁用。

FEAT-088（2026-09-16）：CharacterSelectCameraSwitcher 的运行时相机独立计算目标对焦平面，场景相机继续提供焦段、光圈、构图与后处理。远景取当前 LobbyCharacter.DisplayMesh Bounds 上半身（中心Z+半高×0.35）；近景优先可见且有资源的 DisplayWeapon，其次 DisplaySkeletalWeapon，取枪体 Bounds 中心；无有效枪则回退身体。焦距为目标点沿相机前向的轴向距离，跟随当前展示角色、枪型与鼠标视差；强制 Manual/FocusOffset0，沿已有 TransitionAlpha 混合旧焦距到目标。无需保存关卡相机的旧手动距离；不改变原焦段/光圈/屏幕比例。


FEAT-092 (2026-09-16): BP_TheManGameInstance.TestMapName now targets /Game/Maps/SpaceStation/SpaceStation. The new map uses BP_TheManGamemodeBase and the existing DT_CharacterRoster/SelectedCharacterID player spawn path. TestMap remains the test map.


FEAT-102 背景纠正（2026-09-17）：用户指出全屏模糊不符合参考图。正式 WBP_PauseMenu 已移除活动 BackgroundBlur，PausedWorldShade 使用 `/Game/UI/Pause/Materials/M_PauseBackdrop`，黑色透明度从左 .92 平滑过渡到右 .18（U=.06..74），保留场景清晰度。运行时不改相机或 PostProcess。InitializePauseMenu 只是最初空资产创建助手，不能据它的历史初始样式覆盖当前正式 WBP。


## 2026-09-17 Infiltrator Blink

FEAT-108: Controller adds BlinkAction/GetBlinkAction registry only. IA_InfiltratorBlink under Characters/Infiltrator/Input maps Q in IMC_Default canonical DefaultKeyMappings.Mappings. AFPSInfiltrator binds Started in its own SetupPlayerInputComponent; Controller does not call character functions.


## FEAT-133 Debug menu (2026-09-19)

ATheManPlayerController owns DebugMenuAction, DebugMenuWidgetClass and transient DebugMenuWidget, with ToggleDebugMenu/CloseDebugMenu lifecycle. IA_DebugMenu in Core/Input/Actions maps Tab once in IMC_Default; BP_TheManPlayerController references /Game/UI/Debug/WBP_DebugMenu. Local combat open cancels ADS, pauses, collapses combat HUD, shows cursor and focuses UIOnly. Close restores GameOnly, HUD/cursor/pause and flushes keys. Existing pause and debug are mutually exclusive; unpossess/end play clean up.

Source/TheManTest/{Public,Private}/UI/Debug/DebugMenuWidgetBase supplies Tab-only preview-key handling with repeat protection; Escape is not a debug close key. UI layout/style/text live in editable WBP_DebugMenu. UI/Debug/Editor/DebugMenuAssetLibrary is an editor-only initial authoring helper and must not overwrite the completed layout. Reference: DreamCatchers debug panel, dark background and upper-left DEV DEBUG / Tab to close / PLAYER / disabled God Mode: OFF. God Mode gameplay is not yet implemented. Build, cold assets, 19 PIE checks and actual Tab/Escape UI verified; see archive/FEAT-133-debug-menu.md.


FEAT-134: WBP_DebugMenu includes editable VersionLabel below controls. UDebugMenuWidgetBase.NativeConstruct reads GeneralProjectSettings.ProjectVersion from GGameIni each time it opens, displaying VERSION YYYY.MM.DD.NN. Packaging owns increments via Scripts/Build/package.py; input/God Mode behavior unchanged.


## FEAT-135 God Mode input and UI

ATheManPlayerController registers GodModeMappingContext, GodModeMoveAction and GodModeLookAction; the character component owns binding and activation. New /Game/Core/Input/Actions/IA_DebugFlyMove (Axis3D), IA_DebugFlyLook and IA_DebugFlyBlock; /Game/Core/Input/Contexts/IMC_GodMode maps WASDQE/RMB/Space/Shift, priority1000 only while enabled. IMC_Default and all Lobby inputs are unchanged.

WBP_DebugMenu now has active Button_GodMode, GodModeLabel, FlySpeedInput (SpinBox), FlySpeedSlider and GodModeHint; C++ binds events and reads the owning pawn's component. UI remains editable. Refresh and construction guard against callbacks resetting retained speed. Tab closes the UI without disabling God Mode; Escape behavior remains FEAT133. Version display stays FEAT134 and is not incremented by this non-packaging task.


## FEAT-136 Lobby Settings

ULobbyPresentationWidgetBase now enables legacy Button_Weapon as SETTINGS and shows its authored SettingsPanel child (/Game/UI/Settings/WBP_Settings, USettingsWidgetBase). OnClosed returns to main menu. START GAME moved from WeaponDetailsPanel to CharacterDetailsPanel; native StartGame guard now requires character view. Tab still cycles characters only on that page; settings Escape reverts a pending display change or returns to main menu. Combat pause/debug inputs unchanged. This supersedes historical reserved/disabled SETTINGS and weapon-page START descriptions.

Core/Settings/UPlayerSettingsSubsystem is a GameInstanceSubsystem that stores MasterVolume/EffectsVolume/MusicVolume in GameUserSettings.ini, applies SMix_UserVolumes on world actor initialization, and persists independently of save-game progress/God Mode. Settings widget owns draft display controls and audio preview; Back/destruction discards unapplied audio, Apply saves, defaults preview unity gains and borderless desktop. Display uses UGameUserSettings, 15 real seconds to confirm or revert; embedded PIE only previews/saves audio and must not save its temporary viewport as display preferences. ApplicationCore queries the current window monitor for borderless sizing; High DPI enabled for real pixel dimensions. All UI hierarchy/styles remain in editable assets. Editor SettingsAssetLibrary initializes only an empty WBP_Settings and installs into known lobby without rebuilding its existing layout.


## FEAT-137 Lobby footer and display modes

WBP_LobbyPresentation owns editable VersionLabel directly under PresentationCanvas, bottom-right (-48,-36), right aligned, ZOrder20 above SettingsPanel10. NativeOnInitialized reads raw v0.N from GGameIni; all four lobby views retain it. WBP_Settings no longer owns VersionText. Debug uses the same version source; this supersedes dated FEAT-134 display text.

DisplayModeInput contains only BORDERLESS and WINDOWED. Explicit mapping: index0 -> WindowedFullscreen; index1 -> Windowed. Legacy exclusive Fullscreen drafts as Borderless and converts when applied. Defaults select index0. Existing resolution/15-second confirmation/audio and Escape behavior retained. C++ owns behavior; layout remains editable UMG.

## FEAT-140 sustained focus time ownership
UBulletTimeSubsystem now also supports BeginFocus(Owner,Scale)/EndFocus(Owner). Placement uses this shared clock owner: beginning focus cancels/restores a transient impact envelope before taking the baseline; impact requests are rejected during focus. Weak-owner cleanup ticks even while paused. Ending focus restores the previous dilation only if it still matches the subsystem-applied value, preserving later external changes. WorldEndPlay/Deinitialize clear both modes. The existing impact envelope behavior is unchanged. Executive deployment consumes existing held E/left-click actions at the character/ability layer; Executive binds E Completed/Canceled in its own input setup. No Controller role casts or new mappings. Camera interpolation uses real time separately from focus ownership.


## Lobby UIOnly focus fix (2026-09-20)
CharacterSelectPlayerController uses UIOnly; CharacterSelectGameMode focuses the created widget after AddToViewport. LobbyPresentationWidgetBase makes its full-screen root hit-testable at NativeConstruct and handles unhandled background mouse/double-clicks, retaining focus in lobby (SettingsPanel while open). Buttons/sliders handle their own events first. Tab/Escape remain UI events; no viewport Tab fallback. Editor-only NativeOnKeyDown bubbles after child controls and bridges existing TestAction mapped keys through HandleLobbyTestKey to original HandleTestInput; no hardcoded/new shortcut or new IA. Combat TheManPlayerController already restores GameOnly/cursor on BeginPlay.


## Scorpion recording map (FEAT-142, 2026-09-21)
User explicitly authorized a separate empty recording map, superseding TestMap-only policy for this map. `/Game/Maps/ScorpionRecording/ScorpionRecording` uses `AScorpionRecordingGameMode` under `Maps/ScorpionRecording`. DefaultPawnClass/HUDClass are null; engine PlayerController views a fixed elevated CameraActor. Mode spawns one production BP_CoreMorphBoss as Scorpion and an invisible TargetPoint; production BT/motor/tail GAS operates normally. After two strikes the target cycles through four ground locations. No player character, weapon or round system; existing gameplay/lobby/TestMap inputs and routes unchanged. Open the map and Play to repeat the recording. Map is not added to shipping cook maps.

## Lobby presentation preparation (FEAT-141, 2026-09-22)
CharacterSelectPlayerController precreates all configured lobby display actors behind the existing loading overlay. It sequentially renders each selectable weapon's relaxed variants and ready pose, requesting mesh/texture residency and component PSO precaching. Readiness uses the engine streaming manager (including current quality LOD bias and optional-mip availability), viewport draw counts and a render fence. Authored initial character, weapon, idle, pose and blend duration are restored before revealing the lobby; subsequent character selection reuses the same actor instances. No map fixtures or asset edits.

GamePresentationSubsystem gates Lobby readiness on ArePresentationsPrepared for return travel and also covers first startup/direct PIE entry, which bypasses RequestTravel. The native opening movie remains unchanged. Lobby render resources stay resident through controller-owned requests renewed every ten seconds; EndPlay releases the requests/references. A 30-second active-focus timeout reports degraded preparation rather than trapping the menu; background time is excluded because the editor may suspend rendering/streaming. bPresentationPreparationFailed and GetPendingPresentationResources provide verification state. Existing world streaming, fade and input restoration remain in use.

Lobby residency budget follow-up: retaining all display textures exceeded the original 800 MB editor / 1000 MB standalone pool. The controller now reserves original pool + measured presentation texture bytes (rounded to 64 MiB) + 128 MiB during lobby use, capped at half reported device working memory (never below the original pool). Automatic/unlimited pools are left as configured. The original positive pool value is restored at EndPlay only if no later user/scalability change has replaced our value. Nothing is saved to config. Runtime RHI dependency supplies device memory statistics. Final standalone: 1021 MiB of presentation textures; pool 1000 ->2152 ->1000 MB, preparation 5.521s; no texture pool overflow in final captures.

FEAT-145 supersedes audio-only embedded PIE settings description: USettingsWidgetBase also owns draft LOW/MEDIUM/HIGH. Apply commits11 engine scalability groups at2/1/0 plus100%render scale; standalone SaveSettings persists, PIE saves only scalability state without viewport size. Close discards drafts; defaults select HIGH without committing. Custom/legacy quality preserved until an explicit preset is applied. SettingsAssetLibrary.InstallGraphicsQuality upgrades the existing WBP_Settings layout idempotently; initialization also adds the row. Lobby RefreshPresentationTextureBudget rebases its temporary reservation after scalability changes so EndPlay restores the selected quality budget.

- FEAT-147: WBP_DebugMenu CHARACTER section exposes three editable buttons (Button_MaintenanceWorker / Button_Infiltrator / Button_TheExecutive). UDebugMenuWidgetBase native bindings close Debug Menu then reuse ATheManPlayerController::SwitchCharacter with matching roster IDs; switching creates the role with default loadout at current location. DebugMenuAssetLibrary::AddCharacterSwitchButtons is explicit editor authoring only, not runtime UI construction.

FEAT-141 2026-09-24 supersedes FPS/initial Loading behavior: GamePresentationSubsystem displays only a persistent combat countdown (TimeRemaining) in the former FPS corner. Engine screen diagnostic messages are suppressed while presentation game instances exist and restored on last shutdown; developer log files and the Dev Panel are retained. IsLoadingScreenVisible reports combat Loading artwork only; IsPresentationBlocking covers startup preparation and lobby fades as well. Opening.mp4 now contains USC only; the subsystem holds the second game-title card while existing lobby preparation runs, then reveals the menu. Return-to-lobby uses plain black fade, no title/spinner/LOADING text. Combat travel keeps the established Loading presentation.

2026-09-24 user correction: retain FPS, Round and countdown. Earlier hiding of FPS/Round was an interpretation error, now corrected. Independent Slate shows FPS in lobby and combat; combat adds Round N | Time Remaining: M:SS.s. Other engine/BP screen diagnostic logs remain hidden, normal HUD and Dev Panel retained. build-hud.log Succeeded; focused actual PIE screenshot PIE-HUD00000.png shows FPS/Round/countdown with SCREEN_LOG_MUST_BE_HIDDEN probe absent; hud.json confirms round1 and decreasing time. Loading changes retained and prior standalone/PIE travel results remain valid. Final editor on TestMap, PIE ended. No packaging/version increment/final commit.

FEAT151: Shipping builds disable Dev Panel, test-map travel, debug time skip, debug character switching and God Mode at input and implementation boundaries (UE_BUILD_SHIPPING). FPS display/sampling excluded; version/round/countdown retained. DebugGame/Development behavior preserved. Top-row0 maps IA_SkipTime for150s fast-forward in non-Shipping; lobby normal Tab selection unaffected. Serialized asset references retained for BP compatibility. Both Editor and Shipping compilation and focused editor tool/HUD check passed; no new package. Details archive/FEAT-151-shipping-developer-restrictions.md.


## FEAT-152 Dev Panel test actions (2026-09-24)

FEAT-299: TEST PLACEMENT has separate `Training Enemy A` and `Training Enemy B` buttons. `WBP_DebugMenu` stores `TrainingAPlacementPreset` and `TrainingBPlacementPreset`, referencing owner assets `/Game/Enemy/Nightmare/TrainingEnemy/Data/DA_TrainingEnemy_{A,B}_Placement`. Both use the existing TestMap-only `UTestPlacementSubsystem`; Undo Last/Clear Placed remove their temporary pawn and controller. A and B have independent role BPs and AI assets under their shared training owner. No TestMap actor is saved.
User supersedes the old keyboard0/1 test routes: time skip and test-map travel now exclusively use WBP_DebugMenu buttons Button_SkipTime / Button_TestMap (TESTING section). Tab still opens/closes combat Dev Panel. Labels: Skip Time +150s / Enter Test Map. UDebugMenuWidgetBase routes to ATheManPlayerController.SkipDebugTime / EnterDebugTestMap; both close the panel/resume before invoking existing round/travel logic. Test travel retains current pawn roster ID; already-TestMap is a no-op after closing UI. UE_BUILD_SHIPPING compiles out both implementations and panel access, consistent with FEAT151.
No runtime binding for IA_SkipTime or IA_Test, no Lobby UI key bridge or Lobby test callback. Removed their mappings from IMC_Default and IMC_CharacterSelect; keep serialized IA references/assets for compatibility/reserved future authoring. Normal lobby Tab selection and gameplay inputs unchanged. Editor helper AddTestActionButtons appends editable UMG controls without rebuilding existing layout.


## FEAT215 Dev Panel infinite ammo (2026-09-28)
WBP_DebugMenu has editable Button_InfiniteAmmo/InfiniteAmmoLabel below the GodMode row, created idempotently by existing DebugMenuAssetLibrary.AddInfiniteAmmoButton. Label Infinite Ammo: ON/OFF refreshes on construct and click. The owning ATheManPlayerController stores transient bDebugInfiniteAmmo, defaultfalse; public IsDebugInfiniteAmmoEnabled/SetDebugInfiniteAmmoEnabled provide the UI/weapon contract. State follows the controller across equipment/character changes, resets with a new controller/session; no save or cross-travel persistence is added. Shipping setter is a no-op/getterfalse, and Dev Panel remains unavailable in Shipping. Character movement, input mappings and ammo HUD remain unchanged.

## FEAT225 Dev Panel blink selection (2026-09-30)
WBP_DebugMenu remains editable UMG. DebugContent is wrapped in DebugScroll with PLAYER/TESTING/CHARACTER/INFILTRATOR sections, normalized spacing, top HUD clearance and version footer. DebugMenuAssetLibrary.OrganizeBlinkControls explicitly authors/reorders these controls, preserving existing player/test/character actions. Button_BlinkDash/Dream/Portal route through UDebugMenuWidgetBase to current AFPSInfiltrator. Non-Infiltrator buttons disabled; selection cyan and BlinkSelectionHint shows actual grant/Q. Clicking stays in paused panel; reopening refreshes current pawn.


## FEAT230 fourth blink choice (2026-09-30)
WBP_DebugMenu adds Button_BlinkRift alongside Dash/Dream/Portal, with native SelectBlinkRift and four-choice highlighting/hint. Existing editable layout is upgraded idempotently by OrganizeBlinkControls. Selection remains current-pawn only; no new input mappings.


## FEAT231 fifth blink choice (2026-09-30)
WBP_DebugMenu adds editable Button_BlinkDirect and Direct label beside the retained four choices. SelectBlinkDirect routes to variant5; highlighting/hint now cover five choices. OrganizeBlinkControls remains idempotent and does not replace existing controls. No input mapping changes.


FEAT231 followup: Direct remains fifth button; tooltip now describes continuous astral pull, trailing hands and delayed audio. OrganizeBlinkControls updates the existing Direct tooltip idempotently. No new input.


## FEAT233 Dev Panel category range controls (2026-09-30)
WBP_DebugMenu adds ENEMY TRIGGER RANGES with global ShowAllEnemyRanges and exactly three WBP_EnemyTriggerRangeRow children: Phantom, Medusa, Boss/CoreMorph. Each switch controls ALL current and future instances in that category, never individual actors or gameplay activation. Category rows stay available on empty maps and show live counts/configured geometry counts. Global checkbox reflects category settings (including mixed state), not current actor population. UDebugMenuWidgetBase refreshes data every0.2s in NativeTick, including paused panel; native row behavior lives in UEnemyTriggerRangeRow, layout remains editable UMG. AddEnemyTriggerControls explicitly authors the two owned widgets. Existing sections retained;650px scroll column, consistent button sizing, God Mode and Infinite Ammo share a row; top88px clearance and scroll-to-start. No new inputs. Row BP class is assigned on main widget CDO.

## FEAT235 category position markers (2026-09-30)
Each editable WBP_EnemyTriggerRangeRow now has independent ShowRange and ShowPosition checkboxes. ShowPosition calls UEnemyTriggerDebugSubsystem.SetPositionsVisible(Category, bool), controlling all living current/future instances in that category. UEnemyPositionDebugWidget paints projected name/distance without a visibility trace, so markers remain visible through geometry. The volatile, hit-test-invisible viewport widget updates with actor/camera movement and does not capture input. State is world-local, survives panel reopen and resets on world teardown; Shipping disables creation/setters. Debug switches never alter gameplay activation or ability settings. Main/row UMG authored and compiled; final screenshot CategoryFinal00000.png and nine PIE checks in D:/UnrealWork/MedusaCombat.

## FEAT252 health-only invincibility
Controller owns transient bDebugInvincible defaultfalse with IsDebugInvincibleEnabled/SetDebugInvincibleEnabled; local non-Shipping only, follows same controller across character changes, fresh controller/session defaultsOFF. No save/travel persistence added. WBP_DebugMenu PLAYER row has independent editable Button_Invincible/InvincibleLabel, label Invincible: ON/OFF and cyan enabled state. Native widget toggles/refreshes on reopen; authoring helper AddInvincibleButton is idempotent. Existing God Mode flight and Infinite Ammo remain independent.


### FEAT-256: cursor and Dev Panel map buttons (2026-10-07)
- Config/DefaultEngine.ini UserInterfaceSettings.SoftwareCursors points Default/Hand to editable /Game/UI/Cursor/WBP_Cursor_Pointer and WBP_Cursor_Hand. Original downloaded Abstergo SciFiCursorPack PNGs imported to UI/Cursor/Textures; 36px widgets compensate Slate centered cursor positioning.
- DebugMenuAssetLibrary.ConfigureCursorWidget/AddMapButtons author editable BP layouts. MAPS follows CHARACTER with independent Test Map, Space Station, Lobby, Scorpion Recording and Main Game buttons. Widget callbacks call PlayerController.EnterDebugMap; current character is preserved. Existing EnterDebugTestMap remains a compatibility wrapper.


### FEAT258 test placement (2026-10-07)
Core/Debug/TestSpawnPreset defines a declarative static-mesh recipe; Core/Debug/TestPlacementSubsystem owns temporary world-local groups with CanPlace, PlacePreset, RemoveGroup, UndoLast and ClearAll. Non-Shipping TestMap game/PIE only. Validates flat supported ground/obstructions before atomic spawning; world teardown removes groups. No map writes or persistence. WBP_DebugMenu TEST PLACEMENT directly shows a Void button and shared Undo Last/Clear Placed controls, without a nested monster heading or part-specific button. DebugMenuWidgetBase binds optional controls and a soft WallPlacementPreset reference. DebugMenuAssetLibrary.AddPlacementControls explicitly authors/upgrades the editable UMG. /Game/Maps/Test/Data/DA_TestSpawn_VoidWalls defines two facing static collision walls plus real BP_VoidGazer. PawnClass/PawnRelativeTransform keep monster configuration in the data asset. Walls spawn first; Void uses its existing BeginPlay auto-attach and default BT controller. Subsystem validates pawn clearance, owns spawned pawn/controller in the same group, and destroys pawn before controller/support geometry. Further presets should use the generic subsystem, with monster names as the user-facing placement choices.


FEAT274 held fire: TogglePauseMenu and ToggleDebugMenu send Input.Weapon.PrimaryFireCanceled before pausing/UIOnly/FlushPressedKeys, alongside the existing secondary-release notification. Returning from menus cannot release an old full charge into a shot.

FEAT276 follow-up (2026-10-11): Dev TEST PLACEMENT now has Test Soldier preset /Game/Maps/Test/Data/DA_TestSpawn_TestSoldier referencing existing BP_Phantom_WeaponTest, pawn-only with88cm capsule+2cm offset,800cm ahead. SoldierPlacementPreset optional property/button/callback in DebugMenuWidgetBase; editable UMG authored via existing AddPlacementControls. Generic TestPlacementSubsystem accepts a pawn-only recipe. Validates forward candidate first then lateral+600,-600,+1200,-1200cm with full floor/support/overlap validation; player ignored in own placement queries. Does not alter/move/remove original test cube. Overlap failures include blocker actor name. Same group Undo/Clear/controller cleanup retained. Actual initial TestMap Void blocked by blue dissolve cube before fix, succeeds with same cube collision intact after nearby search;3soldier placement/undo/clear and actual UI callback pass. See FEAT276 archive.

FEAT276 airborne test support: TestSpawnPreset.PawnTransforms optionally defines multiple pawn transforms; empty retains legacy PawnRelativeTransform, including Void wall attachment. TestSoldier preset supplies3positions at lateral -180/0/180cm in one atomic group; each has ground/clearance validation. FTestSpawnMesh.ActorClass optionally uses an authored StaticMeshActor subclass; Test Cube button uses DA_TestSpawn_DissolveCube + BP_Test_DissolveCube with copied original blue material/DisintegrationComponent/profile and150cm edge. Cube BP root scale normalized1, recipe supplies1.5. Shared Undo/Clear removes the whole group including controllers. Existing placed map cube/labels preserved; nearby placement avoids them. WBP contains Void/Soldiers x3/Test Cube/Undo Last/Clear Placed. No runtime-created UI or persistent test-map changes.

2026-10-14 four-prop follow-up supersedes the single Test Cube preset above. The existing `Button_PlaceCube` reads `Props x4` and loads `/Game/Maps/Test/Data/DA_TestSpawn_DissolveCube`, whose recipes spawn Blue Cube, Orange Sphere, Station Vent and one seven-mesh SpaceStation Door. `ATestInteractionProp` owns the runtime meshes and DisintegrationComponent. On 2026-10-22 its HighlightComponent was removed at the owner's request: these fixtures are neither scannable nor interactable. `UTestPlacementSubsystem` validates all prop bounds and spawns the four actors as one transient undoable group; saved acceptance props and TestMap are untouched. Each part references an authored `/Game/Maps/Test/Fracture/GC_Test_*` Chaos asset. ExplosionGunBullet's radius query invokes `Fracture` on these opt-in test props; the actor hides its source meshes and starts the matching Chaos components. This is a TestMap fixture path, while the existing generic GeometryCollection explosion path remains. See `archive/FEAT-285-dev-panel-four-props.md`.

2026-10-22 powered interaction: `AInteractableBase::GetInteractionPromptState(Viewer)` returns a label plus visibility and E-enabled flags; default behavior keeps undiscovered scan-gated actors hidden. `ATheManPlayerController` reevaluates the state each tick, including when focus stays on the same actor, and prefixes `[E]` only when enabled. `/Game/Actors/Interactables/PoweredDoor/Blueprint/BP_PoweredDoor` derives from native `APoweredDoor`, uses seven SpaceStation door meshes, and reads `ATheManGameStateBase::IsPowered()`. `UMainPowerSwitchComponent` starts off by default and sets GameState accordingly. The door shows `No power` when off, `[E] Open door` when on, opens once with eased sliding panels, then disables collision and overlap. `AllowedSide` is instance-editable (Both/Front +X/Back -X). See `Documentation/Interaction/PoweredDoor.md` and FEAT-302 archive.


FEAT302 world interaction, revised 2026-10-21: `AInteractableBase` owns mesh components, Pawn-overlap `InteractionRange`, an empty `Interact(AFPSCharacterBase*)` BlueprintNativeEvent, and a per-actor `OnScanCompleted(AFPSInfiltrator*)` BlueprintNativeEvent. It has no scan state or highlight component. Normal interactables are usable immediately. An optional `UScannableInteractableComponent`, derived from `UHighlightComponent`, makes an `AInteractableBase` scan gated: `CanInteract` stays false until the scan wave calls `CompleteScan` once, which stores discovery and invokes the actor event. The controller's existing nearest eligible overlap selection then exposes `[E] <InteractionLabel>` and enables E; discovery persists after scan highlight retracts. `ARobotVentCover` owns the scan component, starts with its hinge at roll 110 degrees to cover the opening, eases between roll 110 closed and roll 0 open over 0.65 seconds, toggles with E after discovery, and keeps its interaction range and white proximity outline while nearby. The MainGame instance has an enlarged, editable `InteractionRange` Box that reaches the floor. `BP_ToggleSwitch` and `BP_ScanCube` have no scan component and do not receive scan highlights. See `archive/FEAT-302-scan-gated-interactables.md`.

FEAT-302 proximity highlight (2026-10-22): ATheManPlayerController requests white highlighting for every nearby AInteractableBase independent of prompt selection/eligibility. Register/Unregister, stale-overlap pruning, UnPossess and EndPlay manage the request. Prompt and E selection logic remains separate. See Documentation/Interaction/InteractableHighlight.md.

FEAT302 (2026-10-22): Powered door now uses fixed local sides: +X front needs power, configured card and E; -X back auto-opens from the existing interaction Box when powered, no card or E prompt. Closed Tick checks the back so power-on with a player already present opens the door. Legacy AllowedSide setting removed. Closing reentry requires powered side access. Existing animation and highlights preserved. Development Editor/Game builds and focused PoweredDoor PIE passed. Team docs: Documentation/Interaction/PoweredDoor.md and MainGameSpawnInteractables.md. Scoped P4 CL 121899 pending.
