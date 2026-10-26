# 资产目录规范

## 具体玩家角色功能目录（FEAT-168 / FEAT-169，优先于下方旧示例）

潜伏者、行政官和维修工统一采用 `Characters/<Role>/{Gameplay,Lobby,_Shared}`。FEAT171 用户要求统一类型分类，主角色蓝图位于 `Gameplay/Blueprint`，大厅蓝图位于 `Lobby/Blueprint`；第一人称、身体资源分别位于 `Gameplay/FirstPerson`、`Gameplay/Body`，各自包含 `Animations`、`Meshes`、`Materials`、`Textures`。只在实际存在资源时建立分类。大厅专属资源留在 `Lobby`，游戏和大厅真正共用的身体依赖位于角色 `_Shared/Body`。

具体技能资源集中在 `Gameplay/Abilities/<Ability>`，其 GAS、输入、音效和效果按类型归入技能内部；角色级属性数据位于 `Gameplay/Data`。角色专属武器表现位于 `Gameplay/Weapons/<Weapon>`，独立武器系统的资产继续归 `/Game/Weapons/<Weapon>`。角色、无人机、炮塔及专属武器 Actor 蓝图统一放所属功能的 `Blueprint` 子目录，即使只有一个也不散放在功能根；AnimBP 仍归 `Animations`，GAS 蓝图仍归 `GAS/Abilities` 等语义类型目录，Widget 归 `UI`。多角色通用基础模板继续留在原公共目录。

旧版与 Native 成品动画保留不同子目录，不能因为名字或外观相近直接合并。移动时保留现有资产名、材质参数及行为；共享准入以实际依赖图为依据。下方旧角色示例不再代表这三个角色的当前路径。完整路径映射见 FEAT-168、FEAT-169 归档。

FEAT170：现用炮塔及子弹位于 `TheExecutive/Gameplay/Drone/Turret/Blueprint`。FEAT171 将相同规则应用到另外 14 个散放的 Actor 蓝图，覆盖此前“单个主蓝图直接位于功能根”的例外。旧 `Turret/Variants/Prototype` 已按用户审批删除。

## Character / Weapon naming (FEAT-123)

Asset names use `<Type>_<Owner>[_<Purpose>]`. The owner token matches the owning directory: MaintenanceWorker, Infiltrator, TheExecutive, RepairGun, ElectricGun, ExplosionGun, LaserRifle, ExecutiveSlowRifle, InfiltratorLaserRifle, InfiltratorScopedRifle or TestGun. Subfeatures follow the owner (`TheExecutive_Drone`, `MaintenanceWorker_Lobby`). Gameplay Cue names remain derived from their complete Gameplay Tags. Shared/base infrastructure retains its common semantic owner.

Canonical type prefixes: `BP_` Blueprint, `BGA_` ability Blueprint, `GE_` gameplay effect, `GC_` gameplay cue, `SKM_` skeletal mesh, `SKEL_` skeleton, `PHYS_` physics asset, `SM_` static mesh, `M_` material, `MI_` material instance, `MF_` material function, `T_` texture, `AS_` animation sequence, `AM_` montage, `ABP_` concrete AnimBP, `TABP_` template AnimBP, `BS_` blend space, `CLC_` linear color curve, `CLA_` color curve atlas. Audio retains the established `S_` wave, `SCue_` sound cue, `SA_` attenuation and `SC_` concurrency convention.

Use `FirstPerson` consistently for animation viewpoint names. Texture channels use BaseColor, Normal, Emissive, Metallic, Roughness, Opacity, AO or ORM. Imported source/vendor names are replaced by project purposes. Distinct references/variants are preserved, never merged merely because they share a type. Native classes retain UE A/U prefixes and matching header/source names; reflected class renames require CoreRedirects and consumer resaves. Roster IDs, Gameplay Tags and component identifiers are separate behavior contracts and are not renamed for cosmetic consistency.

The complete FEAT123 old/new asset mapping is recorded in `archive/FEAT-123-asset-rename-map.json`.

## Ownership rules

- 顶层与所有者层按功能所有权组织；所有者内部再按资源类型组织。
- 具体角色、武器、敌人或 Actor 的专属资源必须留在其所有者目录，不得堆入 `CharacterBase`、`WeaponBase` 或 `_Shared`。
- `_Shared` 只接收至少两个已存在使用方明确共同依赖、并且计划保持共同生命周期的资源。
- 即使当前多个具体对象临时使用相同表现资产，只要后续需要独立替换，就为各所有者保留独立资产，不提升为 `_Shared`。
- 不为目录对称创建空目录；资产实际出现时再创建对应分类。

