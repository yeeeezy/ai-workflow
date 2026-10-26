# FEAT168 保留资产待确认清单

Latest execution (FEAT174): all19 approved assets deleted through Unreal. Older pending counts below are historical; the final manifest is FEAT-174-approved-final-cleanup.md. Cold-load/runtime verification underway.
## 分批审批记录

**最终状态（FEAT174）：审批结束。第七、八批的3项已统一删除并验证，不再待执行；下方审批文字记录当时状态。旧部署 UI/圆环、无人机物理资产、现用无人机三条动画和角色属性数据均保留。完整执行记录见 FEAT-174-approved-final-cleanup.md。**

- 第八批：用户批准将 `AS_Infiltrator_Lobby_RelaxedIdle_02` 和 `BB_TheExecutive_DroneTurret` 加入最后统一删除清单；两项目前仍保留。`PHYS_TheExecutive_Drone` 明确保留，不再列入待删审批。加上第七批手臂材质，共3项已批准待统一删除。本文件下方旧候选表的状态以此最新审批为准。

- 第七批：用户同意删除 `MI_Infiltrator_FirstPersonArms`，但要求后续先详细说明并集中审批，最后统一删除。此材质尚未删除，状态为“已批准，待最后统一执行”。
- 属性数据复核：潜伏者、行政官和维修工的 DA_Attributes 均从清理候选移出并保留。初始化代码确实读取 CharacterData 并应用 InitGEClass，但现用三角色 CDO 两字段均为空，属于待排查配置问题，不应据此删除原始属性配置。详见 2026-09-26-character-init-data-audit.md。

- 第六批：用户批准删除旧 `M_TheExecutive_DroneTurret`，已删除并验证。现用 `M_TheExecutive_DroneTurret_Original` 保留且文件未改动。
- 第五批：用户批准删除 `Drone/Armament/Textures` 下 BaseColor、Normal 两张贴图，已删除并验证。它们来自下载的 All combat mech.blend 内对象 9 的原材质图片，经脚本导出后导入；不是从零绘制。外部原文件保留。
- 第一批：旧部署 UI 和地面圆环共 11 项，用户决定“这些先保留”。范围为 `Deployment/UI` 下 9 项资产、`Deployment/Materials/M_TheExecutive_DeploymentRing`、`Deployment/Textures/T_TheExecutive_DeploymentRingAtlas`。不删除、不清空相关配置，不重复列入待删审批。
- 第二批：用户已批准删除旧炮塔原型，15 项全部删除并验证；实现记录见 FEAT-170-turret-prototype-cleanup.md。外部 Blender 原工程保留。
- 第三批：用户明确无人机 Attack、Death、Landing 三条动画是目前的资源，必须保留。已从待删除候选移出，不再称其为弃用或备用动画；不再次提交删除审批。
- 第四批：用户批准删除三个旧扫描材质，已删除并完成冷加载及当前扫描验证；实现记录见 FEAT-171-scan-cleanup-blueprint-folders.md。高亮母材质、当前网格和烟雾材质保留。

以下表格为目前仍保留的资产。第一批已明确保留；其余等待分批审批。不能仅凭零引用判定弃用。

Current review status:6 rows remain (2 retained UI/ring entries,1 approved-pending arm material,3 undecided). Character initialization data is preserved separately and excluded.

- 旧部署表现：按住 E 的旧 UI、圆环材质及配套 UI 材质、圆环图集。
- 备用角色材质：第一人称手臂材质。
- 备用动画：潜伏者大厅备用站姿。
- 其他：两名角色的属性数据、炮塔黑板、无人机物理资产。

直接候选数量不包括关联依赖；第一批旧 UI / 圆环已明确保留。

| 资产 | 保留原因 |
|---|---|
| `/Game/Characters/TheExecutive/Gameplay/Drone/Deployment/Materials/M_TheExecutive_DeploymentRing` | 旧部署表现，仍有兼容字段和配套资源。 |
| `/Game/Characters/TheExecutive/Gameplay/Drone/Meshes/PHYS_TheExecutive_Drone` | 当前无注册表引用，可能作为备用版本或源资源保留，待你决定。 |
| `/Game/Characters/TheExecutive/Gameplay/Drone/Deployment/UI/WBP_TheExecutive_Deployment` | 旧部署表现，仍有兼容字段和配套资源。 |

## 已确认弃用并删除

`/Game/Characters/Infiltrator/Audio/S_Infiltrator_Scan`：已被 TMIIR07 开始/持续/结束音效替代。确认注册表和磁盘无引用，代码、配置和脚本无加载，两项旧技能音效字段均为空，已删除。现用扫描声音保留。

## 正在使用，不列为删除候选

- `Gameplay/Drone/Animations/AS_TheExecutive_Drone_Attack`、`AS_TheExecutive_Drone_Death`、`AS_TheExecutive_Drone_Landing`：用户明确现用并要求保留。注册表无直接引用不能推翻用户确认。

- `Gameplay/Drone/Deployment/Materials/M_TheExecutive_TurretPlacementHologram`：由 C++ ConstructorHelpers 加载，已验证实际 E 键部署预览。
- 炮塔 Fire 和 Construction Reveal Gameplay Cue：由两个 GameplayCueNotifyPaths 动态发现，冷加载及实际构筑已验证。

旧 UI 的配套材质和圆环图集按第一批审批保留。旧炮塔原型已按第二批审批删除。
