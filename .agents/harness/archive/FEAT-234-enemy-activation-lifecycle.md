# Enemy activation architecture — deferred design

Recorded: 2026-09-30
Status: done (2026-10-08)
Original design was deferred; user authorized implementation on2026-10-08, completed below.

User: “这样可以 先记下 这次先不弄”. This deferral applies to the shared activation architecture; it does not cancel the separately discussed Sevarog enemy work. No activation or behavior-tree implementation was changed during this discussion.

## Accepted direction

- Unify the activation / engagement lifecycle through a shared component, while preserving enemy-specific behavior subtrees.
- Configurable signals: sight perception, proximity, encounter trigger volumes and receiving an attack. Conditions remain different for each enemy type.
- Proposed lifecycle: idle/patrol -> optional encounter entry -> combat -> disengage/recovery. Death and invalid targets receive common cleanup.
- Inactive does not mean completely stopped: patrol, idle presentation and ambient Boss flight can continue.
- Ability duration/cooldown is separate from engagement state. Medusa's nightmare ending or Sevarog finishing one attack does not automatically mean disengagement.
- Behavior trees read the common state and select the appropriate subtree. Do not force every enemy into one large tree containing type checks.
- Future debug presentation may expose activation state, activation reason, target and current behavior. Existing Dev Panel range switches remain visualization-only.
- Share blackboard conventions and genuinely reusable tasks incrementally; do not refactor working Phantom/Boss behavior merely for symmetry.

## Existing implementation (unchanged)

- Phantom: AI Controller sight/hit response changes target and combat state; humanoid tree also runs outside combat.
- Medusa: native proximity logic activates its nightmare GAS ability; no behavior tree yet.
- CoreMorph: linked encounter box activates encounter; Boss logic coordinates entry/transition and combat tree. Ambient flight can precede activation.

## Separate Sevarog intent retained

Use UE58Blank /Game/ParagonSevarog model/native animations/FX in TheManTest. Proposed simple tree: ranged Subjugate without extra warning -> original dash to close -> limited melee pressure -> continue melee if still close, otherwise recover ranged spacing; retreat after the melee-chain limit with navigation-aware fallback. Finish an attack before reevaluating distance. Add Sevarog category controls and a position marker debug tool. Only read-only asset/source inspection and design discussion have occurred so far; no Sevarog migration, native class or tree asset has been created. Shared activation unification is not a prerequisite for this work.

## Completion criteria

- [x] Provide a shared activation component with configurable sight, proximity, encounter-volume and hit signals.
- [x] Unify idle/patrol, optional entry, combat, disengage/recovery and death/invalid-target cleanup; preserve inactive patrol and ambient flight.
- [x] Keep ability duration/cooldown separate from engagement state and execute enemy-specific combat subtrees.
- [x] Share necessary blackboard conventions and reusable tasks while preserving Phantom, Medusa and CoreMorph behavior differences.
- [x] Expose activation state/reason, target and current behavior in Dev Panel; range display toggles remain visualization-only.
- [x] Pass Development Editor build, affected Blueprint/tree compilation and per-enemy activation, disengagement, death and travel regressions.

## 2026-10-08 implementation authorized
User explicitly approved component/profile composition independent of parent class, common lifecycle and specific execution, and fixing the reproduced TestMap/Void pursuit issue. Subsequent steering requires discovery/chase/memory/home-leash values configured in the component and consumed according to selected activation rules. Supersedes earlier implementation deferral. FEAT268 accepted scope archived; checkpoint74dc0b31 preserves preceding source/docs, unrelated config/rifles/external actor deletion excluded.

Added shared profile/group conditions and per-owner runtime component, reusable region volume, lifecycle adapters for humanoid/Medusa/Void/CoreMorph, debug state/ranges, Void ground spawn fallback and continuously updated pursuit paths. No retargeting. Initial build failed only TObjectPtr range-loop auto* deduction, fixed explicit TObjectPtr access; next Development Editor build27.03s succeeded. Implementation/asset migration/PIE checks still in progress; do not treat source compilation as behavior acceptance. Evidence D:/UnrealWork/EnemyEngagement.


