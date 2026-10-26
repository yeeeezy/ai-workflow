# FEAT-095 Manta pre-encounter patrol

Status: done. Closed: 2026-09-16. User authorized ambient patrol before beacon, then transform and fight.

- CoreMorph opt-in PatrolBeforeEncounter starts existing Flight GA after GAS/form initialization, only while encounter is gated and a route is assigned. Combat tree remains gated; no attack targeting or automatic reassembly before trigger. Closed routes loop, open routes hold at endpoint. Missing/invalid route holds and emits developer warning once.
- Existing reassembly GA captures live flight then cancels Flight. No common Enemy/humanoid or BT asset restructuring. Generic noncombat/entry/combat dynamic-subtree architecture discussed as future design, not implemented this batch.
- Build/map/PIE passed. Evidence D:/UnrealWork/MantaPatrol.

- Checkpoint 9ec4a40 preserves FEAT-094. Development Editor build.log Succeeded. SpaceStation Station_Manta now sets PatrolBeforeEncounter=true, FlightRoute=Station_MantaRoute, RouteSpeed=3000cm/s. Existing 8-point closed spline length 110005.73cm retained; Boss editor placement aligned to spline first point to avoid startup reposition surprise. LandingReference and beacon unchanged. Author editor exited 0.
- Recommended future design: shared state/blackboard contract; family-level main tree with NonCombat, EntryTransition and Combat branches. Dynamic combat subtree only where runtime strategy replacement is needed. Keep root switching conditions on main tree; cancel owned GA/movement on branch interruption. PhaseSkillSets already handles ability-set changes, so changing skills alone does not require swapping trees. No new shared main tree/dynamic subtree in this batch.


## Final verification

- Fresh editor PIE pie.json ok=true: saved flags/route/speed loaded; full closed route wraps key7.9859->0.00215 after37.03 game seconds, stays Manta/unactivated with inactive combat Brain and null Scorpion target. Test-only time dilation4 accelerates loop observation; restored1 before triggering, not saved.
- Runtime invalid Layout rejects entry while patrol keeps moving >100cm and beacon remains visible/unconsumed; restored Layout retries while player remains inside. Activation succeeds once; AI waits during live-flight GA reassembly. Scorpion completes within1000cm horizontal of landing reference, combat Brain running with one ASC; leaving preserves encounter.
- First test script tried reading protected MissileCombat Target; corrected to public Scorpion target/inactive Brain assertions. Successful transition session exited abnormally after normal engine shutdown logs; separate fresh patrol-exit.json verifies cold startup/movement and ending PIE during active flight, with clean editor exit0. No product-code errors observed. Tests do not save runtime changes or fixtures.
- Default PatrolBeforeEncounter=false preserves existing other placements. Closed routes repeat, open routes hold at end; invalid/missing route logs once and stays idle. No GA, BT, EnemyBase or humanoid changes.
- Final source/map changes local, not committed/pushed. Checkpoint9ec4a40 preserves previous feature. Evidence D:/UnrealWork/MantaPatrol.
