# GAS 技能系统

> 当前清理状态（2026-09-13）：用户要求立即清理测试设施，取代下文旧批次“总验收后清理”的安排。5张CoreMorph检查地图、专属地面材质、5个Tests cpp与CoreMorphFlightReview h/cpp均已删除。Boss的StartFlightPreview／ResetFlightPreview及Flight的ResetPreview已删除；Reassembly内部ResetPreview改名ResetAssembly，仍用于正式变形初始化。ReviewTarget已删除，正式BT目标由LastThreat／玩家决定。保留FlightRoute、正式主BT、GA／GE／Cue及490个头领资产。以后只有TestMap作为测试地图，不能再照旧地图入口操作。


- FEAT-081 第三批：`GA_CoreMorphTailStrike` 通过现有 `PhaseSkillSets[0].NearAbilities` 授予一次，DefaultAbilities 仍仅 Flight／Reassemble，共三个能力。GA 管理 Windup→Thrust→Recover，订阅同头领战斗组件的阶段完成和接触事件；组件不决定何时启动技能。`GE_CoreMorphAttacking` 持有攻击 Tag；GA 结束移除并施加 `GE_CoreMorphTailCooldown`，时长读取本头领参数；`GE_CoreMorphTailDamage` 使用负值 Data.Damage 修改目标共享 Health。取消后无伤害地收回，死亡清状态和停止运动。无源 ApplyPointDamage 路径，无新增音效／粒子资产，现有 Cue 分工不变。

- FEAT-081 第二批：同 ASC 默认授予 Flight 和 Reassemble 各一次。`GA_CoreMorphReassemble` 用 `GE_CoreMorphTransforming` 持有 `State.CoreMorph.Transforming`，阻止重入和飞行；成功时具体头领 SetForm 更换 Manta/Scorpion 无限 GE，不更改战斗阶段、Health 或技能列表。`GC_CoreMorph_Reassembly` 注册在头领专属 GAS/GameplayCues，负责金属流／沙尘池和灯光生命周期；取消立即移除，成功保留至冲击后 8 秒。此处迁入的是源单向变形，R 仍为检查复位，不是逆向技能。第二批验证见 FEAT-081 archive。

- FEAT-081 第一批：`Enemy/Boss/CoreMorph/GAS/Abilities/GA_CoreMorphFlight` 通过同头领 ASC 激活，结束／取消统一停飞；`GAS/Effects/GE_CoreMorphManta` 用具名 `UTargetTagsGameplayEffectComponent` 默认子对象授予 `State.CoreMorph.Form.Manta`。死亡移除形态 GE。预览复位不重授技能或回血，战斗阶段不改形态。当前只迁飞行，无攻击／重组 Cue；完整 GA／GE／Cue 战斗按后续批次接入。

- 死亡布娃娃：Health回调仍负责首次OnDeath，但Actor延时销毁。ABulletBase施加伤害后在命中骨骼处施加物理冲量，致命一枪同样生效；已有尸体跳过GE扣血，通过显式Enemy Hit Cue保留肉体声/血迹，GCN_EnemyHit对死人不新触发PainSound。爆炸仍仅新击杀或Chaos新破碎请求子弹时间。

## 2026-09-05 人形共享全身爆炸受击（覆盖历史Phantom专属Rig路径）

