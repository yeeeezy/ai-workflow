# 装备管理组件

**何时读取：** 修改背包容量、装备切换或初始化流程时。

- `Source/TheManTest/Public/Weapons/_Shared/Components/EquipmentManagerComponent.h`：InitializeEquipment、SwitchEquipment、GetCurrentEquipment、Inventory、AttachTargetMesh。
- 同路径 Private 实现：切换先 Unequip 回收技能，再 Equip、链接新层并挂载。首次装备与后续切换统一调用 QueueEquipPresentation，等待一帧姿势求值后调用装备 PlayEquipEffect 并显示，标记一次 Camera Cut 清理旧画面历史。
- 共用 VFX 由每个 AEquipmentBase 的 UEquipmentEquipEffectComponent 播放，时序/材质状态归属 `Weapons/_Shared/EquipmentBase/Effects/`，Manager 不再持有重复的 0.5 秒定时器。额外滚轮输入在待显示帧或当前装备效果播放期间被忽略。
- 可选装备动画由装备自身 bPlayEquipAnimation / EquipMontage 控制，和共享 VFX 同时播放；EquipmentAnimLayerClass 保持各自配置。
- FPSCharacterBase 只指定 ArmsViewMesh 并初始化背包、预热动画姿势，已移除 PlayInitialEquipEffect 及重复的首装视觉逻辑。
- FinalizeUnequippedEquipment 负责隐藏或挂回 Holster；EndPlay 先 Unequip 回收能力再销毁 Inventory。


2026-09-19: InitializeEquipment and SwitchEquipment apply GetEquipAttachmentTransform() to the actor immediately after socket snap, before deferred reveal. Default identity retains existing equipment behavior; InfiltratorLaserRifle uses explicit placement so its SM/component axes stay canonical. Holster handling unchanged.


## FEAT-149 native role integration (2026-09-23)
Native role loadouts: Worker ExplosionGun,RepairGun,ElectricGun; Executive ExecutiveSlowRifle only. All four now use owner presentation BP copies with matching skeleton/layers/montages/mounts. Original weapon BPs remain for other consumers, including accepted Infiltrator silver. EquipmentManager switching logic unchanged. See archive/FEAT-149-native-worker-executive.md and D:/Blender Projects/WorkerExecutiveNative for evidence.

FEAT-149 user correction: Executive has exactly ONE weapon, the native BP_TheExecutive_ExecutiveSlowRifle. Extra Executive-only InfiltratorLaserRifle/ScopedRifle weapon and animation folders were deleted, not available loadout options. Worker keeps its three guns. Obsolete /Game/Weapons/RepairGun/Meshes/SM_RepairGun deleted and its static-component references cleared in both RepairGun BPs; actual SKM_RepairGun remains. This supersedes earlier six-profile/secondary-Executive framing notes.


## Non-visual backpack items (2026-10-22)

UEquipmentManagerComponent owns ItemInventory (FInventoryItemStack: UInventoryItemDefinition Blueprint class plus quantity) separately from the existing AEquipmentBase Inventory. InitializeItems copies AFPSCharacterBase.InitialInventoryItems at BeginPlay; AddItem, GetItemCount, HasItem and RemoveItem are Blueprint-callable/pure. Item classes are UObjects with no actor, mesh, equip slot, animation, or weapon-wheel entry. BP_AccessCard lives at /Game/Core/_Shared/Items/Blueprint/BP_AccessCard. BP_TheExecutive starts with one card; other roles have empty initial item arrays. APoweredDoor queries this manager for the required class without consuming it. See Documentation/Interaction/PoweredDoor.md and FEAT-302 archive.