## 统一分类名

FEAT-124: the same `Animations` spelling applies to the matching native Characters/Weapons directories. `Characters/CharacterBase/FPSCharacterBase.h/.cpp` lives directly under its semantic owner; animation instances live together in `CharacterBase/Animations`. Shared firearm/equipment animation instances live in `Weapons/_Shared/Animations`. Public and Private use matching relative directories.

所有者目录按需使用以下名称，不得混用单复数变体：

- `Blueprint`
- `Data`
- `Animations`
- `Meshes`
- `Materials`
- `Textures`
- `Audio`
- `Effects`
- `AI`
- `GAS/Abilities`
- `GAS/Effects`

禁止长期保留 `Animation`、`Mesh`、`Material`、`DataAsset`、`GameplayAbility`、`GameplayEffect` 等同义目录。

Asset type placement is determined from Asset Registry, not the filename: textures belong in `Textures`, linear color curves and atlases in `Curves`, MaterialFunctions in `Materials` (optionally `Functions`/`Layers`). Worker layer textures use `FirstPerson/Textures/Layers`, separate from existing `Textures/Arms` consumers. Infiltrator scan materials and its MPC live together under `Effects/Scan/Materials`. Shared weapon firing ability is `Weapons/_Shared/GAS/Abilities/BGA_Shoot`, not CharacterBase. Niagara systems use the effect's `Systems`; supporting Niagara scripts/effect types may use `Niagara`. Existing semantic feature folders such as Lobby, Drone, Input and equipment effects remain valid.

## 角色

```text
/Game/Characters/
├─ CharacterBase/
│  ├─ Blueprint/
│  ├─ Data/
│  └─ GAS/
│     ├─ Abilities/
│     └─ Effects/
├─ <ConcreteCharacter>/
│  ├─ Blueprint/
│  ├─ Data/
│  ├─ Animations/
│  │  ├─ Body/
│  │  └─ FirstPerson/
│  ├─ Body/
│  │  ├─ Meshes/
│  │  ├─ Materials/
│  │  └─ Textures/
│  ├─ FirstPerson/
│  │  ├─ Meshes/
│  │  ├─ Materials/
│  │  └─ Textures/
│  ├─ Audio/
│  ├─ Effects/
│  └─ GAS/
└─ _Shared/
```

`CharacterBase` 只包含与抽象角色基类直接绑定的 Blueprint、默认数据和 GAS 基础设施，不包含具体身体、手臂、动画、材质或贴图。无骨架、由多个具体玩家角色继承的 Template AnimBP 属于抽象角色架构，可放在 `CharacterBase/Animations/{Body,FirstPerson}/Logic`；具体 Skeleton、Sequence、BlendSpace 和最终子 AnimBP 仍必须留在具体角色目录。

动画资产数量较少时直接放入 `Animations/Body` 或 `Animations/FirstPerson`。只有目录已明显拥挤时，才继续拆分 `Logic`、`Locomotion`、`Actions`。Skeleton 与 Physics Asset 跟随对应 Skeletal Mesh 放在 `Meshes`。

同类型资源出现同名但服务不同消费者时，不得直接覆盖或删除。应在统一分类目录下按消费者继续分层，例如维修工第一人称手臂使用的材质层与贴图分别放在 `FirstPerson/Materials/Layers/Arms` 和 `FirstPerson/Textures/Arms`；保留用于追溯或派生的源材质放在 `FirstPerson/Materials/Source`。

## 武器、敌人和 Actor

2026-10-23 Actor 分类：`/Game/Actors` 第一层按用途使用 `Hazards`、`Interactables`、`Destructibles`、`LevelObjects`、`Tools` 和有实际多方依赖的 `_Shared`。具体对象位于下一层，再按资源类型分类。当前完整映射见 `Documentation/Actors/ActorDirectoryLayout.md`；对应 native 目录使用相同用途分类。`WorldPersistenceTestDoor` 仅为持久化自动化测试，位于 `Core/Persistence/Tests`。

```text
/Game/Weapons/<Weapon>/...
/Game/Enemy/<Family>/<Enemy>/...
/Game/Actors/<ActorFeature>/...
```

同样遵循所有者优先和统一分类名。武器动画可按需分为 `Animations/FirstPerson` 与 `Animations/World`；敌人共享 AI 放在最近共同语义根的 `_Shared/AI`。

单个敌人同一类型目录资产数量很大、且存在明确形态边界时，可在类型目录下按形态继续分层。例如 CoreMorph 的静态网格位于 `Meshes/Manta` 与 `Meshes/Scorpion`。共享 Cover 资源仍按类型放在 `_Shared/Cover/{Blueprint,Meshes,Materials,Textures}`。

