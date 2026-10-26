# 音效接入规范

- 死亡后尸体中枪继续使用现有Enemy Hit肉体声与血迹，GCN_EnemyHit只对存活目标启动新PainSound；致命一枪的原痛呼可自然结束。开火/爆炸音频资产及用户音量不变。

- 2026-09-05晚新增Enemy爆炸：ExplosionGun/Audio/S_ExplosionGun_EnemyDetonation（程序合成1.3秒/48kHz/mono，非外部素材）→SCue_ExplosionGun_EnemyDetonation；Modulator pitch .95~1.05、volume .95~1，复用本枪SA_ExplosionGun_Detonation与SC_ExplosionGun_Detonation。只由爆炸GC的EnemyExplosionSound消费，环境Alien Cannon不改。配置/冷验证Scripts/Audio/configure_enemy_explosion.py，源生成器synthesize_enemy_detonation.py；ExplosionRadialDamage断言真实Enemy AudioComponent正在播放。Gameplay增益仍可能推高峰值，不宣称最终输出绝不削波。

- 2026-09-05：同一个Explosion GC新增EnemyExplosionSound/EnemyVolumeMultiplier，空槽不回退环境Sound Cue；目前Enemy专用声音未提供，保持空。环境SCue_ExplosionGun_Detonation/VolumeMultiplier=3保留，用户震屏最新覆盖=4。未来Enemy爆炸音频仍须封装随机Sound Cue，不能直接接裸Wave。验证Scripts/Audio/validate_explosion_feedback.py（-FeedbackValidateOnly冷只读）；痛呼回归现为TheManTest.Feedback.BulletTimeAndPain，旧HitStopAndPain已替换。

## 职责与所有权

- Gameplay Cue不是Sound Cue。前者决定触发时机/命中位置/目标分工，后者封装音频素材、随机和声音配置。
- 新增一次性玩法音效必须使用owner-local `Audio/SCue_<Owner>_<Purpose>`；源Wave保留原样作为叶节点，不让Gameplay直接引用裸Wave。
- 单素材用Wave Player→Modulator→Output；多素材先通过Random（无放回）选择，再Modulator。不要复制相同素材冒充音频变体。
- 随机只在一层实现，C++/Blueprint/AnimNotify调用端不再追加随机音高/音量。基础音量倍率仍可调。
- UI/扫描识别音/对白/音乐/循环音允许例外，但必须记录原因；不得将长循环简单套随机音高导致变调或循环接缝。
- 音效留在具体武器/敌人Audio目录。跨系统共享的命中衰减放Core/_Shared，三枪共用枪声/机械声配置放Weapons/_Shared；只有爆炸枪使用的爆炸配置留在ExplosionGun。

## 当前参数

| 用途 | 音高 | 音量 | 衰减资产 / 全量半径 + 衰减距离 |
|---|---|---|---|
| 开火（含TestGun） | 0.96–1.04 | 0.95–1.00 | Weapons/_Shared/Audio/SA_WeaponFire / 250 + 4750cm |
| 空仓机械声 | 0.97–1.03 | 0.95–1.00 | Weapons/_Shared/Audio/SA_WeaponMechanical / 100 + 900cm |
| 武器环境命中 | 0.95–1.05 | 0.95–1.00 | Core/_Shared/Audio/SA_ProjectileImpact / 180 + 2200cm |
| 肉体受击 | 0.92–1.08 | 0.95–1.00 | 同上 |
| Enemy痛呼 | 0.97–1.03 | 0.95–1.00 | 同上；每敌人独立冷却/并发 |
| 爆炸 | 0.96–1.04 | 0.95–1.00 | ExplosionGun/Audio/SA_ExplosionGun_Detonation / 300 + 5700cm |

- 均为球形、线性、3D空间定位，StereoSpread=0；未额外启用遮挡。Sound Cue.VolumeMultiplier固定1，避免引擎默认0.75意外压低声音。
- 并发上限：开火16、机械8、环境命中12、肉体8、爆炸8；StopQuietest，防止连发无限叠加。
- 用户Gameplay倍率保留：Enemy Hit音量5，Explosion音量3、震屏8。肉体Sound Cue路由到既有SMX_EnemyFleshHit限幅总线，不能丢失这层路由。
- Infiltrator扫描音为稳定启停识别音，暂保留裸Wave且不随机。RepairGun旧Equip与模板Fire Wave没有引用，未凭空增加切枪/换弹播放行为。

