# 装备系统

## FEAT-297 ExplosionGun interactions (2026-10-20)

`AExplosionGunBullet::ProcessHit` resolves `UExplosionInteractionComponent` on the hit actor or its parent (for ChildActorComponent glass panes). Enemy is an explicit compatible path. An eligible direct target attaches the dart and starts the existing fuse; an unconfigured surface places the dart outside the impact face, simulates its narrowed root CollisionSphere against blocking ground (the visual mesh has no simple collision), lets it fall under gravity without added rebound velocity or fuse, then clears it after ExplosionDelay (zero delay means next tick). On detonation, the dart preserves damage, impulse, bullet time and cue behavior, then dispatches a once-only break context to the direct target and opted-in nearby targets. It no longer fractures all nearby Geometry Collections by type.

`AChaosDestructibleCube` owns its Chaos break using the received strain/impulse; `ATestInteractionProp` preserves its legacy presentation. `AExplosionGlassWindow` owns the interaction component and a vertical `AExplosionGlassPane` Child Actor using Tempered Glass skeletal shards and Niagara. Placeable Blueprint: `/Game/Actors/Destructibles/ExplosionGlassWindow/Blueprint/BP_ExplosionGlassWindow`. Glass does not detonate. See `Documentation/Dependencies/TemperedGlassIntegration.md` and FEAT-297 archive.

Direct glass hits set `Data.Explosion.SuppressBlastVFX` through the glass window's interaction component. The ExplosionGun Gameplay Cue omits only its generic blast Niagara for that context; audio and shake remain, and other targets retain normal blast VFX. Nearby glass break does not suppress the source explosion's visual.

## FEAT-298 vault presentation (in progress, 2026-10-17)

`AEquipmentBase` exposes `WindowVaultLowerOffset` and `WindowVaultLowerRotation` for a shared camera-viewmodel weapon-lowering gesture. `AFPSCharacterBase` temporarily gates fire/switch, then hides/restores the equipped actor during window traversal. Inventory ownership, equipment abilities, Linked Anim Layer and ordinary `Unequip` lifecycle remain intact. Production montage hookup awaits external animation visual approval; see FEAT-298 archive.

## 2026-10-11 消解枪模型取景（待用户摆位）

正式 BP_DisintegrationGun 仍使用原表现；新 SKM_DisintegrationGun 及必要枪械 Skeleton/PhysicsAsset、材质贴图已归入 Weapons/DisintegrationGun。两套可选特效 NS_DisintegrationGun_MuzzleFlash/Plasma 位于 Effects/Plasma/Systems，尚未接入开火。通用摆位蓝图 /Game/Actors/Tools/GunFraming/Blueprint/BP_GunFraming 新增 GunSkeletalPreview 骨骼组件作为 Gun 子级；Gun 保持相机相对定位父组件，静态模型清空，骨骼子级单位变换。用户调整 Gun 并保存取景目标后才开始正式安装/动画匹配，当前起始变换不是验收值。详见 FEAT-274 archive。

## FEAT-269 统一满弹匣（2026-10-08）

`AFirearm` 只使用 `MagazineCapacity` 配置弹匣容量；独立 `InitialAmmo` 字段已移除。BeginPlay 与成功换弹均令 `CurrentAmmo = MagazineCapacity`，出生不消耗备用弹匣，换弹消耗一个。爆炸枪 `BP_ExplosionGun` 容量为2，因此出生和换弹后均为2发；其他枪容量保持原配置。满弹匣不能换弹，无限弹药模式不扣弹且不允许换弹浪费备用弹匣。回归入口：`TheManTest.Player.Weapons.SavedDamageAndMagazineAmmo`。

## FEAT224 foam traction (2026-09-30)

On environment impact BulletMesh receives WalkableSlope_Increase60degrees so rounded upper facets support characters while steep/vertical sides remain unwalkable. AFPSCharacterBase owns2x ground/braking friction and walking braking only while walking on this mesh, restoring snapshots on exit/jump. No physical-material asset required; CharacterMovement uses its own friction. See arch03 and FEAT224 archive.

## FEAT-223 Foam player separation (2026-09-30)

Environment impact immediately removes Pawn blocking from the expanding root sphere. BulletMesh provides walkable Pawn collision during growth. Before accepting each new size, query player capsules against the actual convex mesh and apply swept minimum-translation separation (4cm/update, 20cm total per foam, no launch/velocity change). If blocked by a wall/other foam or beyond the budget, retain the previous safe size and finish; discard only the new foam if even its previous/initial size encloses a player. FinishExpansion preserves the accepted size instead of resetting to maximum. Growth delta capped at .05s for frame hitches. Settled projectile blocking, hazard overlaps, enemy slow and airborne range remain unchanged. Retention now follows the restored configurable timer below. This supersedes the historical growing-sphere Pawn behavior below. No temporary player pass-through state.

## FEAT-106 RepairGun 驻留泡沫（2026-09-17）

- 2026-10-10 用户要求恢复定时消散，覆盖原永久驻留规则：LifetimeAfterExpansion 默认10秒（Bullet|Dissipation），从 FinishExpansion 起计时，0关闭。环境命中先清除 Actor LifeSpan；膨胀结束后设置原生 lifespan timer，LifeSpanExpired 复用 Dissipate 和原 Gameplay Cue/NS_RepairGun_PixelDissipation。到期停止碰撞并销毁，危险区通过已有 OnDestroyed 恢复。命中敌人仍减速后立即销毁；Phantom 穿透规则不变。
- 飞行时仍使用 CollisionSphere；固化后 BulletMesh 使用 SM_RepairGun_Bullet 的单个 26-DOP 简化凸碰撞，QueryOnly 阻挡 Pawn/RepairGunBullet 并允许踩踏。根球体不再阻挡这两通道，保留危险区 Overlap。后续子弹可命中固化泡沫形成堆叠；危险区压制随驻留泡沫持续。
- 重复 ProcessHit 不再重启膨胀；零膨胀时间或最大倍率<=1 时直接完成，固化后 Actor Tick/ProjectileMovement 关闭。无跨关卡/存档持久化。
- 验证入口 TheManTest.Player.Weapons.RepairFoam：TestMap PIE 临时生成碰撞地板与正式 BP 子弹，以真实 ProjectileMovement/CharacterMovement 验证落地、跳跃、堆叠、关闭定时后的持续站立及敌人减速。RepairDissipation 验证默认10秒到期与两种消散共用特效。测试不保存地图。

- 2026-09-06用户澄清为仅延时爆炸击退：ExplosionGunBullet首次命中只走原伤害/附着流程，不调用击退组件。延时爆炸对所有实际扣血且存活的Enemy调用EnemyHitReactionComponent，不再限定AttachedHitActor；各目标在伤害前按当前朝向和爆心计算自己的局部方向，水平距离近零时回退弹道。统一Strength=1，前后左右4条动画，不区分部位，播放中不重启。伤害数值/遮挡/去重/死亡布娃娃/子弹时间/径向物理/附着骨骼保留原流程。正ExplosionDamage但GE没有实际扣血同样不触发。

## 2026-09-05 当前触发条件与致命枪击击飞（覆盖下文旧Chaos结果监听）

- ExplosionGunBullet在有效ProcessHit时缓存真实命中组件是否GeometryCollection。Fuse结束时，本次范围伤害杀敌或该直接Chaos命中标志满足即请求子弹时间一次。不要求Chaos实际破碎；周边物体被波及破碎/击飞不算。范围Chaos物理场保持原行为，但不再调用ExplosionOutcomeSubsystem.Watch；旧监听类暂留无调用。
- ABulletBase直接致命一枪添加Enemy.ProjectileKillKnockbackSpeed沿弹道和ProjectileKillUpwardSpeed沿世界上方的全身速度，默认250/120cm/s，随后施加原命中点冲量。所有弹体子类共用；非致命/旧尸体不加全身速度，范围爆炸不走这条直接枪击逻辑。两参数可分别设0。

## 当前死亡布娃娃与子弹物理

- ABulletBase统一在伤害/警觉转向前定位敌人Mesh PhysicsAsset上的命中骨骼与骨骼局部点，伤害后对模拟身体施加AddImpulseAtLocation。致命一枪与后续尸体命中共用路径；零伤害同样可推物理身体，存活非模拟身体不受此冲量。冲量强度统一取Enemy|Death.ProjectileHitImpulse。尸体不再扣血或刷新寿命，但仍通过显式Hit Cue播放肉体声/血迹，不新触发痛呼。
- ApplyPhysicsImpulse移除Enemy类型排除，IsSimulatingPhysics已过滤非物理目标；仍排除GeometryCollection防重复Chaos冲量。先范围伤害切布娃娃，再对刚死/已有尸体及其他模拟物体施加径向冲量。PhysicsImpulseRadius/Strength仍为400/800，距离衰减、墙体遮挡保留；推动旧尸体不触发子弹时间。
- 死亡不再等于EndPlay：附着弹保留Fuse并跟随布娃娃骨骼，可在尸体上爆炸；尸体寿命到期销毁Actor时，清理未爆弹与身体血痕。地面血迹按原寿命。

## 2026-09-05 爆炸普通物理冲量和附着生命周期

- ExplosionGunBullet新增Bullet|Explosion|Physics：PhysicsImpulseRadius=400cm、PhysicsImpulseStrength=800（速度变化强度）。范围内SimulatePhysics组件去重并排除Enemy与GeometryCollection，以Visibility测试墙体遮挡后AddRadialImpulse线性衰减、bVelChange=true。Chaos仍走独立破碎/冲量路径，不重复施加；普通物体飞散不请求子弹时间。
- 身体附着爆炸弹订阅目标OnEndPlay，目标死亡/销毁时清除尚未爆炸的弹体及Fuse；本弹正在Detonate造成击杀时允许自身爆炸调用完成。首次伤害直接杀死敌人也不留下新附着弹。GCN_EnemyHit的身体血痕现在为Character拥有的Decal组件，骨骼附着+原淡出/寿命，死亡随Owner清理；环境血迹仍独立寿命。MovingEnemyAttachmentCleanup覆盖行走/转身/受击及倒计时死亡、爆炸击杀两条清理路径。

## 2026-09-05 当前爆炸弹身体附着修正

- ExplosionGunBullet在Super伤害/转身前沿真实入射方向、Sweep中心线（非球体ImpactPoint）追踪Mesh PhysicsAsset，长度按Mesh Bounds计算；只接受有效骨骼、非初始穿透、朝入射侧法线。若掠空，仅允许胶囊半径+10cm内的入射侧最近PhysicsAsset表面补点；无法可靠定位时隐藏弹体Mesh，原伤害/倒计时保留，不显示悬空胶囊附着。
- 解析出的点/法线先存到父骨骼局部空间，再于Super之后转回当前世界坐标并KeepWorld附着；实际爆点跟随骨骼。环境附着、准星汇聚不变。PhysicsAsset表面仍是模型近似，不是渲染蒙皮逐三角形表面。
- StickyBodySurfaces六方向真实物理飞行验证入射侧位置、有效Mesh骨骼、可见弹体与受击骨骼随动；StickySurfacesFinal.log Success，准星和原Sticky回归均Success。

## 2026-09-05 当前爆炸结果触发（覆盖下文无条件子弹时间和Enemy Ground配置）

