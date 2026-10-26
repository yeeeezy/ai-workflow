# FEAT-120 Weapon damage and Station patrol targets

Status: in_progress

User explicitly requests10 damage only for previously nondamaging guns, several patrol test Phantoms permanently placed in SpaceStation clear area about15m from spawn, and no-movement diagnosis/fix. This overrides earlier manual-acceptance freeze and no-permanent-test-fixtures rule for these requested Station placements only. Plan4 targets with routes; preserve positive damage and utility effects. Root owns asset/map writes/build/harness; phantom_weapon_dummy owns113 diagnostics/minimalfix. Checkpointf5cc70f preserves current source/assets/user saved map before changes.

Final build succeeded (StationPatrolTargets/final-build.log). Saved4 targets+8 waypoints+4 cubes;1822 preexisting actor transforms preserved. Repair/Executive damage10, positive damage preserved; ExplosionGun InitialAmmo2, capacity30 unchanged. Manual acceptance released. Actual Station all4 moved; full arrivals not yet verified because external script report used unavailable get_mesh API. Script corrected, not rerun; no production patrol changes justified. New damage/ammo automation compiled but not run. Old package is outdated.