## 命中声音分工

- GCN_ImpactFeedbackBase::ShouldPlayImpactSound：命中ACharacter不创建武器声音，环境正常；Niagara保持所有目标相同、武器贴花仍只落环境。
- GCN_EnemyHit重写上述策略，自身Hit Cue负责肉体声。禁止因目标没有配置Hit声就回退播放武器环境声。
- PitchVariation/CharacterSoundMultiplier旧字段已移除。ImpactAttenuation/ImpactConcurrency保留为显式调用覆盖，与Sound Cue所选资产一致，不是第二层衰减/并发叠乘。

## 新增音效必做检查

1. 确定用途/所有者，建立Sound Cue与正确源Wave，按用途设置轻微随机或登记例外。
2. 选择对应用途的衰减及并发；需要不同传播范围时独立配置，禁止全项目套一个半径。
3. 更新实际消费者引用；Blueprint打开/编译/保存，检查AnimNotify等入口不重复播放或随机。
4. 冷加载验证Cue图连接、源Wave、衰减/并发和消费者；PIE确认连续播放有细微变化，世界声有左右/远近差异。
5. 将正式新增消费者加入Scripts/Audio/configure_audio_cues.py清单或等价的专属验证测试，不能只靠记忆。

## 工具与验收入口

- Core/Editor/TheManAudioAssetLibrary.InitializeVariationCue：仅初始化空Sound Cue，不覆盖已编辑图；标准UE节点在运行时执行，没有新增全局播放Subsystem。
- Scripts/Audio/configure_audio_cues.py：显式12项现用音效消费者接入；`-AudioValidateOnly`只读冷验证。该脚本不是全项目任意Wave自动覆盖工具。
- TheManTest.Audio.AssetVariationPolicy：12个图/源Wave/参数/并发/衰减、未来多素材及禁止覆盖测试。
- TheManTest.Player.Weapons.SpatialImpactFeedback：实际PIE命中声分工和左右/远近录音；StickyExplosionAndBlood验证真实角色Hit声及既有伤害/血迹。
- 新痛呼验收：Scripts/Audio/configure_enemy_pain.py导入指定424116音频并创建SCue_Enemy_Pain/SC_EnemyPain，`-AudioValidateOnly`冷回读；并发1、LimitToOwner、PreventNew，组件另做0.6秒真实时间冷却与播放中防重入。TheManTest.Feedback.HitStopAndPain验证两敌人独立、冷却、重播和销毁停止；StickyExplosionAndBlood验证真实Hit路径痛呼与肉体声同时播放。


## User volume routing (FEAT-136)

/Game/Core/Audio owns SClass_Effects, SClass_Music and SMix_UserVolumes. This is actual cross-system policy: weapons, enemies, character scan and lobby ambience all use Effects through Engine.AudioSettings.DefaultSoundClassName; existing source waves/cues retain null explicit class and their source gains, randomizers, attenuation, concurrency and flesh-hit submix. Do not assign a new common submix or rewrite source asset loudness. DefaultBaseSoundMix is SMix_UserVolumes. PlayerSettingsSubsystem applies master*effects and master*music once via class overrides with .05s interpolation and applies after every world initialization. Do not add another master multiplication upstream.

Music has bIsMusic=true; no music track exists in the audited project. Future music must explicitly route to SClass_Music (SoundBase.SoundClassObject or AudioComponent.SoundClassOverride). New ordinary SFX inherit the default Effects class; explicitly customized classes must join the intended category hierarchy. Native UI sliders preview; Apply saves config, Back/destruction restores last applied values. Settings do not alter source SoundCue variation policy.


FEAT-143 (2026-09-22): user requested removing Lobby background sound. All13 Lobby AmbientSound actors and the AudioComponents in Lobby-only BP_Fan/BP_Gravity_Reactor_Rotator were removed. Fresh PIE verifies zero scene AudioComponents; Lobby/these BPs have no Audio package dependencies. Existing global Effects/Music routing and gameplay SFX remain unchanged. Historical original waves/cues are retained as unreferenced source assets.


## FEAT162 Executive drone audio (2026-09-26)