## 地图

每张正式地图或测试地图在`/Game/Maps/<MapName>/`拥有自己的目录，地图本体、BuiltData、HLOD以及只服务该地图的Blueprint／Meshes／Materials／Textures／Audio均放在该目录内。`/Game/Maps`根目录不直接放资产。World Partition生成的`__ExternalActors__`和`__ExternalObjects__`跟随地图包路径，由引擎管理。

面向正式关卡的白盒文件、命名和 Perforce 提交细则见项目根目录 `Documentation/LevelDesign/WhiteboxFileStandard.md`。纯临时验证仍复用 TestMap；白盒关卡从创建时使用正式关卡路径。
预建白盒类型目录需要团队同步时，可放 `.p4keep` 让 Perforce 记录目录；真实资源进入后删除对应占位文件。Level01 的五个类型目录采用此方式。

## 新建、导入与迁移

1. 先确定唯一所有者，再选择资源类型目录。
2. 外部素材不得以供应商或素材包目录作为正式路径。
3. 移动和重命名必须通过 Unreal AssetTools，不能直接移动 `.uasset` 文件。
4. 迁移后检查目标存在、旧路径消失、引用更新、定向 Redirector 为零以及磁盘空目录清理。
5. 相关 Blueprint/AnimBP 必须编译保存；涉及硬编码路径时同步更新 C++、测试、脚本和架构文档。


FEAT-092 (2026-09-16): /Game/Maps/SpaceStation/SpaceStation owns263 used packages: map/BuiltData, Blueprint, Meshes, Materials, Textures, Effects and Data/Landscape. No vendor ModularSciFiStation directory or ThirdPersonBP template/animation branch remains.


FEAT-139 native-engine exception: startup movie must be Content/Movies/Opening.mp4 for MoviePlayer discovery/staging. Reproducible source images live outside Content in SourceArt/Opening; encoder in Scripts/Opening. Do not relocate the movie into UI without adapting native discovery.

FEAT174: user-approved19 unused role assets removed (manifest in archive/FEAT-174-approved-final-cleanup.md). Worker obsolete AnimBP/model/material/skeleton/texture candidates are no longer retained; current Native assets and their dependencies remain. Preserved all3 role attribute data assets, Drone physics, current Drone animations and old deployment UI/ring.

## Enemy animation organization (FEAT-279, 2026-10-12)

All Enemy animation owners use `Animations/Logic` (AnimBP), `Locomotion` (idle/movement/turn/jump and BlendSpaces), `Actions` (attacks/reload/death/look/tentacle clips and action montages), and `Reactions` (hit reactions). Create only populated categories. Phantom historical skeleton variants are retained separately in `Animations/Legacy/PreviousSkeleton/{Logic,Locomotion,Actions}` and `Legacy/Mannequin/Reactions`; these are historical finished variants, not new retargeting work. Do not merge incompatible skeleton versions or infer deletion permission from zero references. Phantom OriginalRifle keeps its own ownership root with matching categories. Shared AnimBPs retain their existing shared owners; Skeletons stay in Meshes, Medusa cinematic LevelSequence stays in Sequences. This supersedes Enemy `Native`, flat animation sequence placement and separate BlendSpaces folders. Player animation Native version separation above is unchanged.

Animation content changes require external preview and user approval before migration/application. Directory moves do not approve FEAT278's rejected animation appearance or root-motion treatment.


2026-10-12 category cleanup: all4VoidGazer ability Blueprints now live in `/Game/Enemy/Nightmare/VoidGazer/GAS/Abilities/` (GroundSlam/Melee/WallLeap moved alongside Tentacle). GameplayCues unchanged. Medusa material parameter collection retains asset name `DA_Medusa_OrionGlobalGameplayCollection_cc180041` but now lives in `/Game/Enemy/Nightmare/Medusa/Materials/Native/`; classify by actual MaterialParameterCollection type, not its historical DA_ prefix. All references updated and cold verified; no behavior/parameter change.


2026-10-12 category cleanup: all4VoidGazer ability Blueprints now live in /Game/Enemy/Nightmare/VoidGazer/GAS/Abilities (GroundSlam/Melee/WallLeap alongside Tentacle). GameplayCues unchanged. Medusa MaterialParameterCollection DA_Medusa_OrionGlobalGameplayCollection_cc180041 now lives in /Game/Enemy/Nightmare/Medusa/Materials/Native; historical name retained. References updated/cold verified, no behavior or parameter changes.
