# FEAT-294 Phantom animation graph alignment

## Scope

Start with Phantom only. Keep its existing skeleton-free Humanoid state machine and Phantom child asset overrides; align skill/hit Montage graph contract with shared Enemy template.

## Findings and implementation, 2026-10-14

- Before-write Git checkpoint `477071effc5a1341ff3c57df05cbaef5769dbca3`. Dedicated Perforce pending CL 120386.
- Cold asset inspection: `/Game/Enemy/Humanoid/_Shared/Animations/Logic/ABP_HumanoidEnemy` has no target skeleton; `/Game/Enemy/Humanoid/Phantom/Animations/Logic/ABP_Phantom_Native` is its skeleton-bound child with no own AnimGraph. Parent main graph has state machine, inertialization, left-hand IK, AimIK and `HitReactionSlot`, but lacks `DefaultSlot`. The ordinary Enemy template has both slots.
- Used a temporary editor helper to insert `DefaultSlot` immediately before `HitReactionSlot` without touching the movement state machine or IK. Removed that one-time helper from project source after saving the asset; the reproducible authoring script/log remain external at `D:/UnrealWork/PhantomAnimUnify`. Strengthened the shared montage graph test to require both slots and their connection. Updated team animation template document.
- User closed the Live Coding editor. Development Editor build passed; saved `ABP_HumanoidEnemy` and recompiled/saved `ABP_Phantom_Native`.

## Final status — done 2026-10-14

- Fresh Development Editor / Win64 build succeeded after removing the temporary helper (`D:/UnrealWork/PhantomAnimUnify/build-final.log`). Cold graph test `TheManTest.Enemy.HitMontage.GraphAndDefaults` passed and asserts `DefaultSlot -> HitReactionSlot -> Output` on all three production families.
- Rendered Phantom suite passed 7/7: animation overrides, weapon dummy, patrol loop, shooting smoke, tactical approach/retreat and combat modules (`D:/UnrealWork/PhantomAnimUnify/PhantomTests`). GPU rendering was kept for the Niagara check.
- CL 120386 contains the Humanoid parent ABP, Phantom child ABP and graph-contract test. The edited team document remains in dependent pending CL 120332. No Perforce submission or animation retargeting.
- Phantom already reused a skeleton-free humanoid movement graph; this feature adds the common skill Montage path, not a new movement graph. Its richer state machine is still the shared humanoid template, while `UEnemyAnimInstance` remains the common native movement-data ancestor.
