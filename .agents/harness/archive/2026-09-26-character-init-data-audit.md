# Character initial-attribute data audit

Read-only diagnosis2026-09-26. User recalls three role DA_Attributes assets being used by Init. Confirmed native intended path: FPSCharacterBase::PossessedBy requires CharacterData && InitGEClass, reads InitialHealth/InitialMaxHealth, sets Data.Attribute.Health/MaxHealth SetByCaller magnitudes and applies InitGEClass. AttributeSet constructor fallback is100/100.

Fresh Editor CDO read of current Gameplay/Blueprint classes finds both fields null for Infiltrator,TheExecutive,MaintenanceWorker. Role data assets remain present with Health/MaxHealth100/100,300/300,200/200 respectively. They have no registry referencers, consistent with unassigned fields, but this does not establish intended obsolescence. Preserve all three and remove them from cleanup candidates. Do not silently assign fields or change gameplay health during asset review. No determination yet when/why configuration became null; no runtime health measurement or proof of migration causation claimed.

Evidence D:/UnrealWork/CharacterInitAudit/read.py,result.json. Editor opened on detected secondary monitor, read only, no dirty packages, exited. No asset/code changes.

User separately approved deleting MI_Infiltrator_FirstPersonArms but explicitly changed workflow: finish reviewing batches and delete all newly approved candidates together at the end. This material remains present, marked approved-pending; no immediate deletion authorized under updated timing.