- 仅本次延时范围伤害把Enemy的Health从正数降至0，或本次爆炸施加Strain后的真实Chaos Break事件，才请求BulletTime。仅命中/受伤/空地爆炸/Strain不足/已散落碎块再次受冲量不请求。首次5点命中伤害不属于本次延时爆炸击杀判定。
- ExplosionGun专属`Effects/ExplosionOutcomeSubsystem`在提交Strain之前短期订阅受影响GeometryCollection的OnChaosBreakEvent，检查组件和爆炸半径，窗口0.2游戏秒；所有组件共享FExplosionOutcome，单次爆炸仅请求一次。死亡已满足条件时不再订阅Chaos。组件通知原状态在结果/超时/WorldEndPlay/Deinitialize时恢复，弱引用不依赖已销毁弹体。BulletTimeSubsystem自身曲线/距离/不重入规则保持不变。
- EnemyExplosionEffect现在是`/Game/Weapons/ExplosionGun/Effects/EnemyExplosion/Systems/NS_ExplosionGun_EnemyDetonation`，源TMIIR Overview_Map的N_ExplosionAir_007，115包本枪独立依赖；EnemyEffectOnGround=false，真实身体爆点播放，无地面也正常生成。环境仍NS_ExplosionGun_Detonation/Ground006地面投射。独立声音、震屏和用户时间参数保持原值。

- 2026-09-05晚：ExplosionGunBullet对原来可见且受范围伤害的存活Humanoid调用EnemyHitReactionComponent，保留附着Surface.BoneName/目标弱引用/入射方向作近零方向兜底。伤害筛选/去重/Visibility墙体遮挡不变，墙后不触发受击Rig；不由Cue处理骨骼状态。
- Enemy已配置迁入的N_ExplosionGround_006（NS_ExplosionGun_Detonation）和独立SCue_ExplosionGun_EnemyDetonation，不再空槽。EnemyEffectOnGround=true时仍用原GroundHit投影，缺地面跳过Niagara，声音/震屏用真实爆点。保留用户最新BP子弹时间TimeScale=.05/SlowIn=.01/Hold=1/Recovery=.01，震屏6，环境音量3；不是原生默认值。

- FEAT-080随机音高/音量现归每个音效的Sound Cue资产，开火/空仓/环境命中/肉体/爆炸按用途配置衰减和并发，详见arch/14-audio-policy.md。GCN_ImpactFeedbackBase命中角色不出声，角色Hit负责自己的声音；粒子仍保持原规则。伤害与爆炸Fuse保持原流程。

- ABulletBase零伤害有效命中Enemy时，单独以真实HitResult调用Enemy的HitReactionCue；穿透/重复Hit先返回，正伤害仍由Health路径负责，不重复表现。电击弹Damage=0保持不变。

## 爆炸枪附着弹（2026-09-05，以下新规则覆盖旧HitStop实现）

- HitStopSubsystem/FHitStopSettings/HitStop属性已删除，改为Core/_Shared/Feedback/BulletTimeSubsystem与FBulletTimeSettings。弹体Detonate独立请求BulletTime，不经过GC。BP_ExplosionGunBullet → Bullet|Explosion|Bullet Time：TimeScale=.2、SlowInDuration=.05、HoldDuration=.08、RecoveryDuration=.25，均真实秒，Inner/OuterRadius=200/1500cm。Smoothstep减速再恢复原World.TimeDilation，无超速；活动期不重启/延长，100ms恢复间隔，外部改速让出控制，WorldEndPlay/Deinitialize清理。世界物理/动画/游戏时间Timer放慢，声音不变速。
- 同一个Explosion GC按Data.Explosion.EnemyImpact分别选择EnemyExplosionEffect/EnemyExplosionSound；两个Enemy槽目前均空，独立跳过且无环境回退；环境Ground规则不变。原Damage5/Fuse2/范围20伤害400cm/Chaos不变。下列旧HitStop参数仅属历史，不再是可调入口。

- 最新覆盖：Enemy与环境均附着/倒计时，结束均执行范围伤害、Chaos、爆炸Cue和HitStop。`ExplosionDamage=20`、`ExplosionDamageRadius=400cm`、`ExplosionDamageEffectClass=GE_BulletDamage`只影响延时爆炸，原Damage=5首次伤害不变。Overlap候选按Enemy去重，Visibility墙体遮挡先判定，敌人不互相充当墙；只向Enemy ASC施加一次GE，源ASC失效时目标ASC构建Context/Spec。玩家与其他Actor不受范围伤害。
- `Bullet|Explosion|Hit Stop`内FHitStopSettings：Enabled=true、Duration=.06真实秒、TimeScale=.05、InnerRadius=200cm、OuterRadius=1500cm、MaxContinuousDuration=.12真实秒。由弹体Detonate独立请求Core/_Shared/Feedback/HitStopSubsystem，不由GC管理；离相机越远越弱，无本地玩家或非单机网络模式不触发。
- HitStopSubsystem是World级管理器：真实时钟计时、相对进入前TimeDilation缩放、最强请求优先、连续时长上限、结束50ms恢复窗口；恢复原速度而非强制1，外部系统改速度时让出所有权，WorldEndPlay/Deinitialize清理。世界运动/动画/Niagara/Chaos和游戏时间定时器会短暂放慢，声音不变速。
- 爆炸视觉类型通过Params.AggregatedTargetTags的原生元数据Data.Explosion.EnemyImpact快照传入，目标死亡后仍保留原命中分类。环境沿用Ground投影；EnemyExplosionEffect独立且默认空，不回退地面大decal。声音/震屏共享同一Explosion Cue配置，不复制两套Cue。

- 延时爆炸两种命中均触发Chaos。Bullet|Explosion|Chaos可调ChaosRadius(400cm)/ChaosStrain(500000)/ChaosImpulse(1200，速度变化)；半径内GeometryCollection去重并排除Enemy所属组件。Strain后0.05游戏秒施加径向冲量，弱引用不依赖已销毁弹体。
- TestMap Dev Panel 的四物品预设使用 `ATestInteractionProp`：爆炸半径查询到其静态网格时调用 `Fracture`，把原网格切换为对应的预切割 GeometryCollection，施加相同的 Strain/Impulse/Spin。测试门的七个网格各有独立的预切割资产，属于同一个可消融 Actor；2026-10-22 起此预设不再可扫描或交互。源网格在消融中或已消融后不会再生成碎块。仅此测试预设走静态网格到 Chaos 的切换，普通关卡物件不自动破碎。
- Bullet|Explosion|Ground：GroundSearchDistance=2000cm、GroundMaxSlope=45度。仅Actor Tags里的ExplosionGround为地面，不是Component Tag或Gameplay Tag；Object Multi Trace跳过阻挡Cube，法线Z判坡度，拒绝Enemy/弹体/GeometryCollection。
- 可摆放类Actors/Destructibles/DestructibleCube/ChaosDestructibleCube拥有GeometryCollection根组件；蓝图/Game/Actors/Destructibles/DestructibleCube/Blueprint/BP_ChaosDestructibleCube。FractureAsset与Toughness可逐实例配置，大小使用Actor Scale。关闭碰撞伤害，使用Destructible通道。显式CreateTestCubeAsset(bRebuild=false)仅编辑器创建预切分GC；true显式重建同路径资产，不在运行时切网格。现为42块大小混合的Voronoi，24分散随机点+18密集点、seed92417、断面0.8cm噪声；在临时集合切割成功后替换资产，保留实例引用。
- ChaosAngularSpeed默认5rad/s，与原ChaosImpulse独立；两种爆炸释放碎块的延迟回调均向同一范围提交RandomVector角速度场，径向Mask限制半径（Culling_Outside保留Mask非零值）。

- `Weapons/ExplosionGun/Bullets/ExplosionGunBullet.h/.cpp`：`AExplosionGunBullet : ABulletBase`，`BP_ExplosionGunBullet` 的原生父类。基类先处理首次伤害、原 Impact Cue 与 Phantom 穿透；有效命中后停止碰撞/移动，附着组件或骨骼并计时。基类仅增加protected只读 `HasProcessedHit()`，其他枪行为不变。
- 蓝图 `Bullet|Explosion / ExplosionDelay` 默认2秒，可调；`AttachmentOffset` 默认4cm。零秒延后到下一Tick执行，重复碰撞不重复触发，EndPlay取消计时。
- 倒计时结束执行上述范围伤害并调用独立 `GameplayCue.Weapon.ExplosionGun.Explosion`，随后销毁弹体。
- ExplosionGun/Effects/Explosion/Systems/NS_ExplosionGun_Detonation 来源TMIIR的N_ExplosionGround_006；全部116包依赖owner-local。Cue包与音频分别在本枪GAS/GameplayCues和Audio。原PhysicalImpact与首次5点伤害不变。

