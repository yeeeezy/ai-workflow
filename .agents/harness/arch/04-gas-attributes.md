# GAS 属性集

**何时读取：** 新增或修改角色属性（血量、耐力、护甲等）时。

| 文件 | 关键内容 |
|---|---|
| `Source/TheManTest/Public/Characters/CharacterBase/TheManAttributeSetBase.h` | `Health` / `MaxHealth` 声明，`ATTRIBUTE_ACCESSORS` 宏用法示例 |
| `Source/TheManTest/Private/Characters/CharacterBase/TheManAttributeSetBase.cpp` | 属性默认值 |
| `Source/TheManTest/Public/Enemy/EnemyAttributeSetBase.h` | 继承 `UTheManAttributeSetBase`，怪物专属属性扩展点 |

> 新增属性后，还需要创建对应的 `GE_` 蓝图来初始化数值，并在角色蓝图的 `InitGEClass` 中引用。

Combat HUD 由 `ATheManPlayerController` 绑定 PlayerState ASC 的 Health/MaxHealth 属性变化委托；界面当前只绘制取整后的 Health，不绘制 MaxHealth。

环境火焰的 `UGE_FlameDamage` 是原生 Instant Gameplay Effect，通过 `Data.Damage` SetByCaller 对 `Health` 做负向 Additive 修改；`AFlameHazard` 从目标 Pawn 获取 ASC 并应用该 Effect。`DamagePerTick` 默认 50、实例可调；喷火期间每 0.5 秒伤害一次，默认喷 2 秒、停 2 秒，停喷时伤害计时器停止。`FireDamageEffectClass` 可换成日后配置 Gameplay Cue 的 GE 子类；当前未绑定 Cue。

敌人头顶血条由 `AEnemyBase` 统一持有 `UWidgetComponent`，使用原生 `UEnemyHealthBarWidgetBase` 绘制黑底红条。`AEnemyBase::BeginPlay` 在 InitGE 后绑定自身 ASC 的 Health/MaxHealth 变化委托并立即刷新，因此 Phantom、Nightmare 及后续所有 `AEnemyBase` 子类自动继承，无需在各敌人 Blueprint 重复配置。


## FEAT-097 replaceable enemy health UI
EnemyBase retains the single Health/MaxHealth ASC subscription. InitializeEnemyHealthBar/GetActiveHealthBarWidget/RefreshEnemyHealthBar are virtual presentation hooks. Ordinary enemies choose EnemyHealthBar component Widget Class (derive from UEnemyHealthBarWidgetBase); custom Designer trees are honored and OnHealthStateChanged plus read-only displayed values/GetHealthFraction supply data. The original native red bar remains the fallback.
BossEnemyBase instead owns BossHealthBarWidgetClass and a local-player viewport widget, suppresses overhead UI, retries creation until a local controller exists, hides until CanRunEncounter, and removes on death/EndPlay. UBossHealthBarWidgetBase draws the minimal cyan design, exposes fill color/width/top offset, name/phase and OnBossStateChanged; custom Designer trees can replace it. CoreMorph updates MANTA/SCORPION on SetForm and preserves actual combat phase and health. Configured asset: /Game/Enemy/Boss/CoreMorph/UI/Blueprint/WBP_CoreMorphBossHealth. No image texture or second ASC required.


## FEAT-135 Debug invulnerability

Native State.Debug.GodMode is added/removed by the character GodModeComponent on the existing PlayerState ASC. UTheManAttributeSetBase checks that tag AND a player-character avatar, rejecting negative health gameplay-effect executions before settlement. PreAttributeChange/PreAttributeBaseChange also prevent health reductions through direct/duration paths; healing remains allowed. Enemies do not inherit player invulnerability. Actor CanBeDamaged is captured/disabled/restored for ordinary engine damage; Character OnDeath is guarded while active. No per-frame health refill. Unpossess clears the tag before the shared ASC can serve a new character.

## FEAT252 independent player health protection
Existing health reduction guards now accept either State.Debug.GodMode or the current player avatar controller's IsDebugInvincibleEnabled flag. Enemy avatars never qualify. PreGameplayEffectExecute rejects negative health execution; PreAttributeChange and PreAttributeBaseChange block health reduction while allowing healing. No per-frame refill, forced max health, movement/collision/input changes or new tags. Controller query avoids leaving protection on a shared PlayerState ASC after unpossess. It does not override explicit scripted death/round-end/kill-volume flow. Shipping flag getter alwaysfalse.