- 共享资产`Enemy/Humanoid/_Shared/Animations/ControlRig/CR_Humanoid_HitReaction`及`ABP_Humanoid_HitReaction`；ABP为无TargetSkeleton的模板，通过Humanoid基类HitReactionPostProcess自动接入。Rig清空PreviewMesh、SourceHierarchyImport、SourceCurveImport，没有Phantom资产依赖。HumanoidReactionFrame经原生AnimInstance传入Rig，含Torso/Follow/Compression/BoneMapping，原生节点按映射处理脊柱、头肩跟随、髋部下沉和双腿脚位保持。默认38度/.85秒恢复，头肩延迟.045秒，腿压缩7cm；胶囊不移动。
- 正式触发仍为爆炸对存活Enemy的范围伤害，普通直接命中未额外接这套Rig。身体血痕由Character拥有的Decal组件实现，死亡随Owner清理；声音和环境血迹规则不变。旧Audio配置脚本不再创建Phantom Rig，新的显式安装/冷验证入口位于Scripts/VFX/*shared_humanoid_reaction.py。

## 2026-09-05 玩家实体弹准星汇聚

- 仰射修正覆盖下文Visibility取点：Projectile瞄准现在使用子弹ObjectType+CollisionResponse的simple射线，命中Character后沿准星射线进一步查Mesh PhysicsAsset，避免Enemy未响应Visibility/complex导致退化到远点。枪口预遮挡遇Character时仅实际Mesh穿过镜头→枪口线段才视为即时阻挡；胶囊假阻挡被排除后继续查其他组件/墙体。真实弹体飞行仍可被枪口前其他身体部位挡住。StickyUpwardAim覆盖1/2米与0/30/60度，实际竖直误差<4cm（保留AttachmentOffset4）；极近大仰角不保证跨身体遮挡仍与准星重合。

- GA_Shoot读取PlayerController.GetPlayerViewPoint的实际视点（无Controller时HeadCamera兜底）。Projectile分支由Visibility中心射线求AimPoint，无命中时用HitscanRange；弹体从枪口朝AimPoint发射，零向量/身后目标回退CameraForward，避免近距倒射。
- 镜头至枪口按子弹CDO球半径、ObjectType及CollisionResponse做Sweep；有阻挡时在表面近侧生成并调用既有ProcessHit，防止枪口穿墙/穿入目标后漏判。无阻挡保持枪口生成，后续伤害仍由实体弹飞行碰撞结算。忽略玩家及挂载装备；没有把胶囊判定改成逐骨骼判定。
- ProjectileCrosshairAim真实PrimaryFire验证1.5/3/10m目标面轨迹误差<0.1cm和实际附着，以及20cm墙面、枪口在墙后时的近侧阻挡。ProjectileAim.log三项Success：此测试、StickyExplosionAndBlood、ThreeWeaponPIESwitch。

## 2026-09-05 当前Enemy Air 007（覆盖下文Enemy Ground配置）

- EnemyEffectScale现可直接控制Air007内部尺寸：22个发射器ParticleUpdate接ApplyOwnerScaleToAttributes，Owner Scale链接Engine.Owner.Scale；Sprite/Ribbon/CameraOffset开启，世界空间另开启初速度/力/Mesh缩放，本地运动沿用组件变换，Drag不缩放。当前Cue保存值0.1；倍率只影响表现，不影响爆炸伤害半径。Editor自动化ExplosionScaleAudit默认只读，仅显式-InstallEnemyScale可修改并保存该System。

- 同一Explosion GC的EnemyExplosionEffect改为本枪`Effects/EnemyExplosion/Systems/NS_ExplosionGun_EnemyDetonation`（TMIIR N_ExplosionAir_007）；EnemyEffectOnGround=false，Niagara使用Params.Location实际身体爆点，不依赖GroundHit。环境ExplosionEffect仍Ground006与原地面投射。
- 子弹时间不在Cue触发；弹体仅在本次范围伤害击杀Enemy或真实Chaos新破坏时请求，具体生命周期见arch09。声音、震屏、Control Rig和原伤害不变。

## 2026-09-05晚 当前Enemy爆炸（覆盖下文空槽状态）

- 同一GC_Weapon_ExplosionGun_Explosion的EnemyExplosionEffect接NS_ExplosionGun_Detonation（已迁入的TMIIR N_ExplosionGround_006），bEnemyEffectOnGround=true。Enemy与环境仍独立字段，当前按用户指定使用同个System；地面类效果必须GroundHit投影，不悬挂在身体/方块上。以后换非地面效果可关闭EnemyEffectOnGround，在实际爆点播放。
- EnemyExplosionSound=SCue_ExplosionGun_EnemyDetonation，独立生成声；环境ExplosionSound仍是指定Alien Cannon。用SpawnSoundAtLocation自动销毁组件，声音留实际爆点且使用Sound Cue的3D/并发/随机。伤害与Control Rig触发由弹体/敌人组件负责。
- 用户最新CameraShakeScale=6、环境音量3；EnemyEffectScale/EnemyVolumeMultiplier保留原配置。TimeScale=.05、SlowIn=.01、Hold=1、Recovery=.01原样保留。

## 2026-09-05 爆炸表现当前入口（覆盖下文历史值）

- 同一GC_Weapon_ExplosionGun_Explosion：环境ExplosionEffect/ExplosionSound/EffectScale/VolumeMultiplier；EnemyExplosionEffect/EnemyExplosionSound/EnemyEffectScale/EnemyVolumeMultiplier独立，无回退。目前Enemy声/VFX两槽为空，等待用户资源，伤害/Chaos/震屏/子弹时间仍执行。
- Explosion|Camera：保留用户CameraShakeScale=4（运行时上限8）、ShakeInner/OuterRadius=200/1800；新增ShakeDuration=.75真实秒、ShakeFrequency=12Hz、ShakeRotationDegrees=1.5。多次衰减余震，零相机位移；Cue初始化Pattern，每相机同类爆炸仅一条，防止无限叠加。方向仍由爆点→视点决定；未改ControllerRotation、手臂挂载或普通开火震屏。
- 环境爆炸音量3，Enemy肉体5，痛呼1。HitStop已删除，改为弹体向Core/_Shared/Feedback/BulletTimeSubsystem请求平滑子弹时间；GC只负责表现，详见arch09。

## 随机命中与空间音频（当前配置）

- Enemy Hit额外播放PainSound=`Enemy/_Shared/Audio/SCue_Enemy_Pain`（下载424116 Wizard Pain），独立PainVolumeMultiplier=1、PainCooldown=.6真实秒，原肉体ImpactSound/倍率5不变。懒创建UEnemyHitAudioComponent保存每敌人冷却与弱AudioComponent，播放期间不重叠，声音附着目标，目标结束时停止；静态GC自身不持有运行时冷却。
- 爆炸GC只负责Niagara/声音/震屏，不处理HitStop或二次伤害。Data.Explosion.EnemyImpact元数据选EnemyExplosionEffect（默认空），使用真实爆点；无该标记时原ExplosionEffect仍使用Ground投影。Enemy分支空VFX不代表跳过声音、震屏、Chaos或伤害。新Gameplay二次伤害在ExplosionGunBullet中处理，详见arch09。

- `UGCN_ImpactFeedbackBase::SpawnImpactSound`统一在命中点创建自动销毁AudioComponent，传入ImpactAttenuation/ImpactConcurrency。随机已移到Sound Cue，原PitchVariation/CharacterSoundMultiplier删除；新增音效强制遵守arch/14-audio-policy.md。
- 当前EnemyHit.VolumeMultiplier为用户设置的5，覆盖下文历史值1。打人时武器Impact完全不出声，由角色自己的Hit Cue发声；ShouldPlayImpactSound由EnemyHit重写拥有自身声音。四项命中Cue共用`/Game/Core/_Shared/Audio/SA_ProjectileImpact`：180cm内全量，外加2200cm线性衰减，Spatialize开启、StereoSpread=0。并发分别SC_ProjectileImpact=12与Enemy专属SC_EnemyFleshHit=8，StopQuietest。
- FleshHit源音频不改样本，输出到Enemy/_Shared/Audio/SMX_EnemyFleshHit，经SFX_EnemyFleshLimiter限制过大峰值；并非保证所有主输出绝不削波。
- 贴花随机旋转、尺度/长宽，并用独立MID的DecalPatternOffset改变Opacity噪声。两枪新增owner-local M_*_ImpactDecalVaried母材质，原MI参数保留；血迹原M_Enemy_BloodStain扩展同样参数。DecalSizeVariation/BloodSizeVariation可调。
- 身体血迹先Trace Mesh表面，失败向最近骨骼补Trace；使用实际表面法线/骨骼附着，无法命中Mesh则跳过，避免贴在胶囊外悬空。BodyStainProjectionDepth默认12cm；附着随骨骼刚性移动，不等于蒙皮纹理绘制。环境血迹和喷溅卡片同样随机，寿命12秒/0.55秒不变。

## 爆炸延时 Cue 与 Enemy 血迹（2026-09-04）

- Explosion Cue参数契约：Params.Location/Normal为真实附着爆点，音效与方位震屏继续使用此位置；Params.EffectContext.HitResult为弹体查询的ExplosionGround地面，仅Niagara使用ImpactPoint+Normal(1cm)。Context没有地面Hit时不生成Ground Niagara/内含decal，不回退Cube表面；Chaos物理不在Cue内。

- 最新用户覆盖：正式Explosion Cue的VolumeMultiplier=3（Alien Cannon素材），CameraShakeScale实际回读8且保留。EnemyHit.VolumeMultiplier=1。下文原生默认/初次接入配置不覆盖用户当前蓝图值。

- 零伤害弹命中：ABulletBase仅在Damage==0的有效非穿透首次Hit显式调用Enemy.ExecuteHitReactionCue(Context,0,true)。新可选bAllowZeroDamageHit默认false，原Health回调仍只接受正伤害；显式零伤害使用即时Invoke，不修改Damage，不重复播放。电击弹Damage保持0也可播放默认血花。

- 爆炸Cue使用用户提供的Alien Cannon音频 `Weapons/ExplosionGun/Audio/S_ExplosionGun_AlienDetonation`，近满幅素材保持VolumeMultiplier=1。`UExplosionCameraShake` / `UExplosionCameraShakePattern`（Weapons/ExplosionGun/Effects）提供0.45秒方位冲击，CameraShakeScale=3。Cue按爆炸点→本地相机方向传入UserDefined播放空间，200cm内全强、1800cm外不震、中间平方衰减；与开火震屏和ControllerRotation独立。默认Enemy Hit的ImpactSound为 `Enemy/_Shared/Audio/S_Enemy_FleshHit`，VolumeMultiplier=1；旧合成爆炸音频已无引用删除。

- `GameplayCue.Weapon.ExplosionGun.Explosion` → `/Game/Weapons/ExplosionGun/GAS/GameplayCues/GC_Weapon_ExplosionGun_Explosion` → `UGCN_ExplosionGunExplosion`。由附着弹Timer触发，不合并原ImpactCue；ExplosionEffect/ExplosionSound/EffectScale/VolumeMultiplier独立配置，Niagara按+Z对齐法线。EffectLifeSpan默认8秒，通过弱绑定Timer兜底清理源效果的长尾焰，防止连续射击累积。无伤害GE。
- 默认 `GameplayCue.Character.Enemy.Hit` → `UGCN_EnemyHit` 现在配置 `BloodSprayMaterial/BloodStainMaterial/BloodScale/BloodStainLifeSpan`，默认12秒血迹，0.55秒喷溅。仍由真实Health扣减后的敌人ASC调用，不在某把枪内添加敌人分支。
- `Enemy/_Shared/Effects/EnemyBloodSpray.h/.cpp` 是默认EnemyHit的瞬态表现Actor（9个无碰撞/无阴影卡片、短弹道、朝向相机、MID Fade、自动销毁）。共享默认Cue由Phantom与其他Enemy继承；差异化敌人可改自身HitReactionCueTag。
- 源PNG、Texture与喷溅/贴花材质归 `/Game/Enemy/_Shared/Effects/Hit`；附着到敌人Mesh/bone的血迹与近处墙/地面血迹由敌人Cue负责，武器ImpactFeedbackBase的环境贴花规则保持原样。

**何时读取：** 新增或修改 GAS Gameplay Ability、调试开火流程、新增 Gameplay Tag 时。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Core/_Shared/GAS/TheManGameplayTags.h` | 全局 Tag 声明；定义在对应 Private 路径 |
| `Source/TheManTest/Public/Weapons/_Shared/GAS/Abilities/GA_Shoot.h` | 多枪械共用的玩家射击技能；实现位于对应 Private 路径 |
| `Source/TheManTest/Public/Weapons/_Shared/GAS/Abilities/GA_Reload.h` | 多枪械共用的玩家换弹技能；监听 `Input.Weapon.Reload`，校验当前 Firearm 的 `CanReload()` 后调用 `ReloadMagazine()` |
| `Source/TheManTest/Public/Characters/Infiltrator/GAS/Abilities/GA_InfiltratorScan.h` | Infiltrator 专属扫描技能；实现位于对应 Private 路径 |
| `Source/TheManTest/Public/Enemy/_Shared/GAS/Abilities/` | 多种 Enemy 可复用的 `GA_EnemyShoot`、自动射击、换弹和掩体技能 |
| `Source/TheManTest/Public/Enemy/Humanoid/Phantom/GAS/Abilities/GA_EnemyAreaBarrage.h` | Phantom 专属二阶段区域轰炸技能 |

`UGA_EnemyShoot` 公共基类统一处理人形敌人的渐进散射（基础、逐发扩散、上限、移动惩罚、恢复）和枪口 Niagara；三连发/扫射子类只负责节奏。默认人形步枪特效为 `/Game/Enemy/Humanoid/Phantom/Effects/Muzzle/Systems/NS_HumanoidRifle_Muzzle`，具体技能蓝图可覆盖；跨系统共享依赖位于 `/Game/Core/_Shared/Effects/Muzzle/`。

FEAT-080 起，玩家 `UGA_Shoot` 取得当前 `AFirearm` 后首先调用 `ConsumeRound()`。空弹仅播放当前枪械独立的 `DryFireSound` 后结束 Ability，不生成弹体，也不播放实弹蒙太奇/音效、Niagara、震屏或后坐力；成功扣弹会同步广播 `OnAmmoChanged` 更新 Combat HUD。
命中反馈采用独立所有权的两层 Cue：`ABulletBase::ImpactCueTag` 由具体武器/弹体选择武器命中表现；敌人 Health 确认扣减后，由目标 ASC 按 `AEnemyBase::HitReactionCueTag` 选择敌人受击表现。共享 `GE_BulletDamage` 只负责伤害，不绑定表现 Cue，也不要为 Cue 单独创建空壳 Gameplay Effect。`ABulletBase` 已将该 GE 设为 C++ 默认 HitEffectClass，新子弹 Blueprint 只需配置 Damage；需要治疗或其他效果的特殊子弹可覆盖。
武器命中 Cue 不区分环境与 Character：`UGCN_ImpactFeedbackBase` 始终播放同一 `ImpactEffect`，贴花只落环境。不同敌人的血花、受击闪光或受击声必须放在各自 `HitReactionCueTag` 对应的 Cue；`GameplayCue.Character.Enemy.Hit` 是默认兜底，同 Tag 不会自动区分敌人类型，需要差异时为敌人定义专属 Tag 并在其 Blueprint 覆盖。
FEAT-080 新增原生 Tag `GameplayCue.Weapon.ElectricGun.Impact` 与 `GameplayCue.Weapon.ExplosionGun.Impact`；对应 Cue 扫描路径在 `DefaultGame.ini` 的 `AbilitySystemGlobals.GameplayCueNotifyPaths` 中逐武器注册。Tag 必须继续由 `TheManGameplayTags.h/.cpp` 声明/定义，具体 Bullet 与 Cue Blueprint 只选择正式 Tag。
成功射击的震屏由 `UGA_Shoot` 读取当前枪械 `FireCameraShake/Scale` 并通过本地 `PlayerCameraManager` 播放，只负责短促打击感；随后 `AddRecoil` 才负责实际控制视角上抬。两者不得互相替代。
2026-09-04 临时测试配置：`UGA_Shoot` 保持 Camera Shake 播放，但以 `AFirearm::bEnableViewRecoil` 包住 `AddRecoil`；公共默认值当前为 `false`，因此三把枪不会推动 Controller Rotation，原后坐力参数未删除。
session251 起，玩家 `UGA_Reload` 由 `IA_Reload(R)` 经 Character 发送 `Input.Weapon.Reload` Gameplay Event 激活。Ability 属于当前枪械，使用独立 `ReloadAbilityClass/Handle` 随 Equip/Unequip 授予回收；当前为即时换弹，满弹或无备用弹夹时拒绝激活。
Phantom 的四个具体射击 Ability（Shoot1/Shoot2/Burst/Suppressive，FEAT-071）统一覆盖为 BaseSpread=3°、PerShot=0.8°、Max=9°、Recovery=2°/s、MovingPenalty=2°；其两种子弹 Damage=6。公共 C++ 默认值保持不变，避免无依据影响未来其他 Enemy。
| `GA_EnemyAutomaticFire` | 数据化 `ShotsPerActivation`/`ShotInterval`；同一 C++ 能力配置成三连发或持续扫射；每发消费 `UEnemyMagazineComponent` |
| `GA_EnemyReload` | 仅空匣可激活，延时/动画均可配置，完成后把弹匣补满 |
| `GA_EnemyTakeCover` | 调通用 `AEnemyCoverPoint::FindBestCover`，可选 RollMontage，移动到 StandPoint |
| `GA_EnemyAreaBarrage` | 不消费普通弹匣；在目标范围上方随机生成可替换弹体并向下轰炸 |
| `Source/TheManTest/Public/Enemy/BTTask_UseCombatSkill.h` | **通用敌人战斗放招 BT 节点**；UPROPERTY `Range`(近/中/远)+`TargetActorKey`；读黑板目标 → `AEnemyBase::UseRandomSkill`；不绑定具体技能 |

> **玩家技能授予时序（武器持有，ASC 在 PlayerState）：**
> - 角色 `BeginPlay` 时 ASC 尚未就绪，`Equip()` 中的 `GrantAbilities()` 会因 ASC 为空而跳过。
> - `PossessedBy()` 初始化 ASC 后，在末尾调用 `Firearm->GrantAbilities(ASC)` 补授。
> - 运行时切枪时，`Unequip()` 回收旧技能，`Equip()` 授予新技能（此时 ASC 已就绪）。
> - **切换角色的技能回收（重要）**：旧角色被 `Destroy` 前已被 `Possess` 转走 PlayerState，`GetAbilitySystemComponent()` 返回 null。`AFirearm` 在 `GrantAbilities` 时缓存 `GrantedASC`(`TWeakObjectPtr`)，`RevokeAbilities` 传入 null 时回退到它，确保技能一定回收，否则 `GA_Shoot` 规格泄漏累积 → 一次开火多颗子弹炸膛。

> **敌人技能系统（ASC 在敌人自身；技能集 = 阶段 × 近/中/远）：**
> - `AEnemyBase`：`PhaseSkillSets`（`TArray<FEnemyPhaseSkillSet>`，[0]=阶段1…；每项含 `Near/Mid/FarAbilities`）+ `CurrentPhase`(默认1) + `SetCombatPhase()`。
> - `BeginPlay` 经 `GrantAbilities()` 授予 `DefaultAbilities` + 所有阶段所有距离档技能。
> - `UseRandomSkill(Target, EEnemySkillRange)`：从随机起点轮询距离档能力；某能力因空匣等条件拒绝激活时继续尝试其他候选。
> - Phantom 阶段1：TakeCover/Burst/SuppressiveFire/Reload；阶段2保留全部并新增 AreaBarrage。用 `PhaseSkillSets` 数据注入，无需 Phantom 专属行为树子树。
> - 触发链：感知发现玩家→`SetAIState(Aim)` → BT 战斗序列 `BTTask_UseCombatSkill`(配 Range) → `UseRandomSkill`。详见 FEAT-032 archive。

> **新增 GAS 技能的标准流程：**
> - 玩家技能：声明触发 Tag（`Input.角色.技能名`）→ 继承 `UGameplayAbility` 用 `AbilityTriggers` 监听 → 角色 `DefaultAbilityClasses` 或武器 `PrimaryFireAbilityClass` 引用 → 编辑器建蓝图子类赋值。
> - 敌人射击技能：继承 `UGA_EnemyShoot`（仅数据不同建蓝图子类；逻辑不同重写 `SpawnProjectiles`）→ 放进敌人 `PhaseSkillSets` 的某距离档 → BT 用 `BTTask_UseCombatSkill`(Range 对应)触发。**不需要触发 Tag**（按类激活）。

### CoreMorph TailCharge／TailBlast Cue

`GA_CoreMorphTailStrike` 默认 2 秒蓄力，激活前投射 WorldStatic 地面并锁定落点／法线／BlastRadius；同一快照通过 GameplayCueParameters.Location／Normal／RawMagnitude 传递预警和雷爆。Tag 为 `GameplayCue.CoreMorph.TailCharge`、`GameplayCue.CoreMorph.TailBlast`，资产为对应完整名称 `GC_CoreMorph_TailCharge`／`GC_CoreMorph_TailBlast`；Native Static Cue 将生命周期交给拥有者 TailEffects 组件，不承载伤害。

尾尖扫掠在锁定地面附近首次触地时，GA 移除 Charge、添加 Blast，并在锁定中心做一次 Pawn／WorldDynamic 球形重叠；按 ASC 去重、检查静态遮挡，调用既有 `GE_CoreMorphTailDamage`（负 Data.Damage）。墙面阻挡不引爆。恢复或取消移除 Charge；取消同时移除 Blast；正常 Blast 1.25 秒后移除自身 Cue。头领终止生命周期兜底清理。阶段技能集及技能授予方式不变。

### 敌人强度伤害倍率：新增技能必接

用户在 2026-09-13 要求后续记住：新增／迁入 Enemy 伤害技能必须使用 `基础伤害 × Enemy->GetDamageMultiplier()`，只乘一次；阶段仅选技能集，波次／回合负责倍率。规则已写入 harness AGENTS.md。已有 GA_EnemyShoot／GA_EnemyAreaBarrage 在弹体生成时计算；CoreMorph TailStrike 现在于击地时读取倍率并保存本次雷爆伤害，所有范围目标共用同一数值，继续按 ASC 去重。不可修改 StrikeDamage 基础值造成后续出招累乘；Cue 不参与伤害。

### CoreMorph Manta MissileBarrage（2026-09-13）

第一阶段曾在 FarAbilities 配置 GA_CoreMorphMissileBarrage；用户当前先做 LobbyMap，已从 ACoreMorphBoss 的 PhaseSkillSets 移除装配。Near 保持 TailStrike，DefaultAbilities 保持 Flight／Reassemble；导弹 GA 及其 GE／Cue／组件／材质资产全部保留，待重新设计后再装配。其原有落点锁定、伤害倍率和范围去重实现仍保存在源码中。
GE_CoreMorphMissileDamage 通过负 Data.Damage 扣共享 Health，每次落地按 ASC 去重并检查静态遮挡。撞墙偏离预警区则消散，不在未预警位置引爆。复用 GE_CoreMorphAttacking；专属 MissileCooldown GE 授予 State.CoreMorph.MissileCooldown。
GameplayCue.CoreMorph.MissileBarrage 对应正式 GC_CoreMorph_MissileBarrage，仅管理专属视觉生命周期。取消／结束清理 Cue 和组件；手动变形先取消轰炸，死亡不再添加冷却。阶段、形态、强度继续独立，无新增声音／计时器。此节取代旧批次“共三个能力”的当前配置描述。

### CoreMorph EncounterEntry (FEAT-096)

`Enemy/Boss/CoreMorph/GAS/Abilities/GA_CoreMorphEncounterEntry` is a server-only InstancedPerActor ability granted in DefaultAbilities. It requires Manta and rejects transformation/death/duplicate entry. The optional beacon opening sequence cancels Flight/MissileBarrage, owns the FlightComponent source climb/dive completion delegate, then requests Reassemble; EndAbility unbinds/stops flight and clears EncounterEntryInProgress. It applies no damage and creates no second ASC. SpaceStation enables ClimbAndDiveOnActivation; the shared default remains false.


## 2026-09-17 Infiltrator Blink

FEAT-108: Characters/Infiltrator/GAS/Abilities/GA_InfiltratorBlink is InstancedPerActor/LocalOnly, triggered by Input.Infiltrator.Blink. Blueprint BGA_InfiltratorBlink exposes MaxBlinkDistance350cm. Horizontal current view direction with actor-forward fallback; scaled capsule sweep,1cm blocker gap, endpoint overlap checks, final swept instant move and floor refresh. Reject dead/wrong/unpossessed avatar; end synchronously and honor commit cancellation. No added cost/cooldown. Blink and physical BlinkKeyboard tests use only unsaved TestMap fixtures.

FEAT-114 supersedes instant movement: UAbilityTask_InfiltratorBlink under Characters/Infiltrator/GAS/Tasks moves over configurable BlinkDuration (default0.15s) along initial horizontal view direction, retaining350cm and LocalOnly/InstancedPerActor. Preflight capsule query plus each-step sweep catches obstacles entering during movement. No reentry; finish/cancel restores living avatar movement mode/velocity and floor, death never reenables movement. UnPossessed/EndPlay cancel before revoke. Ability drives its owner presentation component directly, consistent with Scan; no new cooldown/cost/tag/Cue. Actual DX12 cloak compilation and new-rifle restoration/HUD rendered checks passed.


## 2026-09-17 LaserRifle GAS ADS

FEAT-107: Weapons/LaserRifle/GAS/Abilities/GA_LaserRifleAim listens Input.Weapon.SecondaryFire and WaitGameplayEvent Input.Weapon.SecondaryFireReleased; caches activation character/weapon and restores on EndAbility. Per-weapon grant/revoke remains Firearm-owned. Controller pause sends release through PlayerState ASC, never calls a character method. GameplayCue.Weapon.LaserRifle.Impact uses GC_Weapon_LaserRifle_Impact with exact cold registry tag and explicit /Game/Weapons/LaserRifle/GAS/GameplayCues in DefaultGame.ini GameplayCueNotifyPaths. Missing path initially suppressed actual effects despite valid registry; cold physical shot/impact-decal test now passes.

## FEAT-110 Executive impact slow (2026-09-17)
Existing BGA_Shoot/BGA_Reload/input grants drive BP_ExecutiveSlowRifle. AExecutiveSlowRifleBullet runs base direct-hit once, then one300cm overlap, deduplicates enemy/ASC, excludes dead/pass-through/instigator and calls EnemyBase ApplyMovementSlow(.5,3). Repeat impact refreshes existing duration without stacking; no lingering zone or radial damage. Bullet destroys after impact. ExecutiveImpactSlow and ExecutiveEquipped cover physical input, actual projectile, refresh/expiry and custom boss locomotion.

FEAT-115: Weapons/ExecutiveSlowRifle owns ExecutiveRifleEnergyFX and GCN_ExecutiveRifleEnergy native classes. GC_Weapon_ExecutiveSlowRifle_Fire/Impact map exact native GameplayCue.Weapon.ExecutiveSlowRifle.Fire/Impact tags, with owner scan path registered in DefaultGame.ini. Fire VFX occurs only after real ammo consumption, not reload. Red tracer/muzzle and impact sparks with300cm sparse ground fragments fade over3s; these are collisionless cosmetics, never a persistent slow zone. Gameplay slot0 hides hologram; Lobby uses BP_ExecutiveRifle_LobbyVisual with original red material.


FEAT-135: native Ability.Movement asset tag identifies movement-owning abilities for God Mode cancellation. UGA_InfiltratorBlink declares this tag and blocks activation on State.Debug.GodMode; its usual movement restoration runs before God Mode captures and disables locomotion. God Mode does not identify concrete ability classes by name. Existing Q/E mappings return after debug context removal.

## FEAT-140 Executive deployment input - v0.8
UGA_TheExecutiveDeployDrone is LocalOnly/InstancedPerActor, triggered by Input.Character.Interact, blocked by State.Debug.GodMode. CanActivateAbility checks shared CanBeginPlacement: valid living local owner, available following companion, no existing placement. It rechecks before commit and synchronously begins focus; failed BeginPlacement ends cancelled. Component owns presentation lifetime. E Completed/Canceled restores; successful left-click dispatch ALSO restores immediately, even if E is held. Deployed/absent drones fail GAS activation before entering focus. No committed-hold exception, new GE/tag/cooldown/cost/input assets. Existing grant/revoke on possession retained. Normal weapons resume after auto-exit, confirming click consumed. No UI.

## FEAT-140 Executive turret GAS boundary
`Drone/GA_TheExecutiveTurretFire` is an instanced server-only ability on the turret's own ASC. The behavior tree requests it; Pawn readiness, aim, range, LOS and cadence must all pass before CommitAbility/ExecuteShot. The owner-local turret bullet reuses ABulletBase -> GE_BulletDamage -> Data.Damage and attributes hits to the player instigator, with10damage snapshotted per projectile. Player/other non-enemy Pawns receive no turret projectile damage.
`GameplayCue.Character.TheExecutive.DroneTurret.Reveal` and `.Fire` map exactly to `GC_Character_TheExecutive_DroneTurret_Reveal/Fire` under Drone/GAS/GameplayCues; DefaultGame.ini explicitly adds that scan path. Both use owner-local `GCN_TheExecutiveTurret`. Reveal initializes per-instance materials and UTheExecutiveTurretConstructionComponent (FEAT159); common reveal progress drives the original-source aerial laser/HoloGen construction and deployment animation. Combat becomes eligible at Active, independently of the drone returning to Following. Fire flashes the socket light; recoil and red projectile presentation remain synchronized with successful shots. Cue registry names must be cold-checked; merely checking Blueprint CDO tags does not prove dispatch.

FEAT160 Reveal Cue prepares construction presentation in Surveying state; laser-only ground survey completes before hologram/Niagara activation and reveal progress. Active-only combat gate covers both phases. Drone orbit and half-scale deployment do not change range, damage, fire cadence or ability dispatch.


### FEAT161 deployment feedback (2026-09-26; supersedes FEAT155/156 HUD behavior)

Hold-E placement creates only the temporary skeletal preview; deployment widget and ground ring are no longer instantiated. Legacy serialized HUD fields/assets remain compatibility-only. Existing focus/FOV/weapon visibility and E/LMB ability routing remain. Invalid blocking hits keep the preview visible; no surface hides it. Preview MID is blue, with eight blocking-component bounds/shape slots feeding a local red intersection mask in M_TheExecutive_TurretPlacementHologram. Static geometry uses signed global distance fields within blocker bounds; capsules/spheres/oriented boxes use analytic intersections. Material depth test is disabled so buried conflict pixels remain visible. Valid placement disables the mask. Settings require mesh distance fields (already enabled). Collision-only clearance failures without a mesh intersection do not recolor the entire turret.

Rejected confirmation starts UTheExecutivePlacementCameraShake with a real-time .22-second native pattern, .18-second retrigger cooldown; placement stays active. Cancel/end-play stops only this component's shake, clears preview MID/mesh, and restores prior focus/FOV/visibility. Blueprint/code callers share this rejection path.

ATheExecutiveDrone::GetTurretPlacementTransform selects the nearest living, visible, targetable AEnemyBase by squared distance from the ground point; excludes destroying and projectile-pass-through enemies. With no eligible enemy it uses player aim yaw. Preview updates live; DeployAt captures the same transform for travel/build, preserving confirmed yaw even if the player turns or target moves. The mesh's existing -90-degree import correction is preserved. FEAT160 half scale, red survey/construction, low orbit, return and retained turret remain.


### FEAT162 (2026-09-26): scan audio and whole invalid preview

User supersedes FEAT161 local-red requirement: invalid blocking-hit preview is entirely red, valid is blue. The material validity Custom node takes only Original RGBA/ConflictEnabled and preserves original fresnel/stripes/opacity. Native per-blocker overlap/parameter uploads removed; unused old graph nodes are not connected to output. No material distance-field dependency for validity. No-hit still hides preview.

Deployment component RejectionSound points to SCue_TheExecutive_DeploymentRejected (specified249205-Game_Error_4); on rejected LMB plays a single2D voice together with existing shake, shares real-time .18s throttle and preserves placement. Cancel fades it .06s; owned audio stops on owner destruction.

Construction component ScanStartSound/ScanLoopSound are Blueprint-configured Cue references in Drone/Audio. At first active laser frame, plays a .34s attack from100835-Laser_scan_07-SFXBible-ss05993; at .24s real time starts a .16s seamless looping segment with .10s fade-in. Both sources track drone laser emitter in3D, stable pitch and no per-call randomness. Loop spans survey and construction while laser exists, completion fades .18s, EndPlay immediately stops remaining voices. UI rejection/scan recognition/long-loop fixed-pitch exceptions recorded in audio policy. No scan gameplay duration changes.


## FEAT163 scan visual replacement
UGA_InfiltratorScan retains its existing input/toggle/audio and ScanEffect call path. Infiltrator ScanPresentation consumes native start/stop delegates, replaces old wave rendering with accepted TMIIR07 and manages range-bound sampled-colour enemy smoke. No source input/camera shake/sound/hologram imported. Original opt-in IHighlightable/HighlightComponent semantics remain; enemies receive smoke only unless explicitly authored as highlightable. Character unpossess retracts all scan presentation.


## FEAT180 single scan owner (supersedes legacy scan layout)
FEAT180 completed2026-09-26. One InfiltratorScanPresentationComponent now owns scan state,2s expansion,3000cm radius,10Hz opt-in highlight detection/cleanup, source grid/return/audio and disabled smoke. Existing native class/subobject names intentionally retained to preserve BP settings. Removed generic ScanEffectComponent h/cpp and base attachment/getter, merged needed logic; eliminated legacy MPC driver and optional unused GAS hologram/sound/state code. GAS verifies Infiltrator owner and only toggles the component. Base tracks only its default-ability grant handles and cancels/clears them on re-possession/unpossess/EndPlay; weapon grants remain equipment-owned. Worker Scan grant removed; Executive unchanged empty config; Infil settings copied from captured old component. Three role BPs and Scan BGA compiled/saved. Removed only old Contours blendable from TestMap and SpaceStation PP actors, retained M_Infiltrator_Highlight. Deleted M_Infiltrator_ScanContours, M_Infiltrator_ScanTerrainOverlay, MPC_Infiltrator_ScanEffect,T_Infiltrator_Scan_MatCap after refreshed AssetRegistry proved no outside referencers; empty Scan/Textures removed. Disabled smoke and previously retained hologramUI assets preserved.


## FEAT198 current Infiltrator blink
FEAT198 completed2026-09-27: user-approved screenshot192420 blink filter plus600cm distance/.26s duration (previous350/.15); existing.28s tail retained. Native/BGA defaults synchronized, swept movement/collision and grant lifecycle retained. Existing BlinkPresentation now owns transient unbound priority110 PP + ScreenFilterMID only during blink/tail; progress ramps strength over first18%, tail smooth-fades from attained strength, cancel/unpossess/death/EndPlay/restart clean up; pause hides PP and resumes. New M/MI_Infiltrator_BlinkFilter under Blink/Materials run AfterTonemapping priority20:16 weighted radial samples, protected centre, blur.24, saturation.18, violet highlights.65/threshold.35 and Purple(.62,.32,1), neutral parent/instance Strength0 driven by native MID. Highlight tint avoids luminance renormalization clipping discovered in first visual pass. Blink energy/cloak runtime color(.4,.12,1) aligned to violet. BP and ability opened/compiled/saved. Development Editor Win64 cold build28.53s Succeeded with no compiler warnings. Existing Blink and BlinkKeyboard tests Success, covering600cm/.26 midpoint, physicalQ/repeat/input, collision/blocked/gap, cancellation/unpossess, gun restore and filter cleanup. Actual unsaved TestMap corridor18checks pass with real GAS600cm, during/arrival/restored screenshots, scan coexistence and pause/resume; first Python probe failed only on inaccessible private PP field and was corrected by inspecting owned PP blendables, final run passed. Independent cold21checks confirm refs/params/shader/dependencies. Two new material assets, two existing assets (BP and BGA), six native/test files. All50other existing ability packages SHA256 preserved, including47scan assets. PIE ended, dirtycontent/maps0; editor on secondary. No map changes, push/package/version change or performance benchmark. Checkpoint b197bee4; evidence D:/UnrealWork/BlinkViolet. Final saved changes uncommitted; user aesthetic acceptance pending.


## FEAT213 current Dream Blink V2
FEAT213 completed2026-09-28. New owner-local BlinkV2 folder contains M/MI_Infiltrator_DreamBlink and BGA_Infiltrator_DreamBlink. BP_Infiltrator now grants V2 on existing Q; original36V1packages unchanged. Rollback: replace DreamBlink class in BP DefaultAbilityClasses with original BGA_Infiltrator_Blink; scan class unchanged.

UGA_InfiltratorDreamBlink derives existing blink, using virtual AdvanceBlink/BeginPresentation hooks. Reuses original activation/commit,PlayerStateASC,collision query,movement ownership and cleanup. Holds origin until85ms, rechecks whole live capsule path, performs exactly one600cm maximum TeleportPhysics landing; moving obstacles shorten/block.45ms radial image pull (2-3frames at60Hz), black/purple multiscale cloud/grain occlusion around teleport, three displaced screen-space scene copies fade over150ms; fixed total235ms. This is screen-space displaced current-scene imagery, not captured historical frames or world geometry deformation. Timing quantizes to game frames; normal60Hz evidence holds0cm through83.3ms and lands600cm at100ms, no intermediate traversal, filter gone by250ms. No persistent tint/exposure changes.

Common presentation minimal mode retains owned accepted audio and existing HUD, skips V1 filter,phase gun/arms,smear and Flash trails; equipment changes also respect minimal mode. V2 owns its PP/MID and destroys both on success/cancel/death/unpossess. Sound may finish its existing1.577s natural tail after success, stops on cancellation. Original weapon/arm materials preserved. Prior FEAT212 six independent cold-white/violet strokes remain3:2:1 inward aligned with no connectors; bottom HUD unchanged. No animation/retargeting, map saves, packaging,version increment or push.

Validation: cold Development Editor Win64 build27.72s Succeeded. Live ability and character BP opened/compiled/saved. Native Blink(V1 transient fixture),BlinkKeyboard(productionV2 Q),BlinkV2 all Success. V2 tests cover no motion during pull/noise,one600cm teleport,235ms completion,no legacyPP/trails,audio,cancel before/after teleport,dynamic obstacle and unpossess cleanup. Final unsavedTestMap review passed520 checks (mostly per-frame position checks) plus originalmaterials,actualaudio,teardown,normal-speed600cm and active scan preservation. Pull/Noise/Arrival/Echo/Restored screenshots inspected. No performance benchmark.

Independent read-only cold asset verification passed exactHLSL,parent/location,two scalar parameters through shader/defaults,600/.235 settings,productiongrant and oldV1 settings,3uniquepackages/noRedirectors. Initial verifier erroneously counted BlueprintGeneratedClass as a fourth package; corrected to unique package names. Cold loading marks unrelated existing BB_CoreMorphBoss dirty in memory; recorded and discarded without saving. Live cleanup is authoritative for editor dirty state. Initial noise looked gridlike; replaced with smooth3octave cloud noise and fine grain,final PIE rerun passed.

Evidence D:/UnrealWork/BlinkV2. Checkpoint42e015a2 preserves completed FEAT212 and priorV1. Final V2 results saved/uncommitted per harness. User explicitly authorized graceful computer shutdown after completion; do not force-discard unrelated unsaved work.


## FEAT214 current Portal Blink V3 (2026-09-28)
Production BP_Infiltrator grants BGA_Infiltrator_PortalBlink on the existing Q Gameplay Event. Assets live exclusively under Characters/Infiltrator/Gameplay/Abilities/BlinkV3; old Blink/BlinkV2 and Scan packages are preserved. Native classes live in matching Public/Private Characters/Infiltrator/GAS/Abilities/BlinkV3.

UGA_InfiltratorPortalBlink inherits the trigger/avatar guards from Blink but invokes UGameplayAbility activation/end directly: it never locks movement or invokes the V1/V2 transition. Its ticking ability task owns AInfiltratorPortal and destroys it on cancellation, death/unpossess or completion. A grounded activation places an entry220cm ahead and an exit1000cm ahead of the activation capsule origin. A walkable floor and clear capsule are required at both; only the short approach is swept, so the portal may bridge intervening walls. Exit safety is rechecked on crossing. The actual player camera crossing the entry plane triggers one exact exit-minus-entry translation, preserving lateral offset, velocity and view. The camera is ~30cm ahead of the capsule; the capsule need not already be past the exit plane at this moment. Conservative central aperture bounds prevent crossing outside the opening.

Opening.4s, hold5s, closing.35s; one use or timeout. Source FX1-derived Niagara particles and original jagged mask/noise are owner-local, blue-violet. Demo galaxy/grid/helper renderers use a transparent owner-local material and are not displayed. Source particle activation waits until the aperture is mostly open so its world-space spawn positions use a valid scale. The portal surface samples a scene-capture RT using screen UVs; capture uses translated camera and actual local-player projection plus exit clip plane. A second lightweight show-only scene-depth capture of visible arms/weapon, UInfiltratorPortalForegroundCapture, uses the player's view owner and the same projection to preserve first-person occlusion without custom-depth/stencil flags, weapon material replacement or unwanted outlines. It refreshes attached meshes for equipment changes. Both RTs are transient and released on EndPlay; no permanent map actors. Display is pushed slightly beyond the main near plane only within the last12cm before entry to avoid near-plane disappearance. Existing accepted blink sound/HUD plays on successful traversal. No source animations/retarget assets are imported.

Rollback: replace only the PortalBlink class in BP_Infiltrator.DefaultAbilityClasses with the retained DreamBlink(V2) or Blink(V1) BGA; keep Scan. Native Blink/BlinkV2 tests use explicit transient legacy grants, while BlinkKeyboard checks current production Q portal behavior. Evidence and final verification: archive/FEAT-214-portal-blink.md and D:/UnrealWork/BlinkV3.


### FEAT214 follow-up: terrain and single-frame visibility (2026-09-28)
Supersedes exact-height placement above: portal uses downward full-capsule sweeps on the character movement channel and component walkability, allows up150/down250cm and searches60/120cm rings around the10m target. Entry follows floor in27.5cm samples with character MaxStepHeight, preferring220cm and shortening only as far as110cm. Exit validation accounts for the camera-to-capsule planar offset and +/-12cm walk margin, so the actual body landing is validated instead of just the exit plane. Crossing rechecks/snap-resolves terrain height; only flat ground guarantees exact initial Offset translation.
Both scene/depth captures now use InfiltratorPortalForegroundCapture for player view-owner visibility; exit exclusions refresh every frame. Initialize capture and OpenAmount0 before displaying surface. Traversal in PostUpdateWork translates both current/previous camera caches in the same frame (fixes measured780cm lag), marks camera cut and hides the stale surface immediately; rim still closes. No character materials/shadow settings change. Final actual SpaceStation12direction opening and walking traversal, native three tests and flat fixture68checks passed. Opening full-body silhouette was not independently reproduced; final21-frame sequence shows no human in portal centre, but do not assert the original opening cause was proven. See FEAT214 archive.

## FEAT218 horizontal pointed Portal V3 (2026-09-28)
Supersedes FEAT214 visual implementation: NS_Infiltrator_PortalHorizontalBorder rebuilt from original5.1 P_Portal_2 and UE58Blank FEAT065 exact28-point outline.hlsl. Three new owner-local assets under BlinkV3: system plus M_Infiltrator_PortalHorizontalBorder and M_Infiltrator_PortalHorizontalView. Original two noise textures and flowing violet emission retained. No UE58 binary asset copied into UE57.
AInfiltratorPortal now uses one Niagara mesh for both live view and border, with runtime MID bound through renderer User.PortalMaterial. Native explicitly adds the material parameter to component OverrideParameters before SetVariableMaterial, so binding works outside editor. Old PortalSurface component removed; capture textures/foreground depth exclusion remain owned by portal. View material replaces source opaque black interior with exit capture, preserving original radial noise/warp/rim and private foreground cutout. Border-only material is the asset preview fallback.
Source mesh is500cm at full particle scale, native1200cm square sheet gives approximately11m horizontal tip span and3m peak-to-peak height. This is gameplay sizing, not source preview actor scale6. Plane normal is localX, so yaw matches portal forward; native animation scales localY width andZheight. System warmup119ticks at1/60 (~1.983s) skips original2s mesh growth; current.4s opening/5s hold/.35s close are authoritative. Display/near-plane guard/crossing visibility now apply to Niagara. On successful crossing unified view/rim hide immediately to avoid stale camera image; timeout still visibly closes.10m placement, camera-cache translation, clipping, audio, cancel/cleanup and legacy abilities unchanged. Old BlinkV3 visual assets retained inactive for rollback.


## FEAT222 deployment XRay preview (2026-09-29)
BP_TheExecutive.DroneDeployment.PreviewHologramMaterial overrides the native old placement fallback with `/Game/Characters/TheExecutive/Gameplay/Drone/Deployment/Materials/M_TheExecutive_DeploymentXRay`. Source is current enemy ScanGoldXRay Fresnel/occlusion shader, independently duplicated for deployment; valid blue HologramColor and invalid red InvalidColor selected by existing ConflictEnabled. Exponent3.5,opacity.8,glow3,bias2/feather3 match source. No preview custom-depth writer: enemy-specific inner cull is disconnected. Existing input/focus/preview destruction/confirmation remain; old material retained as native fallback. Main42PIE checks and cold62checks pass. See FEAT222 archive.

## FEAT225 current-pawn debug blink selection (2026-09-30)
AFPSInfiltrator exposes SetDebugBlinkVariant(1=Dash,2=Dream,3=Portal) and GetDebugBlinkVariant derived from the actual tracked grant. DebugBlinkClasses contains3soft references to retained production BGA assets. Validate/load before revoking old handle; cancellation and BlinkPresentation.Shutdown clear movement/effects, then new grant is tracked by existing BlinkAbilityHandle/BlinkGrantedASC for UnPossessed/EndPlay cleanup. Scan/weapon/other grants untouched. Same selection no-op, invalid/noncontrolled/non-authority rejected, Shipping setterfalse. Does not alter DefaultAbilityClasses/save/loadout; replacement pawn starts with configured Portal default. Existing Q event trigger unchanged.

## FEAT226 layered portal border (2026-09-30)
The two existing M_Infiltrator_PortalHorizontalBorder/View materials now share one signed-distance shader over the original28vertex horizontal silhouette. Three panning samples of existing engine TilingNoise05 perturb the contour and form fine core, violet plasma and feathered outer wisps; derivative AA and uniform plane-space width replace the old radial threshold ring. BorderWidth=.006 (normalized doubled UV), CoreGlow=1.15, FlowSpeed=1.0. Both graphs use the same shader prefix, maintained together; preview fallback omits scene composition. Runtime keeps PortalView/ForegroundDepth/OpenAmount parameter names, combines scene*aperture with rim radiance and uses common opacity, foreground depth exclusion and10cm intersection fade. Existing Niagara User.PortalMaterial binding, shape dimensions, timings and all native traversal/capture behavior unchanged. No texture imports or additional scene capture. Archive FEAT226 / evidence D:/UnrealWork/PortalBorder.

## FEAT227 portal width and highlight contrast (2026-09-30)
Supersedes FEAT226 tuning: BorderWidth=.0102 (1.7x), CoreGlow1.15/FlowSpeed1 unchanged. Shared SDF also derives cumulative perimeter coordinate. Two moving periodic waves(5/9cycles) create local highlight weights, which drive violet-to-lavender core color/energy(.28..1.30), body/wisp brightness and width(.80..1.22). Texture body attenuates captured scene only at the rim(up to55%) to retain a dark violet shoulder. Same28point silhouette, three noise samples and runtime parameters/captures/traversal. Two existing material packages only. See archive/FEAT-227-portal-highlights.md; evidence D:/UnrealWork/PortalHighlights.

## FEAT228 localized portal dream presentation (2026-09-30)
Two existing horizontal portal materials retain FEAT227 width/highlights and add noise-led fold shading plus sparse inward-moving planar mote dots using the same contour. Runtime View material adds DreamFilter=.15 (desaturation and shadow-weighted cool tint), RefractionStrength=.006 (screenUV amplitude), GhostStrength=.24 (two weighted offset exit samples). Shared MoteStrength=1. InnerBand fades refraction/echoes from signed distance.012 to.070 (about7..42cm on fully open1200cm sheet); central view has no spatial warp. Three exit samples replace the original one; no extra capture or global postprocess. Existing OpenAmount scales onset/close, foreground depth gate preserves gun/arms, native crossing immediately hides owned surface. Fold shading/motes are on existing plane, not3D geometry/Niagara simulation. Same silhouette/view offsets/gameplay and all other assets. Evidence D:/UnrealWork/PortalDream / archiveFEAT228.


## FEAT229 staged portal opening (2026-09-30)
Supersedes FEAT218 opening scale animation: native OpenDuration=.6s and OpeningAge MID scalar (initialized0, clamped.6) drive shared border/view contour. Keep full mesh scale during opening; close still uses existing.35s shrink. Shader reveals a center-out slit during first.12s, locally deforms28polygon vertices with upper/lower directional delays during.12-.4s, then restrained damped overshoot/highlight sweep and inward motes until.6s. At.6 original full contour and steady FEAT228 styling restored. OpeningAge default.6 permits fallback material preview; live actor explicitly initializes0. Existing OpenAmount retains monotonic readiness/filter/close behavior; initial slit opacity uses independent reveal so it is visible immediately. Traversal explicitly requires Age>=OpenDuration; five-second hold begins after opening. ScreenUV exit sampling, foreground protection, camera ownership and cleanup remain unchanged. No new assets/captures/global postprocess. Evidence D:/UnrealWork/PortalOpening; archiveFEAT229.


## FEAT230 independent Rift portal (2026-09-30)
UGA_InfiltratorPortalBlink has configurable PortalActorClass, default AInfiltratorPortal; Initialize is virtual. Existing V3 retains its material/system and VisualHeight25. New AInfiltratorRiftPortal (matching BlinkV4 native folders) inherits placement, RT/foreground capture, camera crossing, velocity preservation and cleanup, adding RiftEnergy Niagara. BP_Infiltrator_RiftPortal supplies source effect, EffectScale.45 (2026-09-30 user correction: size the core like the old portal, not the outer trails; multiplied by initial planar view distance/190cm), VisualHeight55 and yaw+90 (source camera faces positiveY). Default BGA is now RiftBlink; Dev Panel offers Dash1/Dream2/Portal3/Rift4 with existing Q input.
BlinkV4 owns thirteen packages: ten renamed source core assets, Rift actor BP, independent RiftView material and RiftBlink BGA. Source meshes/textures and other materials remain unchanged after rename; RiftMark now composites the live capture inside its original opacity silhouette. System gains local-space emitters plus one engine ApplyOwnerScaleToAttributes update module per emitter (sprite/ribbon/camera true; local mesh/motion false). Explicit editor PrepareForUniformScale is path-guarded/idempotent and recompiles; never called at runtime. No NiagaraFluids, vendor/demo folders, extra audio, maps or retarget assets imported.
RiftView reuses V3 sheet geometry through per-component User.PortalMaterial. Live capture reveals only while approaching from70% to20% of initial view distance via ViewReveal, retaining original violet/gold vortex at activation distance and removing the extra gold circle. Original.6s readiness,5s hold,.35s close and safety checks retained. Source FX hides immediately on crossing and deactivates on cancel/timeout. No global postprocess. Evidence D:/UnrealWork/BlinkV4; archiveFEAT230.

FEAT230 lifecycle followup: Rift actor CoreMaterial points to M_Infiltrator_RiftMark, with per-instance CoreMID passed through User.RiftCore (sprite renderer SortOrderHint100). Existing RT/foreground textures reused; original gold channel selects the rim, centre samples PortalView with a brief refraction pulse. Reveal completes by.34s without approach; old inherited overlay is hidden. Niagara local-space owner scale drives charge.09s, expansion to.44s,4.5% settling until.6s; timeout collapse lasts.35s with105degree inward spin and terminal core fade. Full BP/native scale.45 retained. Successful traversal/cancel still immediately cleans source/captures; no second exit actor. No extra textures/captures/global postprocess. Prior proximity-only reveal paragraph is superseded. Evidence lifecycle-review.json60 checks, lifecycle-capture.json94 frames and Rift_Opening_Closing.mp4.

FEAT230 palette followup: original warm rim mask still defines the live scene boundary, but rim emission is now violet (.6,.08,1.2)*source peak, preserving the untinted captured scene. New owner-local M_Infiltrator_RiftSpark duplicates the engine default sprite material and remaps warm spark emission to violet; four Rift sprite renderers reference it. Engine material unchanged; BlinkV4 now owns14unique packages. Size/timing/native code unchanged.

FEAT230 layered violet followup: RiftMark keeps the original aperture mask but replaces flat peak-based rim tint with capped emission, deep-violet base, five-lobed angular flow,21-frequency counterflow detail and sparse pale crest highlights driven by Time. Warm-hue conversion is independent of the aperture mask to avoid orange transition fringes. RiftSpark output halved for rim/particle brightness separation. Live scene and lifecycle unchanged; two materials only.


## FEAT231 Direct blink (2026-09-30)
UGA_InfiltratorDirectBlink lives in Characters/Infiltrator/GAS/Abilities/BlinkV5 and derives from Blink for shared trigger/avatar validation/tick task only. Activation/End deliberately call UGameplayAbility, bypassing V1 movement/presentation ownership. Grounded current safe capsule required; horizontal view direction1000cm plus local60/120cm eight-direction alternatives use InfiltratorPortal.ResolveDestination floor/capsule validation. Unsafe destinations report blocked and do not move or create presentation. Accepted activation teleports immediately, preserves velocity, forces floor refresh and translates both camera caches in the same call.
Transient PostProcessComponent owns M_Infiltrator_DirectEcho MID, DirectAge0..0.25: three displaced scene samples with staggered fade, short radial contraction and mild defocus; scene depth protects near weapon/arms. This is screen-space reprojection, not persistent world mesh copies or historical frame capture. Before/after samples protect near geometry; HUD draws after the pass. No persistent scene filter.
Transient SoundMix ducks configured Effects/Music classes to.001, fadein.005. At.1s Pop with.025 fadeout, then BlinkPresentation.BeginBlink/EndBlink reuses existing Cue. Normal visual cleanup at.25s; cancellation, invalid owner/unpossess and EndPlay release owned mix/component/MID. User volume configuration unchanged. Audio device scheduling adds measured~25ms to audible recovery.
Assets: /Game/Characters/Infiltrator/Gameplay/Abilities/BlinkV5/GAS/Abilities/BGA_Infiltrator_DirectBlink and BlinkV5/Materials/M_Infiltrator_DirectEcho. BP_Infiltrator default5; four prior variants retained. Evidence D:/UnrealWork/BlinkV5 includes successful Editor build,42PIE checks, cold assets, real master recordings and SpaceStation59frame capture.


FEAT231 astral-pull followup (2026-09-30) supersedes the immediate teleport above. FindDestination first uses Blink.FindBlinkDestination for path-limited capsule clearance, rejects less than60cm, then resolves destination floor/capsule. No position/cache teleport on activation. Save/disable normal movement; first40ms travel2.5%, then smooth accelerated catch-up to250ms with SetActorLocation swept every tick. Live blocking hit stops further progress and projects saved velocity off hit normal. Super(Blink).EndAbility now restores owned movement with existing death/external-mode guards and floor refresh. Finish350ms; cancellation/unpossess releases all ownership.
UInfiltratorPullCameraModifier (same BlinkV5 header/cpp) is created per cast and removed by the ability; Additive POV lead up to9cm/1.5cm high,7deg FOV and1.1deg pitch during separation, small damped100ms landing rebound. Camera lead sphere-swept on Camera channel; no component transform/authored FOV changes, so attachments briefly lag the viewpoint and ADS base remains intact. Modifier Age follows ability time.
M_Infiltrator_DirectEcho now runs0..350ms: three environment echoes and slight defocus throughout movement, plus a separately depth-gated shifted near-field sample tinted violet for hand/weapon residue. No cloned mesh, new skeleton, persistent material replacement or HUD postprocess. Same100ms sound release/mix/Cue policy as before. Evidence D:/UnrealWork/BlinkPull, review.json29checks and capture.json.


FEAT231 ZhongKui-reference mist followup: native/BGA maximum1400cm, first40ms2.5% then280ms catch-up, total movement320ms and final100ms camera settle (420ms ability lifetime). Camera lead fall/rebound and DirectAge material lifespan synchronized. Existing M_Infiltrator_DirectEcho adds3peripheral radial samples for a25..170ms spatial impulse, central radius mask, outward-moving procedural grey-violet wisps and12sparse dark chips. These are view-local shader layers, not Niagara or world-persistent particles. Near-depth protection reduces overlays on actual gun/arms; existing violet offset residue retained. Effects fade by420ms, no persistent scene filter.100ms audio release and collision/cancel cleanup unchanged. Evidence D:/UnrealWork/BlinkMist;29actual PIE checks verify1400cm and320ms with static/dynamic collision and restoration.


FEAT231 fracture-tunnel followup supersedes the swept-path and mist behavior above. User explicitly requests crossing intermediate walls. FindDestination requires a safe grounded start, then ResolveLanding probes1400cm forward and local60/120cm ring alternatives using existing portal floor/capsule checks. Advance uses continuous unswept SetActorLocation (not activation teleport); normal movement remains ability-owned/disabled. Each safe intermediate standing location updates LastSafeLocation. Arrival revalidates the desired landing and falls back if newly occupied. EndAbility also checks the actual capsule and resolves last-safe/start vicinity before releasing movement, covering cancellation/death/unpossess while inside walls. Capsule collision settings are not globally disabled or edited.
Same owner PP material now renders four perspective-scaled, noisy fragmented violet boundaries and depth-projected irregular dark/refractive plates with sparse edge glints. Near-hand quadrant/depth mask protects weapon, core stays visibly open; temporary desaturation/cool violet grading and longitudinal tears only during420ms ability. It is a view-space procedural tunnel, not world geometry, Niagara or purchased tutorial content. Prior thin mist and tiny chips removed. Studio Cloe Tunnel Effect and CGHOW Magical Portal inform layering/polar dissolve/edge contrast; screenshot2026-09-30143838 is user visual target. Evidence D:/UnrealWork/BlinkFracture;35PIE checks plus final visual/cold reports.

FEAT231 clean followup: user removed current fracture/tint effect. Existing M_Infiltrator_DirectEcho now returns Scene.rgb (identity); material reference remains because native activation requires it. No tunnel, shards, mist, echoes or grading. Native14m continuous phase pull, camera lead/recoil,100ms audio and safety cleanup unchanged. DevPanel still five variants; visual iterations are not separate variants.

FEAT231 approved subtle soul followup supersedes identity-only material: two neutral RGB displaced scene samples at screen periphery (centre radius protected, source/sample depth masks exclude foreground). Rise0..60ms, displacement shrinks during travel, final reunification320..400ms; identity from400ms. No tint, shards, tunnel, cloned spirit or historical frame capture. Camera lead now12cm at60ms, fades by280ms; rebound320..400ms.14m/320ms phasing and100ms audio unchanged.

FEAT231 nightmare awakening supersedes soul separation. Direct5 moves2.5% over80ms then smooth catch-up over240ms, remains14m continuous unswept safe-endpoint travel. Camera3cm lead/1.5cm dip, pitch-2.2deg and4deg FOV during dream;320..400ms head-lift/recoil; authored FOV/attachments intact. Existing material replaces two echoes with neutral peripheral multi-tap defocus/radial drag and partial edge darkening (never full black), clarity returns320..345ms. World duck fadein25ms, release at travel completion320ms with15ms fadeout and existing Blink Cue; previous100ms audio timing obsolete. No new audio asset or human gasp. Ability cleanup420ms and cancellation safety retained.


FEAT231 eyelid blink is the current Direct5 implementation. Activation validates safe start/landing and owns disabled movement without moving. Advance holds origin until75ms, revalidates landing, then makes one unswept TeleportPhysics relocation and flags a camera cut to discard stale temporal history. If a frame crosses75ms, presentation age clamps there for one fully closed frame, including low FPS; subsequent ticks resume normally. Newly blocked landing aborts at origin. Existing EndAbility safe-position recovery and base movement ownership restoration remain. Curved asymmetric upper/lower black eyelids close0..60ms, stay closed through110ms, reopen110..240ms; neutral4-tap blur clears by280ms. No historical-frame residue or displaced scene echoes. No active camera modifier; native legacy class retained with no-op ModifyCamera. Existing SoundMix/Cue release once when reopening starts at110ms; full cleanup300ms. Authored FOV, arms materials, five variants and14m unchanged.


FEAT231 latest brightness correction: User correction implemented: Direct5 uniform full-screen light/dark pulse replaces visible upper/lower eyelids. RGB gain1->.04 over0..60ms, holds through110ms, recovers by240ms; neutral brief refocus clears280ms. Existing75ms jump,110ms audio and300ms cleanup unchanged. No spatial aperture/eyelid masking remains. Earlier native timing names/comments refer to the retained timing phases; no literal eyelid is rendered.


FEAT231 final confirmed cinematic disorientation supersedes all brightness/eyelid/nightmare-dimming versions. Confirmed cinematic disorientation: normal brightness, radial stretch/asymmetric lens flex, edge RGB separation and mild double vision/defocus. No brightness pulse, tint, desaturation, vignette or eyelids. True departure-environment snapshot max30% overlaps live destination after75ms jump, residue fades350ms/distortion400ms/cleanup420ms.14m and110ms existing audio retained; tinnitus describes visuals, no new sound.
Direct5 owns transient DepartureFrame RGBA16f RT (width cap1280/aspect matched,height cap1280) and unattached one-shot SceneCapture2D at source cached camera/custom projection. FinalToneCurveHDR capture occurs before owner PP; player+attached equipment hidden, no movement/every-frame capture, no motion blur. Capture stores linear tone-mapped RGB, shader explicitly converts it to SDR sRGB before mixing with after-tonemap live scene. HasDeparture defaults0/runtime1; residue starts75ms, depth-gated environment, max30%, no cloned gun/body. EndAbility clears MID binding, destroys capture, drops RT reference on all cleanup paths. Shader RGB-separated main sample .72 plus two ghost taps .14 each preserves constant-field color; no gain modulation. A legacy inactive camera-modifier class remains for compatibility. No exact-backbuffer or performance claim; capture is a separately rendered environment view and adds one activation render.


FEAT231 dynamic pull and shake now supersede the instantaneous cinematic version. Dynamic pull restored on user request: continuous14m/320ms phasing (first40ms2.5%, accelerated smooth catch-up),12cm view lead while hands/body follow,5deg FOV pulse, bounded damped pitch/yaw/roll+sub-cm translation shake,80ms arrival rebound. Existing normal-brightness stretch/warp/RGB split/departure snapshot shader unchanged; all effects settle400ms/cleanup420ms. Existing audio releases at320ms arrival.
UGA_InfiltratorDirectBlink again advances the physical pawn continuously via unswept SetActorLocation; only safe endpoint is required, with LastSafeLocation updated at safe intermediate positions and revalidated at320ms. Dynamic-blocker fallback and existing EndAbility safe rollback remain. UInfiltratorPullCameraModifier is created after departure snapshot and removed on all cleanup paths. Lead ramps0..55ms, fades70..300ms; camera offset sphere-swept on Camera channel. Shake smooth attack25ms, release140..320ms; bounded components use deterministic67/83/59rad/sec sinusoids so onset/end have zero envelope and no accumulated drift. Pitch amplitude.8deg/yaw.5/roll.45 plus pull/rebound; local Y.45cm/Z.35cm shake. Landing320..400ms damped recoil. Camera attachments, arms transforms, authored FOV and all materials unchanged. Residue still starts75ms during pull and fades350ms, now overlaps continuously moving live view. Sound duck release follows actual320ms travel completion, not prior110ms.


## FEAT232 Medusa nightmare
UGA_MedusaNightmare (Enemy/Nightmare/Medusa/GAS) is InstancedPerActor/ServerOnly, granted once by AMedusa.BeginPlay using BGA_Medusa_Nightmare. Native proximity requests ActivateNightmare; ability calls BeginNightmare, owns a game-time duration timer and ends presentation through EndAbility on timeout/cancel. No damage/attack or new input binding. Single-player runtime scene state is coordinated by UMedusaNightmareSubsystem. Existing projectile alert callback triggers hit reveal before damage; original bullet and scan assets remain intact.


## FEAT235 Medusa combat (implementation in progress)
GA_MedusaCombat handles Spell/Dash/Melee through owner combat component; editable BGA subclasses populate existing Near/Mid/Far phase pools. GA_MedusaNightmare lives in BackgroundAbilities and uses random owner cooldown independently. Common BTTask_UseCombatSkill waits/cancels its own spec only, with configurable recovery(default3s, Medusa.15s). Direct damage samples EnemyBase.GetDamageMultiplier once at settlement. See FEAT235 archive for validation status.


FEAT238: darkness skill renamed Dark Veil: GA_MedusaDarkVeil and BGA_Medusa_DarkVeil, scene ownership MedusaDarkVeilSubsystem. Medusa DarkVeilAbility/Duration/Cooldown/IntervalMax and activation functions use new names. Exact CoreRedirects preserve old serialized values. Nightmare is only the enemy family. Background phase pool still triggers darkness independently of Near/Mid/Far combat; introduction blocks activation while it owns presentation.

## FEAT247 Void Gazer wall leap
UGA_VoidWallLeap / AbilityTask_VoidWallLeap live under Enemy/Nightmare/VoidGazer/GAS. BGA_VoidGazer_WallLeap is granted by BP_VoidGazer.DefaultAbilities; BT activates it explicitly after the component has selected a valid landing plan. No player input tag, damage or phase skill is involved. GAS owns the entire windup/flight/landing lifetime and abort cleanup; see15-enemy-movement.md for geometry and ownership boundaries.


FEAT248: BGA_VoidGazer_Melee derives UGA_VoidMelee; one ability randomly chooses from four Animations and owns execution/cooldown through VoidCombatComponent. BGA_VoidGazer_GroundSlam reuses UGA_VoidWallLeap with GroundOnly=true. Both use PhaseSkillSets; only patrol WallLeap remains in DefaultAbilities. UGE_VoidDamage applies negative Data.Damage to player Health with enemy damage multiplier once. Ground impact hits each living player once inside locked radius/height and LOS. All timers/warnings clean up on cancel/death/endplay.


## FEAT257 Void tentacle GAS integration (verified 2026-10-07)
Supersedes the earlier presentation-only scheduling boundary. GA_VoidTentacle is InstancedPerActor/ServerOnly and owns the existing component from windup through retraction; cancellation calls CancelPresentation and releases owned player controls/UI/strands. Cooldown uses GE_VoidTentacleCooldown and native Cooldown.VoidGazer.Tentacle tag, applied after accepted activation ends or is cancelled; editable ability CooldownSeconds defaults8. Component keeps existing4s Hook/450cm Whip/shake0 and single-multiplier damage. Read-only CanStartPresentation validates owner/controller, live player, reach, LOS, action exclusion and captured-victim ownership.
EEnemySkillRange appends AnyDistance after Background without renumbering existing entries. FEnemyPhaseSkillSet.AnyDistanceAbilities is granted with existing class deduplication and selected by UseRandomSkill. Its selected target is exposed through GetSkillTarget; the ability uses this target rather than an implicit local player. AnyDistance bypasses distance-group partition only, not skill reach/LOS/cooldown. BP_VoidGazer configures BGA_VoidGazer_Tentacle under GAS/Abilities in both phases. BT retains close ground melee first, then common UseCombatSkill(AnyDistance), then prior slam/transfer/look/crawl branches; failed activation falls through immediately. The shared task waits for the owned GAS handle and cancels it on abort. Full patrol/combat hierarchy and ground-to-wall re-entry remain outside this batch.


## FEAT271 directional charged blink (playable; strict reference fidelity pending)
BP_Infiltrator now grants BGA_Infiltrator_DirectionalBlink (V6) by default. Prior five variants are retained; sixth Directional button uses the existing tracked-grant lifecycle. UGA_InfiltratorDirectionalBlink derives the original swept blink; base adds only virtual GetBlinkDirection. V6 resolves pending then last movement input, otherwise view-forward projected horizontally;750cm/100ms,3charges and3second serial recharge are editable provisional defaults. Recovery uses world time and preserves existing partial recovery when another charge is spent. Fully blocked activations consume no charge; accepted shortened movement consumes one. No invulnerability, wall phase-through or Recall.
UInfiltratorDirectionalBlinkComponent is a pawn-owned presentation component. New owner-local BlinkV6 materials implement neutral scene plus cyan speed lines, body overlay, static departure pose and world trail meshes. No gun outline meshes/materials. Bounded firstperson arms pull is applied additively and restored; no animation retargeting/source assets. Common BlinkPresentation uses minimal mode for HUD/audio; optional sound override preserves legacy consumer settings. HUD queries actual V6 instance charges; no separate recharge clock. Normal ability end permits short visual tail; cancellation/death/unpossess/debug switch/endplay restore overlay/arms and destroy transient components. Defaults and final verification recorded in FEAT271 archive.


FEAT271 V6 timing supersedes the previous continuous100ms pawn dash: accepted capsule movement occurs once in BeginPresentation after full-path validation. AdvanceBlink owns duration/lifecycle only; camera catches up independently. Serial charges and old five variants remain unchanged.


FEAT271 reference correction (2026-10-08): V6 gameplay still uses a same-frame swept capsule relocation,100ms camera catch-up and3 serially recharged charges. Presentation now uses arm-only procedural IK (not spine translation), channel2 local cyan point lighting (not gun emissive MIDs), thin unwarped screen streaks and separately timed departure/trail/body fades. See arch/07-character-classes.md and arch/12-anim-blueprint.md for current ownership and cleanup. Original camera/FOV/light internals are not claimed.


FEAT271 airborne/inertia correction (2026-10-08): V6 restores captured movement mode and velocity immediately after relocation and releases bOwnsMovement; it no longer freezes walking/gravity for100ms. Existing common activation has no ground-only gate. Ascending and descending horizontal blinks retain Z velocity and land through CharacterMovement. Camera presentation independently uses acceleration/deceleration plus an editable24cm/160ms monotonic tail (short path cap8percent), without extra gameplay boost or endpoint overshoot. Legacy V1..V5 retain their movement lock. See arch07 and FEAT271 verification.


FEAT271 no-cooldown preference (2026-10-08): bUseCharges defaults false. V6 skips charge spending and recharge gating; normal active-instance exclusion remains for the100ms skill execution. Presentation sends Maximum=0 to hide charge bars while retaining V6 screen treatment. Optional MaxCharges/RechargeSeconds are inactive unless explicitly opted in. No changes to camera/movement/airborne logic.


FEAT274 (2026-10-10): six native tags/new GC assets: GameplayCue.Weapon.DisintegrationGun.Dissolve, GameplayCue.Weapon.RepairGun.Dissipation, GameplayCue.Character.Enemy.VoidGazer.{EyeCharge,EyeRelease,Landing,TentacleImpact}. Local single-player presentation uses GameplayCueManager.HandleGameplayCue; a target object does not require an ASC. All names/registry GameplayCueName must match and are cold checked. No damage GE added. Object removal belongs to DisintegrationComponent, Cue starts cosmetic helper only; Void and Repair gameplay remain in their original owners. This is local dispatch, not a new network replication implementation.

FEAT271 ImpactFrame integration (2026-10-10): DirectionalBlinkComponent owns optional ImpactFrameSystem and ImpactFrameDuration(default.3s,0disabled). BP_Infiltrator references NS_Infiltrator_BlinkImpact under BlinkV6/Effects/ImpactFrame;9owner-local source-derived assets. Creates Niagara attached to HeadCamera only for local controlled current view target, OnlyOwnerSee, no collision/shadows; User.Lifetime set before activation. Cutoff at duration, view-target loss and existing Shutdown prevent lingering effects; repeated Begin replaces previous instance. Existing camera carry/light/pose/gameplay/no-charge policy and current saved1500cm preserved. Surface/SceneColor Niagara sprite, separate from existing PostProcess material slot.
Runtime also sets User.cameraPush=100cm before activation: source0 places the sprite inside the game-camera near plane. Rendered first-person verification and final80ms cleanup passed.

2026-10-10 user timing correction supersedes80ms: original source particle lifetime300ms is restored in native defaults and BP. Original Niagara curves unchanged; normal100ms ability completion lets all300ms play.

FEAT271 screen visibility hardening: ImpactFrame Niagara explicitly disables scene occlusion queries/scalability culling and uses local bounds[-6000,6000]cm around attached camera, enclosing source10000cm sprite. Material DisableDepthTest alone does not disable Niagara scene proxy occlusion. Owner-only/current-view restrictions and300ms cutoff remain.

FEAT271 rotation correction: ImpactFrame follows HeadCamera translation but uses absolute identity world rotation. The source emitter is local-space while ReturnCameraProperty001 supplies a world-space vector; inheriting camera rotation applies it twice and can put the overlay behind the view. Preserve SetAbsolute(false,true,false) and zero world rotation. was_recently_rendered alone is insufficient; verify actual pixels across yaw/pitch.

FEAT271 user opt-out (2026-10-10): production BP_Infiltrator DirectionalBlink.ImpactFrameDuration=0 disables the optional Easy Impact Frames overlay. Original blue streak/camera/pose/light/sound presentation remains active. Impact assets/code are dormant for possible future explicit opt-in; do not re-enable automatically.

FEAT271 ground traversal: legacy FindBlinkDestination remains fixed-height via new virtual ResolveBlinkDestination hook. V6 BuildBlinkPath follows walkable ground using actual capsule/channel responses, character MaxStepHeight/IsWalkable/CanStepUp, swept up/forward/down segments including head clearance,1024segment work cap and horizontal range. Ledges transition to horizontal airborne continuation; airborne start preserves original horizontal resolver and velocity. V6 executes swept waypoints in the same activation frame, not a final straight chord. Saved1500cm/nocharges/ImpactFrameDuration0 retained.

FEAT271 blue streak expansion supersedes static-width/scrolling-tip screen description: original M_Infiltrator_DirectionalScreen scales view-centered streak coordinates1->2.25 over120ms, uses fixed per-ray tips, and retains original180ms fade/cyan palette. Only streak geometry expands; scene sampling/FOV/camera unchanged. Purple ribbons and native direction parameter experiment reverted. See FEAT271 archive and D:/UnrealWork/BlinkBlueExpansion.

FEAT271 Crysis screen tail: DirectionalBlinkComponent owns ScreenTailMaterial and ScreenTailAfterArrival(default1s), transient ScreenTailPostProcess/MID and end age. Source-derived3assets under BlinkV6/Effects/Crysis preserve hex refraction, replace global MPC Radius with local scalar and add BlinkAmount(identity at0). Owner PP priority114/material-1 precedes blue streaks(priority115); normal Finish resets expiry to currentAge+tail, last250ms fade. Tick lifetime extends only while tail MID exists. Repeat/Shutdown/cancel/death/unpossess/variant switch clean it; view target loss destroys it. Existing blue180ms, camera260ms, pose/light/body ages unchanged. BP configured1s, ImpactFrame0; production1500/nocharges. See FEAT271 and D:/UnrealWork/BlinkScreenTail.

FEAT271 Glitch16: component now also owns ScreenGlitchMaterial/ScreenGlitchDuration(.22s) and transient ScreenGlitchPostProcess/MID. Priority113/material-2 before hex114/-1 and blue115; original source16 function drives bounded viewport displacement and blue/orange emissive mask. Source-derived14assets under BlinkV6/Effects/Glitch16; owned triplanar functions converted to viewport projection, no Surface transforms/foreground actor.12ms onset/fade after40percent; repeat/expiry/view loss/Shutdown clean. Crysis normal1.35/UV8/Radius1/tail1s. Blue layer now expands over220ms and fades75..300ms. No gameplay range/camera/pose changes.

FEAT271 latest user correction: production ScreenGlitchMaterial=None/ScreenGlitchDuration=0 disables Glitch16; do not re-enable automatically. Crysis MI Radius=0.8 (was1), extending hex inward; normal1.35/UV8/tail1s/blue220ms expansion and300ms fade retained.

FEAT271 latest blue timing: expansion350ms, fade100..450ms. Hex tail1s inclusive of250ms fade; actual normal-speed measurement expires1.10s after activation. Glitch remains disabled.


FEAT275 Executive remote control: AFPSTheExecutive owns UTheExecutiveDroneRemoteComponent and grants UGA_TheExecutiveControlDrone. Q uses existing BlinkAction; possesses existing ATheExecutiveDrone, pauses its AI, retains real body ASC/grants/health. Remote transition bypasses FPS base possession initialization via ACharacter to prevent healing/regrant. Companion eligibility accepts remote session. Drone existing swept Flight handles manual WASD/QE; actual camera class BPC_TheExecutive_DroneBodyCamera under Executive/Gameplay/Abilities/DroneControl/Camera, FOV112..45, mouse look/wheel/F return. Source BodyCam DisplacePower setter fixed0 (original ArtifactsAmount*5 wiring); Distortion default0 alone does not disable water. Glitch1 owner PP lasts.28s and cleans on return/cancel. Restore body visibility and AI. Drone EndPlay destroys AIController only; remote return tolerates PC Pawn already cleared. External role change cleans abandoned body/drone without stealing new possession. No core/base edits. See FEAT275 archive for evidence/settings.


FEAT274 charged fire: Input.Weapon.PrimaryFire still starts the configured primary ability. GA_ChargedShoot waits on PrimaryFireReleased and PrimaryFireCanceled (exact event tags); also listens for Reload. Release before ChargeDuration cancels; release at/full executes the shared accepted-shot path once. Optional per-weapon bFireWhenFullyCharged fires at full without repeating. No ammo use/hit during charge. EndAbility clears timer, wait tasks, focus delegate, charge Cue and component presentation; switch cancels the active primary spec before removal. Application deactivation, death/unpossess, reload and menu focus transitions cancel instead of firing. GameplayCue.Weapon.Charge is presentation-only. LocalOnly follows existing single-player GA_Shoot policy; multiplayer replication is not added.


FEAT274 release VFX: GA_Shoot::ExecuteShot supports opt-in bAttachMuzzleEffect on Firearm; DisintegrationGun enabled, others default unchanged. Attaches local-space Niagara to skeletal MuzzleSocketName and bounds looping tail by AttachedMuzzleEffectLifetime1.2s through weak timer. Charge Cue cleanup remains immediate; released muzzle tail is a separate accepted-shot effect.


FEAT276 airborne attraction: existing GA_Shoot -> AAttractionBullet -> independent AAttractionField. UGE_AttractionDamage supplies periodic Health damage2/.5s; cleanup removes per-field active handles. UGCN_AttractionField/GameplayCue.Weapon.AttractionProjectile.Field manages field presentation; existing character Hit Cue routes electrical SourceObject to in-place configured hit animation without projectile blood. See arch09 for ownership/parameters and feature archive for runtime evidence.


FEAT-276 final revision (2026-10-11): ElectricGun creates field instantly500cm ahead; no projectile flight or attraction. Periodic GE2damage/.5s,2.5s duration and field Cue retained. Electrical reaction strength1 (supersedes .45); configured Front/Back/Left/Right selected from orb relative direction, in-place flag suppresses animation root displacement without changing movement mode. Four actual selected sequences verified by native automation; actual GAS PIE damage and cleanup passed. See arch/09-equipment-system.md and feature archive.


2026-10-12 category cleanup: all4VoidGazer ability Blueprints now live in /Game/Enemy/Nightmare/VoidGazer/GAS/Abilities (GroundSlam/Melee/WallLeap alongside Tentacle). GameplayCues unchanged. Medusa MaterialParameterCollection DA_Medusa_OrionGlobalGameplayCollection_cc180041 now lives in /Game/Enemy/Nightmare/Medusa/Materials/Native; historical name retained. References updated/cold verified, no behavior or parameter changes.


## FEAT-280 CoreMorph GA ownership (2026-10-12, supersedes FEAT-081 executor descriptions)
GA_CoreMorphTailStrike owns target snapshot, Windup/Thrust/Recover, timing, sweep/contact deduplication, damage, cancellation and cooldown. GA_CoreMorphMissileBarrage owns salvo schedule, missile state/flight/contact, damage and cleanup. Their durations/ranges/damage settings now live on GA defaults; all saved BP baseline values matched native defaults and were preserved. AbilityTask_CoreMorphTick scopes callbacks to ability lifetime: tail uses Body's 120Hz pose substeps, missile uses task tick; invalid body availability also cancels the tail GA. Physical tail return is a Body pose blend after cancellation.
GCN_CoreMorphTailCharge/TailBlast/MissileBarrage are now GameplayCueNotify_Actor classes (A prefix in C++, reflected names unchanged). CoreMorphTailCue shares rendering math; each Cue Actor owns its meshes/decals/lights, with no persistent Boss attack-effects components. Three existing GC assets retain their paths and exact GameplayCueName tags; all 7 material assignments migrated from BP components. Blast retains its 1.25s tail; cancel/death removes effects.
GA_CoreMorphReassemble remains in DefaultAbilities for explicit StartReassembly activation, excluded from PhaseSkillSets. BT Reassemble action refuses automatic selection. Existing opt-in encounter opening in SpaceStation remains; manual preview uses ReassembleOnActivation=false. No new input or preview map. Tail stays phase1 Near; missiles remain unequipped (runtime validation grants temporarily).


## FEAT281 scan smoke removal (2026-10-12)
The previously disabled scan smoke branch is removed, superseding historical FEAT163/164/166/180 smoke descriptions above. AEnemyBase no longer owns EnemyScanSmokeComponent. InfiltratorScanSmoke actor and all smoke settings/discovery/solver lifecycle are removed from InfiltratorScanPresentationComponent. Six exclusive Scan/Materials/Smoke fluid materials are deleted; saved BP references and enemy component templates are resaved. Current scan state, grid, gold enemy highlight, filter, return and audio remain owned by InfiltratorScanPresentationComponent. Do not restore obsolete smoke components or materials.


## FEAT-283 VoidGazer GAS migration status (2026-10-13)

GA_VoidMelee and AbilityTask_VoidMelee now own the melee clock, facing, hit and cooldown after removal of VoidCombatComponent. GA_VoidTentacle owns activation, cancellation and GE cooldown, while AbilityTask_VoidTentacle currently observes completion only; the Tentacle component still owns its attack phases and victim transaction. GA_VoidWallLeap and its AbilityTask own activation, cancellation and frame calls, and GE_VoidWallLeapCooldown owns automatic planning cooldown; the Leap component still owns Start/Air/Land. These two phase migrations remain open in FEAT-283. Do not copy this intermediate split for a new enemy.


## FEAT-283 final VoidGazer GAS ownership (2026-10-13; supersedes the intermediate status above)

GA_VoidMelee owns melee state. GA_VoidTentacle owns Windup/Locked/Extend/Hook/Whip/Slam/Retract, timers, locked target and escape/impact results; AbilityTask_VoidTentacle receives PostPhysics callbacks from the motor component and advances the GA. Cancellation synchronously ends the ability and restores victim controls. GA_VoidWallLeap owns Start/Air/Land, timing, outcome and cooldown; AbilityTask_VoidWallLeap advances it, while the Leap component supplies path/collision/movement helpers. EyeCharge Cue actor owns the warning decal as well as eye effects. Persistent Tentacle/Leap components are lower-level physical executors; do not put new attack phase state there.


2026-10-14 supersession: the Dev Panel no longer offers Blink variant buttons. Historical variant grant API and assets remain for internal tests; Q continues to activate the configured Blink ability.


2026-10-15 Void combat selection: UGA_VoidTacticalAction provides six native GAS classes for close chase, ground retreat, ground pounce, ground return, wall roam and wall transfer. These classes are entries in existing PhaseSkillSets. They call existing ground/wall movement components and GA_VoidWallLeap executor, retaining support, swept path, impact and cooldown rules. Ground return owns direct leap or one safe preparatory ground hop followed by wall leap. GA_VoidTentacle rejects ground targets within 200cm but allows wall close range; GA_VoidWallLeap ground-slam subclass plans its wall-to-ground destination on activation. An action that immediately rejects an unsafe plan is skipped by random pool selection.

2026-10-15 Void correction: UGA_VoidGroundReturn remains a PhaseSkillSets candidate during the shared WallLeap cooldown. It requires a valid direct wall return or one safe preparatory hop. Ground pounce, ground retreat and wall transfer retain their own cooldown checks. 

2026-10-15 user clarification: Void Near is <=500cm. GroundChaseAttack and GroundRetreat are the two random Near ground behaviours; GroundReturn is all-distance. The configured melee GA stays in the Near PhaseSkillSets pool as GroundChaseAttack's executor and is skipped as an independent random choice. StartMelee directly activates that configured GA instead of rerolling the pool. Ground tentacle requires >500cm; pounce is selected from Mid/Far beginning beyond500cm. No 200cm chase cap.

## Void attack facing (2026-10-15)

Void melee already turns before its strike. The ground tentacle GA turns the actor during its Windup phase and may enter Locked only inside 5 degrees of the live target; an unachievable alignment cancels within one extra second. The damaging ground pounce GA turns during its Start warning and admits takeoff only inside 5 degrees of the locked landing direction. The wall attachment pose remains constrained by its wall basis.


2026-10-15 Void ordinary melee reach correction: GroundChaseAttack FollowActor acceptance is Max(5cm, MeleeRange * .15), currently 24cm at 160cm melee reach. Nav path following also includes collision radii; the former 136cm acceptance ended the chase before the configured melee GA activated. The live BT regression now requires melee activation and health damage. Wall-to-ground slam requires at least 300cm horizontal travel and retains swept path validation.


## Current FEAT302 scan interaction gate (2026-10-21)
The scan wave highlights only AInteractableBase actors that carry UScannableInteractableComponent, a UHighlightComponent subclass. First wave contact calls CompleteScan once, unlocks E interaction and invokes the actor OnScanCompleted event. Highlight clears when scan retracts; discovery persists. Ordinary interactables and arbitrary IHighlightable actors are not object scan targets. Enemy gold overlays remain on their separate path.