FEAT175 更新：按用户要求，扫描 Start/Loop 的源替换为 Downloads 中 `1254509.audio-ELECEmf-Glitchy_Printer.wav`，原文件保留。沿用现有两个 Wave 路径及 Cue/消费者；派生单声道48kHz PCM16，开头0.34秒，持续段取原声0.24–12.5秒（12.26秒），末尾0.15秒与开头前导段交叉衔接。音高和源增益不变，保留原有开头重叠、循环、空间衰减、并发和完成淡出。下文100835来源及0.16秒循环是历史配置，已被本次替换。拒绝部署音效保持249205。验证复用 Scripts/Audio/validate_drone_scan_audio.py，冷加载及 SpaceStation 实际部署录音见 D:/UnrealWork/DronePrinterAudio。

Owner /Game/Characters/TheExecutive/Gameplay/Drone/Deployment/Audio: SCue_TheExecutive_DroneScanStart/DroneScanLoop/DeploymentRejected wrap corresponding S_ Waves. Specified sources100835-Laser_scan_07-SFXBible-ss05993.wav and249205-Game_Error_4.wav remain unchanged in Downloads. Scan derived mono attack .34s and .16s middle-section overlap loop retain source rate/pitch; attack ends with .10s crossfade, loop starts at .24s with .10s fade-in. Completion .18s fade; abort stops. Scan Cues use SA_TheExecutive_DroneScan:3D linear sphere150cm full+1850cm falloff; SC_TheExecutive_DroneScan count2/owner/StopOldest accommodates attack and sustain. Loop Wave Player loops, virtualization PlayWhenSilent preserves position.

Error cue uses exact1s source,2D feedback and SC_TheExecutive_DeploymentRejected count1/owner/StopOldest, shared .18s click/shake throttle. No random pitch/volume: explicit scan-recognition/loop/UI exception, Cue Modulator min=max1, native call sites do not randomize. Default Effects SoundClass routing retained; no global volume/source-gain edits. Consumers are BP_TheExecutive_DroneTurret.Construction.ScanStartSound/ScanLoopSound and BP_TheExecutive.DroneDeployment.RejectionSound. Read-only manifest validator Scripts/Audio/validate_drone_scan_audio.py checks actual references, graph leaves/loop flags, stable pitch/volume, attenuation and concurrency. Actual PIE master recordings and lifecycle checks under D:/UnrealWork/DroneAudio.


## FEAT166 Infiltrator source07 scan audio (2026-09-26)

Latest user instruction replaces the previous S_Infiltrator_Scan playback with the original TMIIR07 start/loop/end set. Owner path /Game/Characters/Infiltrator/Gameplay/Abilities/Scan/Audio contains SCue_Infiltrator_ScanStart/Loop/End and corresponding S_ Waves, migrated natively; SC_Infiltrator_Scan max3/owner/StopOldest caps overlap. Original one-leaf Cue graphs, source pitch1 and Cue gain.75 retained: explicit stable scan-recognition/loop exception, no random Modulator or native randomness. Loop wave/player both loop, Cue PlayWhenSilent;20s source repeats at unchanged pitch. New audio inherits default Effects SoundClass/user settings; no global volume changes.

BP_Infiltrator.ScanPresentation consumes StartSound/LoopSound/EndSound, gain1, loop fade-in1s/fade-out.5s. AudioComponents are owner-managed local2D feedback without distance attenuation. Start+loop activate once; return stops attack, fades loop, plays closing1.0719s; closing may outlive visual return, with owner validation active until tail finishes. Restart, invalid owner and EndPlay stop/destroy all audio; normal completion disables tick. BGA_Infiltrator_Scan ScanActivateSound and ScanDeactivateSound both null: the former bare Wave is no longer played. Source07 assets retained; the replaced unused bare wave was subsequently deleted by FEAT168 after reference verification.

Read-only consumer/graph/reference policy manifest: Scripts/Audio/validate_infiltrator_scan_audio.py. Source/migration/cold validation and SpaceStation actual gameplay recording/lifecycle evidence under D:/UnrealWork/ScanAudio.

FEAT168: deleted the replaced bare S_Infiltrator_Scan after native/config/scripts/registry checks and null legacy ability fields; source07 cues and all drone sounds preserved with updated consumer manifests.