## Implementation and runtime verification

All four production families now consume the shared component. Saved owner-local profile assets: Phantom sight/hit15m discovery45m chase5s memory; Medusa proximity+LOS/hit20m60m5s; Void proximity+LOS/hit25m/unlimited8s; CoreMorph automatic versus SpaceStation placed actor's region profile. Four BBs have common Lifecycle/Activity/LastKnownPlayerLocation; Void BT rebuilt with explicit chase and preserving existing wall/skill branches. Four BPs opened/compiled/saved; cold profile references and all BB keys verified. Native skill distance remains separate. TestMap navigation volume now encloses49 existing landscape tiles (1774m square incl margin), old34x34.5m coverage replaced; only bounds/Recast external actors changed, prior unrelated deleted actor remains deleted. SpaceStation changed only placed Boss's engagement profile external actor. No production fixture actors saved.

Development Editor builds27.03s,23.20s,22.90s succeeded. Native Engagement.Lifecycle and VoidGazer.GroundCombat both Success twice, latest02:29:31/02:29:40UTC. Lifecycle covers discovery versus chase,0 unlimited, profile isolation, locked-hit rejection/no stale replay, AND groups, region membership, named events, memory, nav failure retaining aggro, unpossess/death. Real saved Phantom activates on sight and exits beyond45m. Real Medusa enters Activating/paused cinematic then Engaged after6.81s, exits beyond60m and clears target. Real SpaceStation region triggers boss entry/flight/reassembly into Scorpion combat with BB player target; manual disengagement clears it and returns Dormant. Existing entry assets unchanged.

Actual generic EnemyActivationTrigger + proximity AND condition verified with live Void: proximity alone rejects, overlap activates, outside box cannot reactivate while still in discovery range; CopyProfileToOverride does not mutate saved profile. Saved TestMap first-person chase119samples:101moving/valid-path samples beyond old player boundary, monster itself crossed old boundary; speed140cm/s, Engaged/Chase/Moving at109m separation. Final camera orientation uses named Rotator args; FirstPersonChase.png visually inspected. Earlier positional Rotator args aimed downward; telemetry remains valid, corrected facing screenshot captured. Python NavigationSystem static path probe produces an engine CDO GetWorld handled ensure once per session; actual native automation and production chase do not require that Python probe. No game-code workaround added.

Initial external fixture errors retained: unavailable asset-editor close API, game-world None assumption, collision-rejected Medusa summon, non-exposed deferred-spawn API, Quat constructor mismatch, BoxComponent location accessor, sample-local variable. Corrected fixtures subsequently passed. Native test logs pass independently of those diagnostic-script failures.

Final compatibility audit found ScorpionRecording intentionally has no player and drives an ATargetPoint; component now has explicit scene-target mode (same rules/gates/lifecycle, no fallback when missing). Recording GameMode and preexisting movement test fixtures migrated to that shared entry. Builds23.24s and6.18s succeeded. Final four-test run (Lifecycle/GroundCombat/Movement.SelectionExecution/Movement.ScorpionExecution) started in release-editor.log; pending completion at this entry. No final commit/push/package/version/retargeting.

## Final regression and handoff

All four final native tests passed in release-editor.log on2026-10-09UTC (local2026-10-08): Lifecycle02:39:47, ScorpionExecution02:40:14, SelectionExecution02:40:53, GroundCombat02:41:01. Scorpion fixture moved3436cm and executed strike phases using explicit target mode. No Error or ensure in the final run; expected pawn-selection fallback and PIE-teardown nav warnings remain. Automation queue empty,4tests performed. Final release-state.json confirms editor TestMap with dirty_maps=[] and dirty_content=[]. Scope complete; FEAT234 moved into completed index. Source/assets remain uncommitted; no push/package/version/retargeting.