**何时读取：** 新增装备类型、修改装备生命周期（Equip / Unequip 行为）、新增插槽或动画层时。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Weapons/_Shared/EquipmentBase/EquipmentBase.h` | `Equip()` / `Unequip()` / `PlayEquipMontage()`；StaticMesh / SkeletalMesh / RectLight 组件；插槽名；EquipMontage；EquipmentAnimLayerClass |
| `Source/TheManTest/Private/Weapons/_Shared/EquipmentBase/EquipmentBase.cpp` | FPS 只对 ArmsViewMesh Link/Unlink；PlayEquipEffect 委托共享组件并按 bPlayEquipAnimation 播放本装备 EquipMontage；Unequip 取消效果、恢复材质并停止自己的动画。 |

装备 Montage 必须包含 `UpperBodySlot` 轨道：主 `TABP_BodyLocomotion` 的中央 `WeaponUpperBody` 会在 `DefaultSlot` 之后从 `spine_01` 覆盖上半身，所以只放 `DefaultSlot` 虽然 Montage 计时正常，动作仍会被最终武器层遮掉。`UpperBodySlot` 位于中央混合之后，适合 Equip/开火/换弹等需要进入最终上半身输出的动作。

开局和切换统一由 EquipmentManager.QueueEquipPresentation 等待一帧新姿势后显示。所有装备继承共享显现效果；bPlayEquipAnimation 默认 false，开启后同时播放本装备 EquipMontage，不影响其他装备的动画配置。
| `Source/TheManTest/Public/Weapons/_Shared/WeaponBase/WeaponBase.h` | 武器基类（继承 EquipmentBase，当前为空壳） |
| `Source/TheManTest/Public/Weapons/_Shared/Firearms/Firearm.h` | 射击参数：`bIsHitscan` / `HitscanRange` / `FireRate` / `BulletClass` / `MuzzleSocketName`；弹药：`MagazineCapacity=30` / `CurrentAmmo` / `SpareMagazineCount=3`、Consume/Reload/CanFire/CanReload 与 `OnAmmoChanged`；主/副射击及独立 `ReloadAbilityClass`；**`GrantedASC` 缓存** |
| `Source/TheManTest/Private/Weapons/_Shared/Firearms/Firearm.cpp` | BeginPlay 按蓝图容量初始化满弹；Consume/Reload 广播弹药事件；`Equip()` 写 AimSource 并 GrantAbilities；`Unequip()` 回收技能 |

`AFirearm` 还提供可按具体武器覆盖的 `MuzzleEffect / MuzzleEffectRotation / MuzzleEffectScale`；MuzzleEffectScale 原生默认 `(2,2,2)`，ExplosionGun 同为 2，ElectricGun 保留 2，RepairGun 保留专属 0.85；`UGA_Shoot` 每发在实际 Muzzle Socket 附着一次性 Niagara，并在激活前把 `MuzzleEffectScale` 的最大绝对轴写入 Niagara 暴露参数 `User.MuzzleScale`。需要响应该统一尺寸参数的 Niagara System 必须暴露同名 float；`NS_ElectricGun_LaserMuzzle` 的16个 emitter 与 `NS_ExplosionGun_PhysicalMuzzle` 的9个 emitter 均在 Particle Spawn 阶段通过 Uniform ScaleSpriteSize 模块读取该参数。建议武器蓝图把 XYZ 设为相同值，避免 Niagara 的统一倍率与组件非均匀 Transform 产生难以预测的组合。RepairGun 当前使用 `/Game/Weapons/RepairGun/Effects/Muzzle/Systems/NS_RepairGun_Muzzle_FlashSmoke`（FEAT-072，从外部 Sniper Scout 精确迁入，包含枪口闪光与烟雾）；专属前向烟雾材质/纹理位于同一 RepairGun Muzzle 目录，其余复用依赖位于 `/Game/Core/_Shared/Effects/Muzzle/`。
该 RepairGun 专属 System 的火焰、Glow、Lens Flare、Y 形火焰和火花颜色曲线已灰度化为中性灰；共享依赖仍保持原色，不得为了 RepairGun 外观修改 `/Game/Core/_Shared`。

`AFirearm` 同时提供默认关闭的 `MuzzleFlashLight` PointLight，以及 `bEnableMuzzleFlashLight / Color / Intensity / AttenuationRadius / SourceRadius / LocalOffset / Duration` 配置。组件固定使用 Unitless、Inverse Square、Cast Shadows 与 Affect Translucent Lighting；`UGA_Shoot` 只在成功消耗实弹后的开火反馈链调用，连续开火重置淡出计时，`Unequip` 立即清理。灯光尺寸与 Niagara 共用 `MuzzleEffectScale`：`AttenuationRadius` 和 `SourceRadius` 乘以最大绝对轴，Intensity、Color、Duration 与 LocalOffset 不变。当前仅 `BP_ElectricGun` 启用：Laser 枪口倍率 1.0，精确线性颜色 `(0.075319,1,0.652928)`、UE 5.7 视觉补偿强度 1800、半径 200、SourceRadius 60、0.1 秒线性淡出；灯位通过枪口局部偏移 `(1.480382,-6.734961,-15.577805)` 对齐源枪体的持枪侧。RepairGun 保持关闭。ExplosionGun 已按 Physical 1 源蓝图启用：线性颜色 `(1,0.551385,0.147041)`、强度 300、半径 87.370407、SourceRadius 60、持续 0.1 秒、局部偏移 `(8.851011,-17.618745,2.96144)`，枪口倍率 2。ElectricGun 后续用户将倍率调为 2，保持该覆盖。`VFXTestMap` 的 Bloom=4、Threshold=0.5 仅用于对照源项目 MainScene/参考视频的临时 VFX 审核环境，不是枪械效果的替代实现。

FEAT-080 起，MaintenanceWorker 的三把初始枪均为 AFirearm Blueprint：BP_RepairGun、BP_ElectricGun、BP_ExplosionGun。ElectricGun 的 SM_ElectricGun 为 Ballistics Rifle 02，使用 LaserMuzzle/LaserImpact；ExplosionGun 的 SM_ExplosionGun_Rifle 为 Ballistics Rifle 01，使用 PhysicalMuzzle/PhysicalImpact。三枪无描边组件、描边壳或描边材质。ElectricGun/Materials/M_ElectricGun_Surface 与 ExplosionGun/Materials/M_ExplosionGun_Surface 为专属科幻 PBR 表面，分别由 MI_ElectricGun、MI_ExplosionGun_Rifle 引用；只有显现函数和噪声共享。枪体主模型、握持偏移与 MuzzleLocalTransform 保持对应。动画、音频、CameraShake、Bullet、Cue、弹体及武器 VFX 仍分别属于各自武器，不能重新引用 RepairGun 专属资源。

RepairGun 的成功射击 SoundWave 为 `/Game/Weapons/RepairGun/Audio/S_RepairGun_Fire`，由 `BP_RepairGun.FireSound` 配置；`UGA_Shoot` 在真实枪口世界位置播放。空弹音效应使用独立字段/反馈链，不得复用实弹 `FireSound`。
`AFirearm` 的空弹配置为 `DryFireSound` 及独立 Volume/Pitch Multiplier。RepairGun 使用 `/Game/Weapons/RepairGun/Audio/S_RepairGun_DryFire`；仅在当前弹匣为 0、`ConsumeRound()` 失败时播放。

`AFirearm::FireCameraShake/FireCameraShakeScale` 是每把枪独立的纯视觉冲击配置，不得与改变 Controller Rotation 的真实 `AddRecoil` 合并。RepairGun 使用 `/Game/Weapons/RepairGun/Effects/Camera/CS_RepairGun_Fire`，当前为 0.14 秒高频短冲击、Scale=1.0。
RepairGun 当前为单独验收震屏而在武器蓝图覆盖 `RecoilPitch/RecoilYawMin/RecoilYawMax=0/0/0`；这是武器数据配置，不代表公共后坐力系统已删除。
2026-09-04 爆炸弹调试期间，`AFirearm::bEnableViewRecoil` 默认临时设为 `false`，`UGA_Shoot` 仅在该开关开启时调用 `AddRecoil`；原有 Pitch/Yaw/Damping 数据均保留。`FireCameraShake` 调用链未关闭，恢复实际视角后坐时只需把该开关改回 `true`。

FEAT-074 起，玩家枪口统一由 `AFirearm::GetMuzzleWorldTransform()` 解析：优先 SkeletalMesh 的命名 Socket，其次 StaticMesh 的命名 Socket，最后使用 `MuzzleLocalTransform * ActorTransform`。因此纯静态枪模也必须在武器 BP 配置正确的 `MuzzleLocalTransform`，不得退回相机位置伪造枪口。

FEAT-080 起，`UEquipmentManagerComponent::OnCurrentEquipmentChanged` 在首次装备和切枪后广播；Combat HUD 依靠它切换 Firearm 数据源。切角色销毁旧 Pawn 前必须解绑 Equipment/Ammo 委托，禁止 UI 每帧轮询。

FEAT-080 session251 起，玩家枪械可独立配置 `ReloadAbilityClass`；`GrantAbilities/RevokeAbilities` 与主/副射击一样按 Handle 授予回收。RepairGun 配置共享 `BGA_Reload`，由 `Input.Weapon.Reload` Gameplay Event 触发；当前换弹即时完成并沿用 `OnAmmoChanged` 更新 HUD。

2026-09-04：AFirearm.StaticMeshOverlay 原生组件已删除，三枪 Blueprint 的旧壳引用均已清空并重新编译。三个 Outline Mesh 与 M_EquipmentOutline 均无外部引用后删除。
| `Source/TheManTest/Public/Weapons/_Shared/Firearms/Bullets/BulletBase.h` | CollisionSphere(QueryOnly) + BulletMesh + ProjectileMovement；`Damage`(SetByCaller 传入 HitEffectClass) / `HitEffectClass` / `bDestroyOnHit`；`InitBullet(发射者, SourceASC)`(忽略发射者防自撞) / `ProcessHit()` BlueprintNativeEvent。`ABulletBase` 构造函数默认把 HitEffectClass 设为共享 `/Game/Weapons/_Shared/GAS/Effects/GE_BulletDamage`，新建子弹无需重复配置，特殊子弹仍可覆盖/清空。FEAT-073 起，玩家/非敌方弹体有效命中 `AEnemyBase` 时统一调用 `ReactToProjectileHit`，穿透判定优先。 |
| `Source/TheManTest/Public/Weapons/RepairGun/Bullets/RepairGunBullet.h` | 环境命中保持指数膨胀（e^(Rate×t)）与危险区压制；敌人命中施加 `SlowPercent`/`SlowDuration`（默认40%/2.5秒）后立即销毁。连续命中刷新时长、不叠加强度。 |

`ABulletBase` 提供可配置 `ImpactCueTag`，有效碰撞时由攻击者 ASC 执行，但基类不绑定任何具体枪械表现。RepairGun 子弹默认使用 `GameplayCue.Weapon.RepairGun.Impact`，对应 `/Game/Weapons/RepairGun/GAS/GameplayCues/GC_Weapon_RepairGun_Impact` 与 `/Game/Weapons/RepairGun/Audio/S_RepairGun_Impact`；Gameplay Cue 资产名中的 `Weapon_RepairGun_Impact` 必须完整匹配 Tag 层级，确保 Asset Registry 的 `GameplayCueName` 可被运行时管理器发现。环境与 Enemy 都播放同一 RepairGun 反馈，Phantom 穿透不触发。命中 WAV 已离线压缩/软限幅，平均响度较原文件提高约 6.9dB、峰值为 -1dBFS，Cue `VolumeMultiplier` 保持 1.0，避免把近满幅瞬态用超大倍率直接推入总线限幅。Projectile 和当前 Hitscan 均汇入 `ProcessHit`。由于射击 Ability 为 `LocalOnly` 且 Projectile 可在 Ability 结束后才命中，命中反馈必须用 `InvokeGameplayCueEvent(Executed)` 立即本地执行；不得使用依赖 Authority/Prediction Key 的延迟 `ExecuteGameplayCue` 队列。

`UGCN_ImpactFeedbackBase` 对所有命中目标统一播放 `ImpactEffect`，不再按 Character 切换武器 Niagara；角色专用受击表现由目标自身的 `HitReactionCueTag` 负责。武器贴花仍只生成在环境表面。电击枪 Cue 使用 `GameplayCue.Weapon.ElectricGun.Impact`、`NS_ElectricGun_LaserImpact`（源 `NE_VFX_Projectile_Impact_Laser_2`）与紫色贴花（1.1）；其枪口使用 `NS_ElectricGun_LaserMuzzle`（源 `NE_VFX_Muzzle_Laser_Burst_2`）。Laser 依赖迁移后曾有 11 个材质实例的 `Main_Texture` 变成空引用并露出矩形 Sprite 面片，现已恢复为 `/Game/Weapons/ElectricGun/Effects/Textures` 下各自的 owner-local 贴图；后续整理依赖时必须校验材质实例参数值，不能只检查包是否存在。爆炸枪 Cue 使用 `GameplayCue.Weapon.ExplosionGun.Impact`、Physical Impact 1 与黄色贴花（2.0）。两个 Cue 的资产名与 Asset Registry `GameplayCueName` 必须完整匹配对应 Tag。

抛射体的根 `CollisionSphere` 必须保持 `Movable`；`ProjectileMovementComponent` 移动的是根碰撞组件，仅把子级 `BulletMesh` 设为 Movable 不足以让 Actor 飞行。若根球体为 Static，PIE 会报告 `CollisionSphere has to be 'Movable'`，子弹将停在生成点，直到其他物体碰到它才触发命中逻辑。

## 武器资产目录约定（FEAT-052）

- 多种武器复用的资源放在 `/Game/Weapons/_Shared/`，并按 `Mesh`、`Material`、`Textures`、`Animations`、`GAS` 等类型继续分层。
- 某把武器专属的资源放在 `/Game/Weapons/<WeaponName>/`，按 `Blueprint`、`Mesh`、`Material`、`Textures`、`Animation` 分层；不要把材质或贴图放进 `Mesh`。
- 通用弹体：`/Game/Weapons/_Shared/Meshes/SM_Shared_Bullet`，材质为 `/Game/Weapons/_Shared/Materials/MI_Shared_Bullet` 与 `M_Shared_Bullet_Accent`。
- RepairGun 科技泡沫弹体：`/Game/Weapons/RepairGun/Meshes/SM_RepairGun_Bullet`，专属材质为 `/Game/Weapons/RepairGun/Materials/MI_RepairGun_Bullet`。

## 武器动画层字段

`Equip()` 只用武器 BP 上的 `EquipmentAnimLayerClass` 操作角色 Mesh 的现有 AnimInstance：

FEAT-077 起，FPS 玩家只对 `ArmsViewMesh` Link/Unlink 武器层和播放装备 Montage。`CharacterMesh0` 不再重复切换武器层，而是在完整身体 locomotion 之后从 `ArmsViewMesh` Copy 最终上半身 Pose；非 FPS `ACharacter` 仍操作 `GetMesh()`。

- **`EquipmentAnimLayerClass`**（`LinkAnimClassLayers`，链接武器层）：必须与宿主使用同一 Skeleton，并实现 `ALI_WeaponAnim`。
- `EquipmentAnimClass` 与 `SetAnimInstanceClass` 路径已删除；武器不得整体替换角色基础 AnimBP。需要专属跑跳的特殊武器应把状态机放进自己的 Linked Anim Layer。

## FEAT-042 上半身武器层（session70）

当前项目使用 `AFPSCharacterBase + 角色专属 AnimBP + ArmsViewMesh` 路线：

- `EquipmentAnimLayerClass` 保持原职责：由 `AEquipmentBase::Equip()` 链接到 `AFPSCharacterBase::GetArmsMesh()`，也就是独立 FP viewmodel `ArmsViewMesh`。
- Current (FEAT-077, verified 2026-09-19): link EquipmentAnimLayerClass only to ArmsViewMesh. CharacterMesh0 retains body locomotion and copies the completed arms upper-body pose. This supersedes session70 dual linking.
- `AFirearm::Equip()` 在基类完成 Link 后，给 `ArmsViewMesh` 和 `GetMesh()` 上已链接的 `UFirearmAnimInstance` 都写入 `MuzzleLocalTransform`。
- CharacterMesh0 casts the complete hidden shadow; LegsMesh follows it. ShadowBodyMesh and ShadowUpperBodyMesh remain empty deprecated components.
- Body uses upper-body Copy Pose; weapon switching links/unlinks EquipmentAnimLayerClass on the arms host only.

## 2026-08-04 装备显现与统一动画层

- 2026-09-04 起，显现移至 `UEquipmentEquipEffectComponent`：MID 参数 `Amount (S)`，0.5 秒 cubic Hermite 1→0，首切线 -5.434987；只在效果期间 Tick，结束/卸下恢复原材质。
- FPS 角色装备/卸下时，武器 Linked Anim Layer 链接到角色所有 SkeletalMesh AnimInstance；Shadow/Legs 即使是 Leader follower 也保持同一最终 AnimClass 架构。FEAT-075 最终由隐藏的完整 `CharacterMesh0` 直接 CastHiddenShadow；重复 ShadowBody/ShadowUpperBody 为空。shadow-only 静态枪体附着 `CharacterMesh0` 的 `GripPoint`，不可附着已弃用 ShadowBody。

## 通用装备特效所有权（2026-09-04）

- 运行时代码：Public/Private 下 `Weapons/_Shared/EquipmentBase/Effects/EquipmentEquipEffectComponent.h/.cpp`。由 AEquipmentBase 构造默认组件 EquipEffect；适用枪械、近战武器及其他装备。
- 共享资产根：`/Game/Weapons/_Shared/Equipment/Effects/Equip/`，只保留 Materials/Functions/MF_EquipmentEquipDissolve 和 Textures/T_EquipmentEquipNoise。旧共享表面仅剩 RepairGun 使用，已迁回 RepairGun/Materials/M_RepairGun_Rifle；描边材质已删除。
- ElectricGun/ExplosionGun 材质实例引用各自 owner-local 科幻表面主材质，保留共享溶解函数。RepairGun 的 M_SCFR_BaseMat 在原图上接同一函数，不改原纹理与配色。
- 自定义装备材质必须接共享函数和 Amount (S) 才参与显现。组件只从原材质创建 MID，显现期间保留材质全部原参数；未接入的槽保持原样，不再替换为通用灰色表面。原备用路径会将 Outline 壳变为不透明灰壳，现已删除。
- 装备动画仍按各装备 EquipMontage / EquipmentAnimLayerClass 独立配置；bPlayEquipAnimation 可在蓝图开启，共用 VFX 不依赖动画内容。
- 测试：EquipmentEquipEffectTests.cpp / TheManTest.Equipment.SharedEquipReveal 验证三枪 MID 的父材质与原表面完全相同、实际渲染、取消/结束恢复、普通装备接入及可选动画；不兼容材质不得出现灰色备用表面。CombatHUDTests 的切枪测试等待实际显现结束，避免加载卡顿时 wall-clock 等待早于游戏时间完成。


## 2026-09-17 LaserRifle GAS ADS

FEAT-107: /Game/Weapons/LaserRifle owns BP_LaserRifle (ALaserRifle), source-exact SM_Laser_Rifle_01 appearance (outline removed by FEAT-121), blue Burst2/Impact2/tracer/decal and first-person layer. Added to MaintenanceWorker equipment cycle after original three guns. Firearm ADS is opt-in (bSupportsADS false by default), exposing AimSightLocalTransform, ADSFieldOfView60, movement.9/sway.3 and hip/ADS fire animation weights. Source projectile damage8, speed75000cm/s,lifetime.25s, shot interval.12s. Primary uses existing BGA_Shoot, secondary BGA_LaserRifle_Aim; reload existing shared ability. ALaserRifle now only enables ADS by default; the former source outline component was removed by FEAT-121. Source content/config/uproject560 hashes unchanged. Initial graph duplicate lost inherited asset overrides; owner helper remaps original template graph-node paths to duplicated GUIDs before six child overrides. Hip and ADS renders plus cold/actual fire/impact tests passed. Old vendor folder contains only empty dirs and cleanup is tool-policy-blocked; not runtime dependencies.

## FEAT-110 Executive impact slow rifle (2026-09-17)
AExecutiveSlowRifle and AExecutiveSlowRifleBullet live under Weapons/ExecutiveSlowRifle. Executive InitialEquipmentClasses starts BP_ExecutiveSlowRifle then original Repair/Electric/Explosion. Exact Executive Lobby sniper and both materials reused. Owner first-person template uses support-hand TwoBoneIK relative to hand_r; weapon attaches directly to hand_r because shared GripPoint follows the support hand. Owner Equip preserves full shadow mesh component transform. Final attachment values are in Task04/weapon-display-handoff.md; they are gameplay transforms, never copy blindly into Lobby CAT presentation. Blueprint bullet SlowRadius300cm/SlowPercent.5/SlowDuration3s; direct Damage0 default and inherited configurable damage path remains. No new audio/VFX. Development build, physical-input/impact/old-gun PIE and cold references passed.


## 2026-09-17 follow-up integration

FEAT112/115: Infiltrator now uses owner BP_InfiltratorLaserRifle child of BP_LaserRifle with generated UV texture/MI_InfiltratorRifleSkin; original Worker Laser asset untouched. Firstperson animationpose revision116 deferred: existing base logic/animations inherited. Executive gameplay BP_ExecutiveSlowRifle slot0 HiddenHologram; Lobby uses dedicated BP_ExecutiveRifle_LobbyVisual child under Characters/TheExecutive/Lobby/Blueprint, slot0 M_ExecutiveSlowRifle_Hologram (renamed and moved by FEAT122). Do not bind Lobby back to gameplay CDO material or red rings disappear. Cosmetic owner GC_Weapon_ExecutiveSlowRifle_Fire/Impact paths registered; impact slow remains impact-once300cm50percent3sec refresh.

## FEAT-122 weapon resource ownership (2026-09-17)

FEAT-123 naming follow-up: all concrete weapon root Blueprints use `BP_<Weapon>`, bullet Blueprints use `BP_<Weapon>_Bullet`, and root static meshes use `SM_<Weapon>`. Executive materials/effects use the full `ExecutiveSlowRifle` owner, including native `ExecutiveSlowRifleEnergyFX` and `GCN_ExecutiveSlowRifleEnergy`. Infiltrator skin assets use `InfiltratorLaserRifle`. Lobby display child is `Characters/TheExecutive/Lobby/Blueprint/BP_ExecutiveSlowRifle_Lobby`. First-person animation asset names use `FirstPerson` instead of mixed FP/implicit viewpoints. Asset paths in source/tests/scripts have been updated; complete mapping: archive/FEAT-123-asset-rename-map.json.

Executive gun resources belong to `/Game/Weapons/ExecutiveSlowRifle`: `Meshes/SM_ExecutiveSlowRifle`, `Materials/M_ExecutiveSlowRifle_Surface`, `Materials/M_ExecutiveSlowRifle_Hologram`, and eight `Textures/T_ExecutiveSlowRifle_*` assets. Lobby display Blueprints consume these weapon resources; combat assets have no dependency on the Lobby folder. Gameplay keeps its hidden hologram override and Lobby keeps the visible hologram override. The Laser smoke texture is `/Game/Weapons/LaserRifle/Effects/Textures/T_LaserRifle_CloudDissolve_BaseColor`; its vendor-name predecessor has been removed with references fixed. Gun outlines remain prohibited under FEAT121.

Saved loadouts, cold-read 2026-09-19: MaintenanceWorker = BP_ExplosionGun / BP_RepairGun / BP_ElectricGun; Infiltrator = BP_InfiltratorLaserRifle; TheExecutive = BP_InfiltratorLaserRifle / BP_ExecutiveSlowRifle, in that order. Supersedes the earlier Executive single-gun audit. Preserve saved user configurations; do not restore historical FEAT110 inventory. No loadout asset changed during documentation correction.

## FEAT-121 Remove weapon outlines (2026-09-17)

All gun outline shells are prohibited by user instruction, including future imports. Removed ALaserRifle.SourceOutline and the owner-local SM_LaserRifle_Outline/M_LaserRifle_Outline assets after zero-reference checks. BP_LaserRifle and BP_InfiltratorLaserRifle compiled/saved. Blink now treats gun meshes uniformly; removed the component-name outline special case and unused visibility snapshot. Scan highlighting and UI borders are separate and preserved. Ten gun/display configurations checked for component/mesh/material outlines; all are absent. Evidence: D:/UnrealWork/WeaponOutlineRemoval.


## FEAT-125 Infiltrator open reflex sight (2026-09-18)

BP_InfiltratorLaserRifle now owns `Meshes/SM_InfiltratorLaserRifle`: original body geometry/UVs, open reflex frame replacing the closed optic. Six material slots: Surface, OpticMetal, Paint, Glass, Emitter, Rubber; resolve by slot name because render section order differs. Glass uses translucent unlit tint with Opacity=.025; lit reflection previously produced a white ADS window. Skin separates paint/metal response with subtle machining normal and muted dark finish. Original LaserRifle mesh/material and all animations/ADS transforms unchanged. Infiltrator Lobby consumes the child CDO and picks up this mesh. Final ADS/fire/release/re-entry/cycle/Blink restoration and cold evidence: D:/UnrealWork/InfiltratorSightRevision.

## FEAT-126 Infiltrator finish and framing (2026-09-18)

Existing owner mesh retains original rifle silhouette with0.7mm bevels/weighted normals. Reflex geometry width is72% and height62% of FEAT125, with transparent unlit glass preserved. AimSightLocalTransform translation now (0.000237,-38,26.613439); rotation unchanged. This accounts for the smaller physical optic and increases eye relief; existing ADS camera alignment remains authoritative. No global viewmodel transform/FOV change. Final actual sight width is approximately15% of screen; prior was approximately27%.

Owner BaseColor/ORM/Normal textures provide differentiated paint/metal roughness and surface microstructure; normal imports flip green for Unreal, ORM uses linear mask compression. Body base color uses0.62 material scale after outdoor exposure review. OpticMetal roughness0.42, dark tint; rubber high roughness, nonmetal. Six named material slots and Blink restoration preserved. Independent first-person layer/animations are now owner-local (see12-anim-blueprint); prior FEAT116 deferral no longer describes this weapon. Worker Laser is unchanged. Infiltrator Lobby continues consuming the existing child mesh/materials. Sources: D:/Blender Projects/InfiltratorFinish; technical/PIE/cold evidence: D:/UnrealWork/InfiltratorFinish.

## FEAT-127 cropped Infiltrator framing (2026-09-18)

Supersedes FEAT126 hip translation only: final authored camera displacement from original source is(-8,12,-10)cm, with the same-10deg yaw. Gun orientation is preserved against126 at all196 sampled frames; only translation differs(-8,+9,-3)cm. Left support advances5cm along weapon+Y. Since GripPoint belongs to hand_l, StaticMesh.RelativeLocation.Y changes11.050698->6.050698, AimSightLocalTransform.Y -38->-43 and MuzzleLocalTransform.Y61.317595->56.317595. These inverse compensations retain the same gun/right-hand relationship and ADS optical alignment. Shared socket/skeleton and other weapons unchanged. Evidence: D:/UnrealWork/InfiltratorFraming/orientation-contact.json, visual.json and cold.json. User explicitly clarified lower-RIGHT corner and asked to retain126 orientation.

FEAT127 final user-approved direction change: add5deg clockwise camera-space cant around right-hand pivot to hip poses (camera-X -5deg before existing yaw-10deg). This supersedes the prior exact-orientation preservation request. Framing translation/support-hand compensation unchanged. Existing ADS alignment returns the sight upright. Six owner pose clips updated; evidence D:/UnrealWork/InfiltratorCant.


## FEAT128 approved Infiltrator pose (2026-09-18)

FEAT128 supersedes127 attachment/framing values. Original gun geometry/materials retained; BP_InfiltratorLaserRifle.StaticMesh scale(.715,.715,.715), relative translation(1.807631,-16.668488,1.862728). AimSightLocal translation(1.807658,-48.551442,12.508291), muzzle uses the same .65 scaling and mesh-relative compensation from the126 baseline; existing rotations unchanged. Shared GripPoint on hand_l remains unchanged. Both wrists now target actual gun coordinates left(6,14,-7), right(-5,-12,-16)cm, rather than old incorrect support/stock placement. External baseline/author script and contact-verification.json in D:/UnrealWork/InfiltratorApprovedPose are authoritative.

FEAT128 latest feedback: level reference gun cross-axis against world-up using fixed3.504deg roll correction around gun forward direction; retain authored yaw/pitch/origin. Left contact is now(6,14,-11)cm. Mesh relativeZ4.722728 (X/Y unchanged), matching aim/muzzle localZ compensation. Six owner clips and weapon BP updated. Authoritative scripts/checks D:/UnrealWork/InfiltratorLevelPose supersede prior ApprovedPose values.


## FEAT129 Infiltrator aiming laser and finish (2026-09-18)

BP_InfiltratorLaserRifle has one owner-local UInfiltratorAimingLaserComponent (Weapons/InfiltratorLaserRifle/Effects). It consumes existing bIsADS and local-owner/visibility state in PostUpdateWork; no additional input, GAS ability, damage or firing path. Runtime-only collision-free Cylinder/Plane components render a depth-tested red beam and surface-aligned point. Visibility traces from view then emitter prevent through-wall lines; eye-to-emitter obstruction hides beam, no hit hides dot, release/unequip/destruction clear presentation. Editable emitter mesh position(-3.8,32,4), range10000cm, beam radius.09cm, dot radius1.2cm. Materials belong to Effects/AimingLaser/Materials. Original Laser has no component.

M_InfiltratorLaserRifle_Skin now uses new Textures/T_InfiltratorLaserRifle_AO baked from existing bevelled geometry at4cm local distance. Coating roughness .60 plus variation versus metal .29 plus variation; modest color desaturation and cavity tint improve rear separation. Original mesh, BaseColor/ORM/Normal textures, poses, scale and blue-cyan unlit lens(.025 opacity) retained. Evidence D:/UnrealWork/InfiltratorAimLaser; cold31 assets and rendered actual PIE verified.

FEAT129 latest feedback supersedes side-emitter/lens/finish values above: laser now starts at GetMuzzleWorldTransform (same source as firing); EmitterMeshLocation removed. Lens tint(.015,.25,.65), opacity.22 for visibly blue transparent glass. Body color scales metal.48/coating.34, cavity multiplier.60..1, desaturation.03, base roughness metal.27/coating.52 plus .14*ORM.G. Paint/OpticMetal/Rubber also darkened. Accepted128 pose unchanged. Latest evidence D:/UnrealWork/InfiltratorDeepFinish.


## FEAT130 ivory compact optic and ADS foreground focus (2026-09-18)
Owner SM_InfiltratorLaserRifle now has seven named slots: prior six plus OpticFrame. Compact continuous protective frame replaces tall posts; lens center mesh(0,-8.13,14.02)cm and AimSightLocalZ14.747028. Rear stock and armor both ivory, charcoal mechanics/aged brass details. Original body geometry/UV, accepted128 pose, scale and muzzle remain. Clear glass uses edge-weighted opacity .012..122, small red reticle. FEAT129 flat blue .22 lens and orange colors superseded.
BP_InfiltratorLaserRifle adds one owner UInfiltratorAimFocusComponent (Public/Private Weapons/InfiltratorLaserRifle/Effects). Runtime transient PostProcessComponent and MID consume local ADS/visibility; depth-limited8-neighbor2.5pixel blur only nearer than physical lens minus5cm,16cm ramp,14/s ADS blend. Near wall cutoff+25cm disables effect; release/hidden/unequip/EndPlay clean up. PP material Effects/AimFocus/Materials/M_InfiltratorLaserRifle_AimFocus, after-tonemap, no custom-depth/stencil or global camera changes. Seven slots must restore after Blink. Character lamps still off. Build/actual PIE/cold verified, evidence D:/UnrealWork/InfiltratorIvoryOptic.


FEAT130 textured-finish correction: Skin/Paint now sample existing BaseColor again; source luminance/paint-metal median normalization restores markings and wear, retaining original UV/Normal/AO. Prior constant palette-only graph superseded. Ivory(.50,.475,.425), charcoal(.055,.052,.047); local-space mechanical end masks add brass. VertexInterpolator carries PreSkinnedPosition to pixel-stage Custom expressions. OpticMetal now dark aged brass(.065,.048,.025), rough.52/metallic.5/spec.28; OpticFrame(.018,.019,.018), rough.48/metallic.22/spec.28. Shader scripts/current visual evidence D:/UnrealWork/InfiltratorTexturedFinish. No mesh/pose/focus/laser changes.


2026-09-19 user laser visibility follow-up: BP_InfiltratorLaserRifle SCS aiming component BeamRadius=0.27cm (diameter0.54cm), DotRadius=2.4cm; M_InfiltratorLaserRifle_AimingBeam BeamOpacity=0.9. Supersedes0.09/1.2/0.75. Remains ADS-only, muzzle-origin, depth-tested, with unchanged obstruction/release behavior. Tripo body appearance and original lens/reticle remain intact. Evidence D:/UnrealWork/InfiltratorLaserVisibility.

2026-09-19 newer Tripo download: SM_InfiltratorLaserRifle.fbx (1).glb replaces the existing TripoBaseColor texture content. Source positions/indices/UVs match previous download; normals differ. Production geometry, original GeometryNormal, all seven material assignments and optics retained. Aiming radius0.27cm, dot2.4cm and opacity0.9 remain unchanged. Same texture asset path; do not rerun older texture import scripts. Evidence D:/UnrealWork/InfiltratorTripoUpdate.

## Current A1553 compact rifle replacement (2026-09-19)

User selected model5 from the supplied A1553 GUN package and authorized replacement preview. BP_InfiltratorLaserRifle now uses owner Meshes/SM_InfiltratorLaserRifle_Compact, Materials/M_InfiltratorLaserRifle_CompactSurface and four Textures/T_InfiltratorLaserRifle_Compact{BaseColor,Normal,Roughness,Metallic}. Names are purpose-based; no vendor directory imported. Three slots Surface/Glass/Emitter supersede the old seven-slot assumption. Original Glass/Emitter materials and thicker laser .27/2.4/.9 remain. Previous production mesh/materials retained as rollback resources, not deleted.

2026-09-19 correction: COLT Pack2/5 was a mismatched atlas and must not be restored. The supplied GUN.jpg white model5 is authoritative. Source Blender5 retains only a UV-checker material; missing consolidated texture/material-region data requires reconstruction. Current four Compact maps are baked from explicitly reconstructed white shell, dark grips/rails and steel regions at D:/Blender Projects/A1553TextureFix. Fine original decals/microdetail are not restored. Existing topology, UV, optic aperture, lens/reticle, mounts and poses remain unchanged in this texture-only fix. Surface samples baked BaseColor/Normal/Roughness/Metallic directly; former .35 metallic multiplier and .42 roughness floor removed. Normal remains linear, normal compression, green flipped. Do not rerun A1553Install/build.py or author.py: they still point to the rejected atlas; use A1553TextureFix/author.py for surface reproduction.

Six owner nonadditive pose clips adapt wrists to vertical grips. Canonical pre-bake mesh contacts: left(6,9,-8), right(-5,-18,-8)cm; hand rotations in gun space -60/-55deg around X. Existing bone lengths/translations preserved; arms solved externally. Socket compensation rotation baked into mesh so BP component rotation stays identity and shared shadow logic remains valid. Historical pre-normalization mesh translation(1.807631,-14.829653,-5.855145),scale .715; superseded by identity mount below. Aim/muzzle transforms recomputed from source optic/barrel coordinates and socket compensation; do not restore old Tripo transforms. Historical install scripts: D:/UnrealWork/A1553Install and D:/Blender Projects/A1553Install; current mount/pose authority is GunFramingApply below. Do not rerun earlier appearance/pose author scripts. No C++ changes or damage/input changes.

2026-09-19 source-value follow-up supersedes TextureFix paint parameters: use D:/Blender Projects/A1553SourceMaterial and D:/UnrealWork/A1553SourceMaterial/author.py. Original source silver sRGB193/186/177, Metallic1, Roughness.2 and black sRGB71/71/71, Metallic0, Roughness.6 now baked into current three Compact surface maps. Existing normal and geometry unchanged. TextureFix face-region reconstruction remains provisional; exact source decal/normal/material assignment recovery is still unresolved. Older TextureFix full author/rebuild must not overwrite current source values.

Final save note: material recompilation marked CompactSurface dirty. Saved compiled material without graph edits. Final scope is three textures plus compiled CompactSurface,41 protected owner packages byte-identical. editor-final.json confirms clean content/maps and PIE ended.

## User-requested camera/gun framing authoring class (2026-09-19)

Generic authoring tool: /Game/Actors/Tools/GunFraming/Blueprint/BP_GunFraming (CameraActor), replacing the old weapon-owned BP_InfiltratorLaserRifle_Framing. FOV90; movable, noncolliding Gun attached directly to CameraComponent. No skeletal components. Gun material overrides cleared so swapping StaticMesh uses that mesh's materials. Select Gun by name, because CameraActor also has an editor camera visualization mesh. Adjust Gun relative Location/Rotation in the camera view; apply instance changes to Blueprint and save the target before pose fitting. The user-saved TestMap preview instance is preserved, renamed Gun Framing and migrated to the new class. Its Gun transform compensates for mesh normalization, preserving the user's visible target despite changed numeric values. This class authors a target; it does not automatically solve another weapon's animation. Old redirectors and their residual disk package removed after confirming no referencers; backup outside project.

User-saved gun framing applied to the six owner Still/Idle/Run/JumpStart/JumpLoop/JumpEnd clips. Character ArmsViewMesh base transforms, shared skeleton/socket, animation graphs and additive Fire/FireBase remain unchanged. Same-skeleton local-rotation editing only; no retargeting. The old mesh installation translation/scale is baked into Compact vertex positions, preserving UV/topology/material slots. BP_InfiltratorLaserRifle.StaticMesh is now Location(0,0,0), Rotation(0,0,0), Scale(1,1,1). Aim and muzzle transforms retain their correct root-space coordinates, not zeroed. Original Glass/Emitter and laser .27/2.4/.9 retained; no surface texture/material edits in this batch.

Current root-space Aim translation(2.577159,-35.347465,-31.476389), Muzzle translation(2.577159,-4.294442,8.885098); both rotations xyzw(.353553385,-.353553385,.612372414,.612372414), scale1. These are functional optical/barrel coordinates and must not be zeroed with the mesh mount.

Checkpoint54e1980 protects the user-saved target and prior assets. Final changes uncommitted; no push/package. Authoritative pose/normalization scripts: D:/UnrealWork/GunFramingApply; external mesh source D:/Blender Projects/GunFramingApply/CompactRifleNormalized.blend and FBX. Do not rerun old A1553Install or earlier pose authors: they restore obsolete mounts/poses. Surface authority remains A1553SourceMaterial; original decal/microdetail/material-region recovery remains unresolved. User explicitly disabled secondary-screen locking: do not restart window-position loops or force the editor back to secondary. Current result preview opened on primary.


## Canonical mesh axes and explicit attachment (2026-09-19)

2026-09-19 user-approved mesh-axis correction: removed the previously baked60-degree X mount rotation and baked installation translation from Compact mesh vertices. Canonical mesh is UE +Y forward/+Z up at the existing physical size; UV/topology/material slots preserved. BP_InfiltratorLaserRifle.StaticMesh remains identity. Explicit AEquipmentBase.EquipAttachmentTransform now stores socket-to-weapon placement; default identity preserves other guns. EquipmentManager applies it after first equip and subsequent switching; shadow proxy composes it after existing legacy mesh handling. Firearm passes muzzle * attachment into the socket-space animation aim source. Infiltrator attachment translation(1.807631,-14.829653,-5.855145), quaternionxyzw(.5,0,0,.866025404), scale1. This replaces hidden geometry compensation; do not bake it into the mesh again. AimSightLocalTransform/MuzzleLocalTransform converted by inverse attachment; world optical and barrel placement preserved. Generic BP_GunFraming and the saved TestMap instance converted to preserve the user's target. No pose, character, skeleton, shared socket, texture or material changes in this batch.

Evidence D:/UnrealWork/GunAxisFix: Development Editor Win64 build.log Succeeded; visual.json18 actual PIE checks pass; capture.json four actual gameplay views; cold.json/pose-cold.json/preview-cold.json/attachment-cold.json pass, including five other weapon identity defaults and1568 unchanged pose samples. runtime-attachment.json confirms visible actor and shadow use identical explicit attachment. framing-match.json camera-relative target error .000998cm/.012786deg. External source D:/Blender Projects/GunAxisFix/CompactRifleCanonical.blend, final FBX, geometry.json (UV preserved, inverse/recomposition checked). Canonical-Side.png is a neutral-gray axis inspection, not a final material render. ThreeWeaponBaseline passes. Historical ThreeWeaponPIESwitch assumes Repair/Electric/Explosion order; current worker starts Explosion, so its fixed-order assertions fail; use switch-runtime.json for explicit current-order and Infiltrator-away/back verification.

Checkpoint3587906 preserves previous framing result. Final work remains uncommitted; no push/package. Generic tool stays /Game/Actors/Tools/GunFraming/Blueprint/BP_GunFraming. User screen unlock remains authoritative: no automatic secondary placement/locking. GunAxisFix author/solution supersede GunFramingApply mesh/mount authoring; six current poses still from GunFramingApply. Original full source texture/UV detail recovery remains unresolved and outside this axis correction. Current numeric optical transforms are in D:/UnrealWork/GunAxisFix/solution.json; older root coordinates in archive are historical.


Final verification: switch-runtime.json passes all seven attachment checks using the actual worker inventory order and temporary Infiltrator->RepairGun->Infiltrator switching. Test actors/inventory changes were PIE-only. Blueprint editor compile/save and clean-package check completed; PIE ended. Native gameplay and neutral-gray axis preview displayed on primary.


## Scoped rifle candidate, 2026-09-19

2026-09-19 scoped rifle candidate for user comparison. User supplied TMIIR /Game/SCI_FI_WEAPON_PACK/SCF_Rifle/BluePrints and screenshot Screenshots/???? 2026-09-19 140953.png (black/gold rifle with red-dot optic). Source BP_Weapon contains SK_SCFRIFLE plus independent SM_SCFR_Sight at relative(0,16,15.325822)cm, rotation0/scale1. Exported reference-pose body and sight externally, combined them into SM_InfiltratorLaserRifle_ScopedCandidate with native +Y-forward/+Z-up axes and ~70.75cm length. Preserved five original material slots: receiver, stock, barrel/magazine, sight and sight glass. Migrated only24 original material/texture dependencies using AssetTools; renamed into owner Materials/Textures as *_InfiltratorLaserRifle_Scoped*. No source skeleton, rigs, animation or vendor gameplay Blueprint imported. Source content not edited;24 material/texture source hashes verified. Temporary vendor path removed from registry and disk.

Candidate mesh /Game/Weapons/InfiltratorLaserRifle/Meshes/SM_InfiltratorLaserRifle_ScopedCandidate is selected in /Game/Actors/Tools/GunFraming/Blueprint/BP_GunFraming and its existing saved TestMap Gun Framing instance. Gun remains directly camera-attached, movable, collision off, zero material overrides; FOV90. Initial candidate camera-relative location(65,12,-20)cm, scale1; rotation retained from the previous authoring basis as a starting point only. User adjusts Gun Location/Rotation, applies instance changes to Blueprint and saves; wait for their chosen framing before any production replacement or pose fitting. Original production Compact mesh, weapon BP, mount parameters, six arm pose clips and character base transforms remain unchanged. Existing Compact framing stays recoverable from GunAxisFix/solution.json and checkpoint80136b5.

D:/UnrealWork/ScopedRifleCandidate: source-audit.json, migration.json, mapping.json, import.json, preview.json, cold.json, visual.json and actual Candidate-Framing/Candidate-Side renders. External FBX/Blender work: D:/Blender Projects/ScopedRifleCandidate, ScopedCandidate.blend and assembled.json; combined28398 vertices/29849 polygons,5 slots. Cold verifies native dimensions, correct materials, new preview mesh, preserved production Compact, no source/vendor dependencies or directory. Initial headless NullRHI skeletal export hit MeshObject assertion; rendering-enabled commandlet export succeeded. Source material graphs retained; lighting is not matched one-to-one to the supplied black-background render. No C++ changes/build needed for this candidate batch. Checkpoint80136b5 protects previous finished axis correction; final changes uncommitted, no push/package. No secondary monitor placement loop.


## Independent Infiltrator scoped rifle (2026-09-19)

2026-09-19 user authorized duplicating the current Infiltrator weapon and installing the scoped candidate plus its own animations. Created /Game/Weapons/InfiltratorScopedRifle/Blueprint/BP_InfiltratorScopedRifle as an independent duplicate of BP_InfiltratorLaserRifle, retaining its inherited laser firearm gameplay and native aiming components. Candidate mesh and24 source material/texture dependencies promoted via AssetTools to the new owner, with semantic InfiltratorScopedRifle names. Generic BP_GunFraming references follow the moved mesh. Original Compact production weapon packages remain byte-identical. BP_Infiltrator initial equipment now [BP_InfiltratorLaserRifle, BP_InfiltratorScopedRifle] for direct wheel comparison.

User-saved target is camera-relative location(24,18,-23)cm, quaternionxyzw(-.0173242734,.0680859016,-.7160806879,.6944729137), scale1. Captured class and new saved preview instance agree; user replaced the earlier preview actor. Created independent Still/Idle/Run/JumpStart/JumpLoop/JumpEnd, Fire/FireBase, Fire/Equip montages, WalkRun blend space and template/concrete animation layer. Six nonadditive poses fit new rifle contact targets; only local clavicle/upperarm/forearm/hand rotations authored, preserving bone translations/scales, shared skeleton/socket, character ArmsViewMesh base and old gun clips. Same-skeleton editing, no retargeting. New mesh component remains identity; explicit EquipAttachmentTransform handles the socket basis. Source grip solver and exact transforms: D:/UnrealWork/InfiltratorScopedRifle/solution.json and corrected-poses.json. Additive Fire uses its own FireBase and both montages reference the new owner's Fire. No playback dependency on old owner remains.

Original scoped sight and translucent red glass retained in5 material slots. New owner AimFocus/laser material copies isolate changes. AimSightLocalTransform is an eye origin with24cm relief behind the rear optic lens: location(0,-12.7111947,19.0188), yaw90; lens focus point(0,11.2888053,19.0188). Muzzle location(0,44.9,9.4), yaw90. Initial lens-plane eye origin was corrected after actual ADS clipping inspection. Original raised iron sights remain part of source geometry and occupy the lower optic view. User selected hip framing remains intact; no character base offset changes.

Checkpoint3bbb6e9 protects candidate and user-saved framing. Minimal editor helper extension accepts InfiltratorScopedRifle when remapping copied parent animation graph GUIDs; Development Editor Win64 build.log Succeeded. Evidence D:/UnrealWork/InfiltratorScopedRifle: author.json, cold.json(45 registry entries,1568 pose rotations,max quaternion error2.23e-13,zero old-owner references/redirectors,old weapon hashes unchanged), visual.json18 actual PIE checks, four final1920x1080 captures, runtime-attachment.json and framing-match.json(.006471cm/.017209deg target error). Solver wrist error<=1.28e-12cm; clavicle adjustment up to17.264deg without bone-length changes. Final switch-back/ADS-cancellation checks pass in switch.json; final result not committed/pushed/packaged. Source model inspection only in D:/Blender Projects/InfiltratorScopedRifle. Keep user screen unlock: no secondary-monitor placement loop.


Final scoped-rifle handoff: switch.json passes old->new->old->new with independent animation classes and ADS/laser cancellation on switch. Four final native captures reviewed; editor-final.json confirms Blueprint compile/save, zero dirty content/maps and PIE ended. New weapon Blueprint and actual gameplay viewer opened for review on primary. No final commit/push/package. User may select Infiltrator and wheel once to the new second weapon. Source raised iron sights still occlude the lower optic aperture; source geometry retained.
# FEAT-146 current Infiltrator native attachment (2026-09-23)

BP_InfiltratorLaserRifle and BP_InfiltratorScopedRifle now attach to the native `Bip01-R-Hand` with their separately measured hand-to-gun transforms. Their EquipmentAnimLayerClass, FireMontage and EquipMontage use native assets under `/Game/Characters/Infiltrator/Animations/FirstPerson/{LaserRifle,ScopedRifle}`. This supersedes the historical GripPoint/Worker-skeleton playback settings below for these two guns. Mesh component transforms, optical/muzzle transforms, materials and gameplay settings remain unchanged. Infiltrator's saved initial inventory remains silver-only; scoped rifle was added transiently for PIE checks. Other characters/weapons retain their existing skeletons and layers. Authoritative offsets and evidence: `D:/Blender Projects/InfiltratorNativeWeapons/gameplay-installed.json`, `runtime-acceptance.json`, `cold-gameplay.json`; FEAT-146 archive.


## FEAT-149 native role integration (2026-09-23)
Native Worker/Executive presentation BPs live under Characters/<role>/Weapons/<weapon>/Blueprint. Original weapon behavior/components copied, only owner animation layer, montage and hand mount changed. Repair uses SKM_RepairGun, legacy static fallback remains unused. Visible mesh component transform included when deriving hand mount. Owner-local copies avoid altering Infiltrator silver. See archive/FEAT-149-native-worker-executive.md and D:/Blender Projects/WorkerExecutiveNative for evidence.

FEAT-149 user correction: Executive has exactly ONE weapon, the native BP_TheExecutive_ExecutiveSlowRifle. Extra Executive-only InfiltratorLaserRifle/ScopedRifle weapon and animation folders were deleted, not available loadout options. Worker keeps its three guns. Obsolete /Game/Weapons/RepairGun/Meshes/SM_RepairGun deleted and its static-component references cleared in both RepairGun BPs; actual SKM_RepairGun remains. This supersedes earlier six-profile/secondary-Executive framing notes.


## FEAT215 infinite ammo and accepted-shot presentation
AFirearm.HasInfiniteAmmo reads only its owning pawn's local TheManPlayerController debug option; Shipping alwaysfalse. CanFire permits empty magazines while enabled. ConsumeRound returns success without changing CurrentAmmo or SpareMagazineCount; disabled mode retains the original decrement/OnAmmoChanged broadcast. CanReload rejects reload while infinite so no spare magazines are wasted. Counts are never refilled or reset by toggling the option.
Every accepted ConsumeRound invokes protected virtual OnRoundFired once. ExecutiveSlowRifle overrides it to play its existing muzzle energy cue; its old OnAmmoChanged/decreasing-count inference and LastPresentationAmmo were removed because accepted infinite shots do not decrease ammo. Ordinary shooting/reload/impact/damage code and enemy magazines are unchanged. UI lives in existing WBP_DebugMenu, not runtime generated layout.

## FEAT217 universal environment explosion visuals (2026-09-28)
Supersedes historical ExplosionGround tag rules above. FindExplosionGround accepts the nearest upward-facing environment surface within GroundSearchDistance/GroundMaxSlope, excluding character/projectile/GeometryCollection and initial penetration. No actor tag required or prioritized. GCN_ExplosionGunExplosion always spawns its configured selected effect: suitable ground when available/requested, otherwise actual blast origin with impact normal. Environment and enemy effects remain separately configured; current enemy effect is NS_ExplosionGun_EnemyDetonation with bEnemyEffectOnGround=false. Damage/fuse/audio/physics unchanged. Obsolete ExplosionGroundTag constant removed;65TestMap actor tags removed and saved (external actors plus map). No such tags in SpaceStation or remaining Content packages.

## FEAT219 Infiltrator ADS zoom and optic size (2026-09-29)
AFirearm exposes opt-in ADSZoomMultiplier (default0 uses legacy ADSFieldOfView) and ADSLookSensitivityMultiplier (default1). Character UpdateWeaponAim computes 2*atan(tan(originalFOV/2)/zoom) for positive zoom and blends with existing WeaponAimAlpha; Look multiplies both axes by lerp(1,weapon sensitivity,alpha), without overwriting LookSensitivity. Existing release/cancel/switch/unpossess cleanup restores both automatically. Only BP_InfiltratorLaserRifle/ScopedRifle opt into2x/.75. User then requested optic body screen dimensions doubled again at that same world FOV: AimSightLocalTransform translation silver(.769527581,-19.413053553,4.958324651), scoped(0,-.7111947,19.0188), existing yaw90/scale1. Eye relief13.0345/12cm respectively. Hip framing/mesh/muzzle/animation unchanged. Silver lens geometry audited from final canonical mesh, not stale historical focus-position defaults. Build/live53checks/cold verification pass; see FEAT219 archive.

FEAT220 user tuning/repair: silver rifle now uses ADSZoomMultiplier1.5, ADSLookSensitivityMultiplier.5, AimSightLocalTransform LocationY=-20 (other location/rotation unchanged). Accidental ScaleX1.939409 restored to unit(1,1,1); nonuniform aim-transform scale deforms the entire viewmodel through inverse alignment. Black rifle settings unchanged.


## FEAT221 RepairGun airborne range dissipation (2026-09-29)
ARepairGunBullet now records its launch origin in BeginPlay and checks MaxAirborneDistance (default2000cm;0disabled) after ProjectileMovement while unhit, including the same-frame distant-impact path. Range expiry spawns configured world-space DissipationEffect with auto-destroy then removes the bullet without executing its impact cue/damage/foam expansion. bDissipating prevents duplicate feedback. Landed foam is exempt from the airborne range even if moved away; expansion finishes and disables Tick, then starts LifetimeAfterExpansion (default10s,0disabled). Enemy slow and pass-through unchanged. BP_RepairGun_Bullet owns the effect reference and DissipationEffectScale1; current Worker RepairGun already uses that class.
Production effect is /Game/Weapons/RepairGun/Effects/Dissipation/Systems/NS_RepairGun_PixelDissipation, four pixel layers, SystemState Once/.6s, independent unlit grayscale fading material and four textures. This is the UE5.7 reconstruction of the accepted UE58 prototype, not a backward-copied UE58 package. All10 packages are owner-local. Current existing expansion scale7/.5s retained. Tests TheManTest.Player.Weapons.RepairDissipation and RepairFoam plus actual Enhanced Input/cold validation pass; see FEAT221 archive.


## FEAT235 shared damage reaction entry

BulletBase captures actor-local source direction before hit alert can turn the enemy. It requests reaction only after actual health reduction and while target remains alive. ExplosionGunBullet opts out of initial attachment-hit reaction; delayed explosion retains pre-damage target-local direction and calls the same component with Explosion=true. Damage, cues, visibility checks, deduplication and corpse impulses remain in their existing owners. EnemyHitReactionConfig is now required for presentation; old component sequence fields remain serialized only for migration compatibility.


FEAT251 (2026-10-07): ABulletBase.ProcessHit forwards the actual FHitResult and incoming ShotDirection to EnemyBase.NotifyProjectileImpact for non-enemy instigators, once per processed hit. Default virtual is no-op; Void uses it for eye-grab release. Damage/impact effects remain on their existing path; this is not a generic damage-based weak-point inference.


FEAT271 blink surface glow: the eight actual material parents used by InfiltratorLaserRifle and InfiltratorScopedRifle add BlinkGlowAmount(default0) * BlinkGlowColor to existing emissive. DirectionalBlinkComponent creates transient actual-slot MIDs on the held equipment, snapshots original slots, settles the equip reveal while glowing, handles weapon changes, fades by280ms and restores only owned slots. Wrapped MIDs are zeroed even if another effect owns the slot. No gun overlay, outline material, shell or mesh is added. Existing base-color/normal/roughness/opacity graphs are preserved.


## FEAT-273 DisintegrationGun starter (2026-10-10)

`/Game/Weapons/DisintegrationGun/Blueprint/BP_DisintegrationGun` is an independent duplicate of the currently used `/Game/Characters/MaintenanceWorker/Gameplay/Weapons/ExplosionGun/Blueprint/BP_MaintenanceWorker_ExplosionGun`. It temporarily reuses ExplosionGun mesh/materials/bullet and Worker native ExplosionGun animation layer/montages/hand transform. Parameters copied exactly, including native presentation BP magazine30 (generic BP_ExplosionGun is2). No disintegration behavior or default loadout entry yet. Original302 gun packages unchanged. See FEAT273 archive.


## FEAT274 object-only DisintegrationGun (2026-10-10)
Supersedes FEAT273 placeholder behavior. BP_DisintegrationGun now derives ADisintegrationGun, BulletClass null, Profile DA_DisintegrationGun_Profile. Uses OnRoundFired camera Visibility trace while retaining shared GA_Shoot feedback. ADisintegrationEffect accepts only non-Pawn mesh owners explicitly tagged DisintegrationTarget (actor or component); all slots must have profile material mappings. No enemy adaptation or AI/death/health handling. Profile retains4 object mappings,5material assets;30 enemy adaptation copies deleted at user request. Manager/particles/functions reside in weapon-owned Effects. Hit-origin expansion uses original material copies and GBuffer particle color. Optional legacy Niagara collision disabled for UE5.7. Duration2s, particle lifetime2s; effect owns completion and cleanup. Worker default inventory appends gun fourth, original three unchanged. Build passes; blue cube VFX verified; complete gun-to-object PIE acceptance pending.


FEAT274 component/Cue integration (2026-10-10) supersedes the DisintegrationTarget tag entry above. UDisintegrationComponent is BlueprintSpawnable and EditAnywhere on placed objects or Blueprint defaults. Profile, TargetMeshNames (empty=all visible static/skeletal meshes on this actor), Duration, ParticleScale/Density and Enabled are target-owned. Pawn owners excluded. GetConfigurationIssue checks all selected slots; PrepareMaterials is an explicit editor button that clones only selected opaque/masked material graphs and their instance parents, inserts existing dissolve functions, preserves original masks/appearance, reuses deterministic copies and saves the profile/materials. Translucent materials report unsupported rather than changing their blend behavior. Editor-only MaterialEditor dependency.
Gun only traces and calls TryDisintegrate; old gun Profile removed. Component owns deduplication, duration, dissolved state and collision removal. GameplayCue.Weapon.DisintegrationGun.Dissolve invokes presentation on that component; helper ADisintegrationEffect owns manager/material MIDs/Niagara and cleans its tail. Cue cannot apply damage or decide collision/removal. No projectile/GE/health writes. Six new GC assets use exact full Tag names and owned Cue scan paths. Repair expiry now dispatches GameplayCue.Weapon.RepairGun.Dissipation with existing Niagara asset and scale; range/stop/destroy stay in projectile. Cue anchors to surviving owner/worldsettings so deferred first-load cannot be lost when bullet dies.
User explicitly authorized SAVED TestMap acceptance objects (exception to previous transient-only rule): blue cube, orange sphere, SpaceStation vent and seven-piece SpaceStation door, each with instance DisintegrationComponent. Original SpaceStation map/materials/meshes untouched. See FEAT274 for validation.

FEAT274 automatic interiors (2026-10-10) supersede the cube-only prototype. Component bFillInterior defaults true; InteriorResolution(4..24, default18) and fallback InteriorColor remain configurable. PrepareMaterials also reads editor LOD0 render triangles and performs orthogonal parity scans with coincident-hit deduplication; cells with all eight corners supported by at least two axes enter Profile.InteriorShapes keyed by source StaticMesh. Unpaired open-surface scanlines are rejected. FDisintegrationInteriorShape serializes centers/cell size/resolution for cooked runtime access, without enabling CPU access or modifying original meshes. Re-run PrepareMaterials after changing the mesh; missing shape caches report configuration issues. This approximates enclosed static-mesh volumes, not exact boolean capping, masked texture apertures or skeletal deformation. Grid-sized features and open/nonmanifold geometry need visual review.

On hit, CreateInterior spawns only occupied cells in one non-colliding ISM per selected mesh, applies the current component world transform, source Color parameter or neutral configurable fallback, then releases fragments according to the shared hit/time/radius. EndPlay removes every core/manager/Niagara component. No persistent inner mesh actors or per-frame CPU transform updates. GPU WPO handles thinning/rotation/bounded drift/fade. All four saved TestMap fixtures now enable auto fill, including7independent door parts; source SpaceStation assets remain unchanged. Runtime gold front is independent from source-colored fragments. ImpactWave Shaded Color output attenuates lit base only within the active front so the gold emissive retains its hue; normal and MaterialAttributes preparation both connect it. Existing Gameplay Cue owns presentation; target eligibility/dedup/removal/collision stay in component. Material preparation uses CreateOrGetInputAttribute to allocate valid SetMaterialAttributes slots. Nine material mappings retained.

FEAT274 ScanFX/XHS wave revision: two original TMIIR standalone functions are owned as MF_DisintegrationGun_ScanRing/ScanGrid. ImpactWave calls them with actual world hit origin, same start/duration/radius as removal, scan lead and short surviving-surface trail. Multiscale radial coordinate warp breaks circle regularity; warped world grid with normal-weighted plane selection gives curved filaments; local breakup/flicker/flecks and bright yellow-gold emission provide reference-inspired glow. Only mapped hit-object materials render it, no fullscreen PP/stencil/global scan state. Existing Cue and automatic interior lifecycle unchanged.


FEAT274 plasma gun: production BP_DisintegrationGun uses SKM_DisintegrationGun at0.7 and independent Worker Native/DisintegrationGun layer/montages, fitted to approved camera-relative(13,22,-25), Pitch3/Yaw-7/Roll0, FOV90. Lobby uses separate Characters/MaintenanceWorker/Lobby/Blueprint/BP_MaintenanceWorker_Lobby_DisintegrationGun at0.7; framing remains independent. ADisintegrationGun owns BodyShadowMesh (skeletal, hidden with CastHiddenShadow): Equip copies mesh/materials and leader pose from the visible gun, attaches to character body at EquipSocketName with composed gun-relative/mount transform; Unequip clears pose/mesh and detaches. Shared static proxy remains empty; visible camera mesh does not cast duplicate shadow. Actual shadow A/B and switch restoration verified.


FEAT274 reusable charged shooting: AFirearm owns FirearmChargeComponent settings/presentation. PrimaryFireAbilityClass selects immediate GA_Shoot or GA_ChargedShoot; both use GA_Shoot::ExecuteShot, preserving ConsumeRound -> weapon OnRoundFired -> shared firing feedback. Charge component configures duration, auto-at-full vs release mode, weapon Start/Hold/Release/Idle clips, electrical socket list and separate MuzzlePulseEffect/socket/scale. Equip removal explicitly cancels the primary ability before clearing its handle. Component destroys all charge Niagara and restores weapon animation on interruption. DisintegrationGun defaults0.8s, release-at-full, six original electrical sockets plus MuzzleFlash_Socket pulse. Original weapon clips were finalized externally against the identical renamed weapon skeleton; no retargeting. GameplayCue.Weapon.Charge blueprint lives under DisintegrationGun/GAS/GameplayCues for the first asset consumer; generic native Cue/ability/component reside with shared firearm infrastructure by explicit user request. Electrical and muzzle Niagara retain local-space simulation and now scale sprite size from Engine.Owner.Scale; attachment effects tick last after gun pose. First-person and lobby framing remain0.7.


FEAT274 latest charge presentation (2026-10-11): optional MuzzleChargeParticle supports looping Cascade and takes precedence over Niagara MuzzlePulseEffect. Production DisintegrationGun uses P_DisintegrationGun_ChargeOrb under Effects/ChargeOrb/Systems, derived externally from TMIIR P_Lightning_Bullet_01 with25 owner-local dependencies. All9required modules local-space and looping; warmup.25s. MuzzlePulseScale.52 scales from25% to100% with charge progress, holds size at full while natural simulation and local roll90deg/s continue. First-person1.7 test was too large and superseded. Niagara DesiredAge option remains available for other configurations. Six original electrical socket systems retained. Both charge types destroy on release/cancel/EndPlay. Cascade follows skeletal muzzle with post-update tick/prerequisite. MuzzleChargeOffset is configurable socket-local translation, default zero; production(12,0,0) moves orb about8.4cm forward at gun scale.7. Latest tint shifts blue orb toward gun-current cyan/white while preserving intensity/white highlights. Old Niagara muzzle ring disabled both charging and releasing. Production release now uses ADisintegrationGun ReleaseShotEffect=P_DisintegrationGun_ReleaseShot (source P_Lightning_Shot_01, Effects/ReleaseShot,18owned packages), spawned only in accepted OnRoundFired before hitscan. Attached local-space Cascade, production scale.24, same charge offset, mesh prerequisite/post-update tick, auto destroy plus bounded1s weak timer. Shared MuzzleEffect cleared to prevent double release. Previous NS_DisintegrationGun_ReleaseCurrent retained unused. Existing charge untouched; no beam endpoint/projectile/gameplay changes. Muzzle light blue-white(.3,.9,1).


FEAT274 audio/collision (2026-10-11): TryDisintegrate disables collision on accepted selected meshes immediately, before animated material removal; completion still hides meshes. FirearmChargeComponent optional ChargeSound/ChargeSoundVolume spawns owned attached AudioComponent at muzzle; EndPresentation stops/destroys it on all release/cancel lifecycle paths. .8s clip ends naturally at full charge, no loop. Existing GA_Shoot consumes FireSound only for actual accepted shot. Audio ownership and manifest: arch/14-audio-policy.md.

## FEAT-276 ElectricGun instant electrical field (2026-10-11)
AAttractionBullet remains ABulletBase-compatible for GA_Shoot/BulletClass. BeginPlay hides/disables projectile movement/collision and immediately creates an independent AAttractionField500cm horizontally ahead of actual player view,200cm above traced ground, with obstacle sweep clamp. Launcher expires without visible travel. Names retained for compatibility; there is NO attraction, RootMotionSource or enemy movement manipulation.
Both BP_ElectricGun and worker presentation child temporarily reference BP_AttractionProjectile_Bullet; original ElectricGun bullet preserved, turret unchanged. Field duration2.5s/radius600cm and unobstructed Visibility LOS. Per-field UGE_AttractionDamage applies2Health damage every.5s to linked living enemy ASCs; own handles removed on exit/LOS loss/death/expiry. Immobile living enemies may be damaged.
GameplayCue.Weapon.AttractionProjectile.Field owns presentation; exact GC_Weapon_AttractionProjectile_Field registry tag and scan path verified. Orb original textured core and MIColor1 halo restored. MainEffect uses separately restored N_Attraction_MainLightning: source timing, branching, color, light and pivot(.8,.5); User.MainSize=ground height/.8 square size aligns original arc from ground to orb. Separate enemy branch variant has midpoint pivot(.5,.5), User.LinkSize updated each tick; production BP LinkWidth250 matches default main width (200cm clearance/.8); it retains shorter lifetime and dimmer material. No global exposure changes.
Enemy Hit Cue recognizes field SourceObject and invokes configured directional projectile reaction at strength1 with in-place flag, bypassing blood/decal/audio. Front/Back/Left/Right selected from orb direction in enemy local space, root movement suppressed without changing CharacterMovement mode. Native selection assertions and actual four-target GAS PIE passed. Single-player local Cue lifecycle follows existing dissolve path; no multiplayer claim.


FEAT276 arrival/pulse revision supersedes constant full-strength electrical reaction: field owns transient arrival light and one-shot .26s core scale envelope, activates main at.07s/branches.14-.24s, independently gates branch visibility (width250 retained). Actual damage Cue schedules reactions per target; first strength1/max.55s, subsequent random.22-.38/max.24s with small independent delay. ReactToElectricalHit reuses direction selection, guard and no-root movement via ReactToHit, then applies1.25x/1.8x rate and shortened fade envelope; SampleAnimation respects ActiveDuration. Ordinary reactions retain full-clip duration. No new animations or global config changes. Death/range/LOS/expiry remove target scheduling with existing field state.

FEAT304 first-person equipment rendering: AEquipmentBase::Equip marks its visible StaticMesh/SkeletalMesh as FirstPerson and owner-only when equipped by AFPSCharacterBase. The independent ShadowStaticMesh remains world scale to cast the gun shadow. Unequip clears the visible mesh render mode and owner-only flag. Camera first-person scale is configured on AFPSCharacterBase; this render behavior does not alter weapon collision, muzzle traces, ADS or the gun pose.

2026-10-22 FEAT-297: On an ineligible ExplosionGun impact, DropDart positions the mesh outside the surface and uses the root CollisionSphere as the physics body because the mesh has no simple collision. The root receives a 100 cm/s outward-biased reflected velocity, then gravity and floor collision settle it. It does not attach or explode; ExplosionDelay controls expiry. Eligible targets retain their attachment and fuse behavior. See FEAT-297 archive.
