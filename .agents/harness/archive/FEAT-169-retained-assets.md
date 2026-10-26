# 维修工保留资产待确认清单

Latest execution (FEAT174): all19 approved assets deleted through Unreal. Older pending counts below are historical; the final manifest is FEAT-174-approved-final-cleanup.md. Cold-load/runtime verification underway.
## 最新审批

**最终状态（FEAT174）：本清单16项全部获批并统一删除，冷加载和运行验证完成，无待审批或待执行项。当前 Native 资源及其依赖保留，DA_MaintenanceWorker_Attributes 保留，初始化配置未修改。下文审批计数是历史记录；最终路径和结果见 FEAT-174-approved-final-cleanup.md。**

- 用户批准删除4项：`ABP_MaintenanceWorker_Body`、`ABP_MaintenanceWorker_FirstPerson`、`SKM_MaintenanceWorker_LowerBody`、`SKM_MaintenanceWorker_FirstPersonArms_Alternate`。均仍在磁盘，等待最后统一删除；关联骨架、材质、动画不在本批授权范围。当前16项中4项已批准待执行，12项尚待审批。下方历史候选表以此状态为准。
- 用户已批准第二批3项：`BS_RepairGun_FirstPerson_WalkRun`、`AS_MaintenanceWorker_Lobby_RelaxedIdle_02`、`SM_MaintenanceWorker_Lobby_Rifle`。仍保留，等待最后统一删除。维修工累计7项获批、9项未决；连同 FEAT168 的3项，总计10项待统一删除。
- 最后9项提交审批：ReferenceBody 材质、SoftMetal 材质函数、ReferenceBody_Source 材质、FirstPersonArms 骨架、ReferenceLogo 贴图、Aluminum01 色彩/法线两张、Rubber_Blue_01 色彩/法线两张。当前仅为候选，尚未批准；删除执行前需要复核当前实际引用，不能仅凭名称断定弃用。下方旧计数以本段最新状态为准。

属性数据复核：`DA_MaintenanceWorker_Attributes` 已移出删除候选并保留。它保存200/200初始血量；代码初始化需要此类数据，但当前 Worker 蓝图 CharacterData/InitGEClass 均为空，需要单独排查配置。不能把缺少引用当成资产无用，详见 `2026-09-26-character-init-data-audit.md`。本清单剩余16项待审。

以下 16 项当前无注册表直接引用，代码、配置和脚本也未找到按名称加载。它们可能是旧版表现、备用版本或编辑源资源，本次全部保留，其依赖也保留，等你统一决定。

主要包括旧身体/手臂 AnimBP、备用移动 BlendSpace、备用模型与骨架、源材质和材质层、5 张备用贴图、大厅备用站姿与旧展示枪。

当前使用的 Native 身体、手臂、三把武器动画，以及大厅/游戏共用资源已验证，不属于这份待确认清单。没有因为“零引用”自动删除内容资产。

| 资产 | 类型 |
|---|---|

整理中只清除了旧路径重定向器、重命名残留和空目录。226 项实际内容资产全部保留。
