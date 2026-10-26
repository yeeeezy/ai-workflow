# FEAT-236 Boss-only enemy health bars
Status: done
Closed: 2026-09-30
User explicitly requests all non-boss health bars disabled. Base InitializeEnemyHealthBar hides overhead component in game and clears its widget class. Boss override retains separate viewport HUD and health updates. Phantom existing hide policy remains; Medusa nightmare snapshots false visibility and restores false. No health/damage/AI changes. Previous FEAT235 saved in local checkpoint before writes, unrelated user assets excluded.

Checkpoint3ca64440 preserves FEAT235. Development Editor Win64 build29.48s succeeded. Actual transient TestMap PIE verifies Phantom, Phantom_WeaponTest, Medusa, FlyingBug2 overhead hidden; CoreMorph overhead hidden and separate boss HUD visible. No BP or map edits. Evidence D:/UnrealWork/BossOnlyHealthBars/pie.json. Final editor17888 restored SpaceStation without PIE. Changes saved/uncommitted; no packaging/version/push.
