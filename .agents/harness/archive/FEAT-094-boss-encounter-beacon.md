# FEAT-094 Boss encounter Blueprint and sky beacon

Status: done. Closed: 2026-09-16. User authorized Blueprint + sky beacon and explicitly selected one Manta in SpaceStation, transforming to Scorpion on trigger.

- Checkpoint 6766f82 preserves FEAT-093.
- Native cosmetic components: core column, soft halo and ground ring; editable color/height/radius/intensity, default hide after activation. Blueprint owns material configuration. No combat logic in visuals.
- Evidence D:/UnrealWork/BossBeacon.

## Final result

- Blueprint /Game/Enemy/Boss/Encounter/Blueprint/BP_BossEncounterTrigger inherits native BossEncounterTrigger. Dedicated M_BossEncounter_BeaconColumn and M_BossEncounter_BeaconRing in Encounter/Effects/Materials; procedural additive unlit shaders, no imported textures. Native meshes use engine Cylinder and Plane, have no collision/overlap/nav contribution/shadows. Core column plus wider low-opacity halo, two ground rings with gentle shader pulse.
- Details expose BeaconColor, BeaconHeight (default 20000cm), BeaconRadius (55cm), BeaconRingRadius (260cm), BeaconIntensity (12), ShowBeacon and HideBeaconAfterActivation. Construction and BeginPlay refresh dynamic material parameters; successful trigger hides all three meshes by default. Dead/already activated target also hides an obsolete beacon. Existing activation logic unchanged.
- SpaceStation now has one Station_Manta at (-4000,-12000,4337.5932), yaw180, WaitForEncounterTrigger and ReassembleOnActivation true. LandingReference points to Station_BossLanding (-4000,-12000,-162.4068), previously verified open ground. Station_BossEncounterBeacon at (-7300,-12000,-64.0125), box extents (650,900,500), targets that Manta. Original 1807 actors/transforms unchanged; retained old flight route. Map and BuiltData saved by editor. No extra Scorpion is placed.
- Development Editor Win64 build.log Succeeded. Authoring compiled both materials and Blueprint, saved all assets/map. First script attempt used unsupported CustomInput keyword constructor, corrected to blank struct + property setter before any asset save. Authoring editor exited abnormally during shutdown after successful saves; subsequent fresh editor loads and both verification sessions passed with exit 0.
- Actual PIE pie.json ok=true: one dormant Manta, visible noncolliding beacon meshes, correct target reference; entering volume activates once and extinguishes all beacon meshes; duplicate activation rejected; existing GA completes Scorpion transformation and normal AI runs with one ASC. No test actors saved.
- cold-visual.json ok=true: cold map 1810 actors, one Manta and one trigger, persisted landing/flags/materials/200m height, three Encounter assets, zero redirects, Blueprint compile. BeaconFinal00000.png inspected from actual player view: blue-cyan column extends beyond frame into sky, ground rings visible, Manta above. This is the visual evidence; earlier showui captures include editor panels.
- Visible editor/PIE windows launched and monitored on secondary display (x >=2560); final session exit 0. Existing MaintenanceWorker material startup warning remains unrelated. No new beacon material compiler errors.
- Checkpoint 6766f82 preserves previous work; final changes remain local, not committed/pushed.
