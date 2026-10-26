# 跨系统修改读取顺序

| 任务类型 | 建议先读 |
|---|---|
| 新增 FPS 可玩角色 | `03-character-base.md` → `07-character-classes.md` |
| 新增 GAS 属性 | `04-gas-attributes.md` → `02-core-framework.md`（PlayerState）→ `03-character-base.md`（PossessedBy） |
| 新增装备类型 | `09-equipment-system.md` → `08-equipment-manager.md` |
| 修改输入绑定 | `02-core-framework.md`（Controller）→ `03-character-base.md`（FPSCharacterBase 对应方法） |
| 修改玩家动画参数 | `06-animation.md` → `03-character-base.md`（ArmsMesh 来源） |
| 修改人形怪动画参数 | `06-animation.md` → `07-character-classes.md`（HumanoidEnemy 驱动状态来源） |
| 修改角色初始属性 | `05-character-data-asset.md` → `04-gas-attributes.md` → `03-character-base.md`（InitGEClass 应用） |
| 新增枪械 GAS 技能 | `10-gas-abilities.md` → `09-equipment-system.md`（Firearm 技能配置字段）→ `03-character-base.md`（PossessedBy 补授逻辑） |
| 调试开火 / GAS 事件 | `02-core-framework.md`（Controller 输入绑定）→ `03-character-base.md`（PrimaryFire 发事件）→ `10-gas-abilities.md`（GA_Shoot ActivateAbility） |
| 修改人形怪 AI | `11-enemy-ai.md`（技能集/现行 BT）→ `15-enemy-movement.md`（选点/执行）；若涉及本体类，再读 `07-character-classes.md`；若涉及动画驱动，再读 `06-animation.md`。 |
| 新增或修改 Enemy 战斗动作、技能集、行为树及选点/移动 | `11-enemy-ai.md`（现行技能池/共享 Task）→ `15-enemy-movement.md`（选点与执行入口）；目标/感知/生命周期还读 `16-enemy-engagement.md`，GA 实现还读 `10-gas-abilities.md`。先核对目标 BP 的 `PhaseSkillSets` 和已保存 BT，再设计分支。 |
