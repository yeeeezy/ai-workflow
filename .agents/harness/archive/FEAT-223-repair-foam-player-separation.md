# FEAT223 Repair foam player separation

Status: done (2026-09-30)

User reports firing while muzzle is inside expanded foam can trap player. User chooses prevention plus gentle push-out if overlapped, superseding temporary pass-through proposal. Source shows near-muzzle immediate hit, expanding blocking sphere and unconditional solid mesh activation without player overlap checks. Existing tests only cover safe landing/stacking.

Checkpoint3652bcbc preserves known FEAT222 changes. User Config/DefaultGame.ini and two rifle BPs excluded/untouched; pycache untouched. Single primary monitor detected. Editor2088 had no dirty content/maps or PIE; closing normally for cold build after Live Coding blocked first attempt. Evidence D:/UnrealWork/RepairSeparation.

Implementation: replace growing sphere Pawn blocking with mesh; preflight growth against player capsule and swept MTD separation, <=4cm/update and <=20cm per foam. No launch/velocity change. If obstructed, finish at previous safe scale; if even previous shape encloses player discard only new foam. Expansion delta capped.05 to avoid hitch jumps. Final size no longer unconditionally resets to maximum. Existing normal lifecycle/slow/range unchanged.

Development Editor build succeeded4.65s after fixing test-local variable shadowing. First actual PIE RepairFoam passed: open case travel20.000cm final scale2.708, wall case travel.875cm final scale1.205, deep initial enclosure removed; every growth step no player overlap. Existing real landing/jump/stack/persistence/slow checks pass. RepairDissipation initially failed only obsolete hardcoded saved20m assertion; current user production range500cm retained and test updated to enabled-range rather than20m. Final rebuild/retest pending.

## Final verification

Development Editor Win64 final build4.41s succeeded, no compiler warnings. Fresh rendered editor PIE RepairFoam and RepairDissipation both Success; former includes every-step capsule overlap checks, <=4cm/update and <=20cm total, wall-limited.875cm displacement, deep initial enclosure removal, then production real projectile floor/stack/standing/jump/persistence/enemy-slow coverage. Production bullet Blueprint opened/compiled, no asset save or asset edits needed. Current user range500cm and scale7 retained.

Actual MaintenanceWorker uses production RepairGun and Enhanced Input primary fire with muzzle confirmed INSIDE settled convex foam (closest-point distance0). Live ammo consumed, player displacement4cm, old foam remains, player successfully sweeps100cm backwards afterwards. Eight checks pass in runtime.json. No launch or velocity modification. External Python fixture was corrected for non-reflected spawn/capsule/muzzle helpers; final uses console summon for transient old foam, reflected socket/local transform resolution and real input. No production helper or test map added.

Final native tests run after cold rebuild; editor21348 placed/verified on newly detected secondary [2572,12,4468,985] (monitor reconnected since initial audit). Restored SpaceStation with PIE stopped, no dirty content/maps. Evidence D:/UnrealWork/RepairSeparation: build-final.log, editor.log, runtime.json, editor-final.json, window.json. Existing AimIK startup warnings unrelated to this change remain.

Three source files plus equipment architecture/harness updated. User rifle BPs/config and all Content untouched by this task; no packaging/version/animation/retargeting changes, final commit or push. Subjective push feel remains for user review. Limits are per newly expanding foam; blocked new foam may finish smaller or be removed when its initial size encloses player. Already settled foam retains ordinary collision.