## FEAT209 Infiltrator blink sound
Owner Characters/Infiltrator/Gameplay/Abilities/Blink/Audio: S_Infiltrator_Blink -> SCue_Infiltrator_Blink (WavePlayer -> Modulator -> Output), supplied Downloads Zoom_On_The_Map_02 full1.57704s stereo converted192kHzPCM24 to48kHzPCM16, no source gain change. Pitch.98?1.02/volume.97?1 only inside Cue; gain1, SC_Infiltrator_Blink max1/owner/StopOldest, local2D without attenuation, inherits Effects. BlinkPresentation.BlinkSound starts once after accepted blink, success lets it finish past visual tail; restart/cancel/death/unpossess/EndPlay stop/destroy owned AudioComponent. Read-only consumer manifest Scripts/Audio/validate_infiltrator_blink_audio.py. Actual master recording/cold checks and lifecycle tests passed; evidence D:/UnrealWork/BlinkSmear.


## FEAT231 Direct blink audio timing (2026-09-30)
DirectBlink is another consumer of the existing BlinkPresentation SCue_Infiltrator_Blink, released once at presentation age>=.1s after instantaneous relocation. Cue graph/randomization/concurrency/gain unchanged; no new Wave/Cue or caller randomness. A transient owned SoundMix adjusts existing SClass_Effects/SClass_Music and descendants to.001 for the initial100ms, then pops with25ms fadeout. No user settings/class asset edits. Cancel before100ms suppresses the delayed cue and restores mix; cancellation afterrelease uses existing presentation shutdown on grant replacement/unpossess. Actual Effects and Music master recordings measured silence ratios.00056/.00033 with recovery; logic release106.7ms on60Hz test, audible return~125ms including audio scheduling. Evidence D:/UnrealWork/BlinkV5/audio-measured.json and audio/*.wav.

FEAT231 nightmare awakening: existing Blink SoundCue now plays at travel completion320ms; transient Effects/Music duck releases at that same event (25ms fadein/15ms fadeout). Previous100ms release is obsolete for Direct5. No new breath/impact assets, randomization or user-volume edits.


FEAT231 eyelid blink supersedes320ms arrival timing: existing Blink Cue and Effects/Music duck release at presentation age110ms when eyelids begin reopening, after the75ms safe relocation. Same25ms duck fadein/15ms fadeout, original Cue/randomization/gain, no new audio assets or settings. Blocked landing/cancel before reopening suppress delayed sound and restore mix.


FEAT231 dynamic pull: existing Blink Cue/Effects+Music duck now release at320ms actual travel completion. Prior110ms timing obsolete; same original audio assets/gains/randomization,25ms duck attack/15ms release and cancellation ownership.


## FEAT242 Medusa original spatial voices
MedusaCombatComponent exposes SpellSound/DashSound/MeleeSound/DeathSound under Combat|Audio. BP_Medusa assigns owner Audio/SCue_Medusa_{Spell,Dash,Melee,Death}. Accepted GAS BeginAction plays one attached head voice; normal FinishAction preserves voice tail, cancellation/new action/death stops attack voice, EndPlay stops all. Death cleanup plays separate DeathAudio once; shared death presentation/body mode unchanged. Source original Sevarog Effort Ability_E/Ability_RMB/Swing/Death respectively, three native waves each. No synthesized sound, dialogue, added warning, or DarkVeil timing changes.

Owner /Game/Enemy/Nightmare/Medusa/Audio:12 original stereo Wave copies retain sample data/duration; source class cleared to inherit Effects. Four Cue graphs WavePlayers -> Random(no replacement) -> Modulator(.98..1.02pitch,.97..1volume) -> Output, Cue gain1. SA_Medusa_Voice linear3D sphere200cm full+2800cm falloff, StereoSpread0 and stereo normalizationtrue, no occlusion. SC_Medusa_Voice max1/owner/StopOldest prevents stacked vocalizations without suppressing other Medusas. C++ gain/pitch1; randomization only inCue. Consumer/dependency/graph manifest Scripts/Audio/validate_medusa_audio.py; real GAS/recording evidence D:/UnrealWork/MedusaAudio.

## FEAT254 Void player ground impact
Owner /Game/Enemy/Nightmare/VoidGazer/Audio contains S_VoidGazer_GrabSlam, SCue_VoidGazer_GrabSlam, SA_VoidGazer_GrabSlam and SC_VoidGazer_GrabSlam. Source is independently synthesized dry heavy thump (.65s mono48kHzPCM16, low sine pitch-drop plus short noise/body transients); source WAV remains D:/UnrealWork/VoidGrabSlam, not a migrated marketplace recording. Cue WavePlayer->Modulator pitch.96..1.04/volume.95..1, Cue gain1/pitch1; no native randomness. Attenuation linear3D sphere250cm full+2250cm falloff, concurrency2 StopOldest. Inherits Effects default routing. BP_VoidGazer.Tentacle.SlamImpactSound consumes cue once on verified ground slam, including InvincibleON. No sound on early escape/cancel/no-floor. Existing one-shot AudioComponent auto-destroys naturally. Manifest Scripts/Audio/validate_void_grab_slam_audio.py; cold graph/deps and live actual playing component verified. Master recording SlamActual.wav nonzero peak.3065/RMS.0474.


## FEAT271 directional blink audio
V6 owns BlinkV6/Audio/S_Infiltrator_DirectionalBlink and SCue_Infiltrator_DirectionalBlink. Independently synthesized .32s mono48kHz electronic chirp/noise whoosh, external source D:/UnrealWork/BlinkV6/BlinkWhoosh.wav; not original Overwatch audio. WavePlayer -> Modulator pitch.98..1.02/volume.97..1 -> Output; Cue gain1 and retained SC_Infiltrator_Blink concurrency max1/owner/StopOldest. Local2D, no distance attenuation, default Effects routing. DirectionalBlink.BlinkSound supplies accepted activation override to existing BlinkPresentation owner-managed AudioComponent; legacy cues unchanged. Read-only manifest Scripts/Audio/validate_infiltrator_directional_blink_audio.py. Real PIE IsPlaying verified.


## FEAT274 disintegration charge/fire audio (2026-10-11)
Owner /Game/Weapons/DisintegrationGun/Audio: S_DisintegrationGun_{Charge,Fire}, SCue_DisintegrationGun_{Charge,Fire}, SC_DisintegrationGun_Charge. Downloaded user-selected408363 charge preblast0..1.13s fitted pitch-preserving to.8s;414390 finalpulse.965..2.68s,1.715s. Mono48kHzPCM16, short edge fades; sources and edit manifest D:/UnrealWork/DisintegrationAudio/edits.json. Cues WavePlayer->Modulator->Output, gain1/pitch1, default Effects routing. Charge Modulator pitch1..1/volume1..1 intentionally fixed to synchronize .8s gameplay charge; no pitch randomness or loop. Shared SA_WeaponMechanical sphere100+900cm, owned concurrency1 perowner StopOldest. Fire Modulator pitch.96..1.04/volume.95..1; shared SA_WeaponFire sphere250+4750cm and SC_WeaponFire16 StopQuietest. Native caller gain/pitch1; no duplicate randomness. BP charge_component.charge_sound and FireSound consume cues. Charge owned attached component cleaned on cancellation/release; fire uses existing accepted GA_Shoot spatial ActiveSound. Read-only cold manifest Scripts/Audio/validate_disintegration_audio.py PASS. Actual PIE charge IsPlaying/cancel and accepted release verified; foreground master FireActual.wav peak.389893/RMS.053740. No subjective listening acceptance claimed.


2026-10-11 charge audio intro correction: user requests only charge portion. Replaced S_DisintegrationGun_Charge with original408363 source .33..1.13s (exact.8s), removing initial transient; no time stretch,8ms fadein/20ms fadeout. Supersedes previous0..1.13s/atempo1.4125 cut. Fire/cues/gameplay unchanged. UE automated replace import saved duration.800000012, existing graph/consumer validator PASS, zero dirty content. No native change/rebuild required. Evidence D:/UnrealWork/DisintegrationAudio/trim.json and trim-installed.json; checkpoint61e991e0. Final not committed/submitted; subjective listening review remains with user. PIE stopped for import; TestMap open.


## FEAT-303 Infiltrator Spider Tank audio (2026-10-20)

The source Spider Tank's seven Wave assets and seven Cue graphs live under `/Game/Characters/Infiltrator/Gameplay/SpiderTank/Audio`. The Cue assets use `SCue_Infiltrator_SpiderTank_*` names, volume multiplier 1.0, 3D attenuation and concurrency: Fire uses shared WeaponFire settings; MeleeHit uses shared ProjectileImpact settings; Activation, MeleeSwing, Stomp, Remote and Walk use shared WeaponMechanical settings. Native robot playback also supplies these spatial/concurrency settings explicitly. The source Cue graphs remain single WavePlayer nodes; no pitch randomization was added to the imported sounds, preserving their supplied timbre. Walk is a movement-controlled ambient component, stopped when the robot stops. Fire, melee swing/hit, activation, remote entry and landing have separate source cues and call sites.
