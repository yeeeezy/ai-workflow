# Enemy engagement (FEAT234)

Current source of truth for acquisition, target lifetime and disengagement. Supersedes historical activation descriptions in arch11 and FEAT234's original deferred design. Movement ownership remains documented in arch15.

## Configuration

Every AEnemyBase owns EnemyEngagementComponent. The component selects an immutable UEnemyEngagementProfile or its LocalSettings when bOverrideSettings is enabled. CopyProfileToOverride copies a profile into the instance before independent editing. Targets, observations, timers, region membership and navigation results are per-component; never stored in the shared asset.

Settings own DetectionRadius (cm), ChaseRadius (enemy-to-target cm, 0 unlimited), MemorySeconds, optional HomeLeashRadius (target-to-home cm, 0 disabled), recovery/return policy and entry timeout. Skill distances remain on skill/combat settings. Medusa's serialized ActivationRadius is retained only as the Dark Veil Cast Range; the acquisition service no longer reads it.

ActivationGroups are ORed. Conditions in each non-empty group are ANDed. Sight and Proximity normally read DetectionRadius; an optional RadiusOverride handles a genuinely different distance inside a compound group. Sight also reads HalfAngle, and both can require LOS. Hit/Event are short-lived signals with a valid living controlled player target by default. Authored no-player scenes may opt into bUseExplicitTarget and supply a specific actor/target point; it still passes the same activation groups, gates, lifecycle and range policy, and a missing explicit target never falls back to a player. ScorpionRecording and its existing movement regression use this explicit target mode. Named Event and optional source filters are supported. Region membership persists only while the source and overlapping target remain valid. Automatic is explicit. RequiredOwnerTags and bActivationUnlocked gate every activation branch, including hit. Rejected locked signals are not retained for a later unlock. Chase/home limits also constrain acquisition, preventing activate/disengage oscillation outside the configured leash.

`HoldSeconds` counts continuous satisfaction of a condition; disabling engagement or failing an outer activation gate clears the accumulated start time. Hit/Event activation may identify a target through cover, but `bTargetObserved` is initialized from the tracking LOS policy immediately; combat skills that require an observed target do not treat a hidden attacker as visible before the next component tick. `bRetainTargetUntilInvalid` and profiles that do not require tracking LOS retain their existing behavior.

Generic AEnemyActivationTrigger emits membership for any linked AEnemyBase, without class-specific activation code. BossEncounterTrigger keeps its existing beacon/one-shot presentation and forwards a region signal through ActivateEncounter. Existing native bosses without a profile retain a compatibility default from bWaitForEncounterTrigger; explicit profiles/overrides always take precedence. Saved BP_CoreMorphBoss uses the automatic profile, while SpaceStation's placed waiting boss overrides it with the region profile.

Current saved defaults:

| Consumer | Acquisition | Detection | Chase | Memory |
|---|---|---:|---:|---:|
| Phantom | sight OR hit | 1500 cm | 4500 cm | 5 s |
| Medusa | proximity with LOS OR hit | 2000 cm | 6000 cm | 5 s |
| VoidGazer | proximity with LOS OR hit | 2500 cm | unlimited | 8 s |
| CoreMorph default | automatic | unused | unlimited | retains valid target |
| SpaceStation CoreMorph | encounter region | unused | unlimited | retains valid target |

Profiles live in each actual owner's Data directory; any enemy component can reference any of them. No inheritance checks select detection rules. Phantom stationary/weapon-test flags still disable engagement when enabled. FEAT278 (2026-10-12): the saved BP_Phantom_WeaponTest now disables both test flags and opts into normal engagement via BP_HumanoidAIController + DA_Phantom_Engagement. Initial state Patrol; sight or hit enters Aim with target tracking and BT movement. Its offensive ability lists remain empty. This supersedes old non-retaliating production preset assumptions.

## Runtime ownership

Lifecycle: Dormant -> Activating -> Engaged -> Recovering -> Dormant; Dead is terminal. Activity is a separate presentation/debug view (Idle, Patrol, Entry, Chase, Search, Attack, Return). Dormant does not stop ambient patrol/wall crawling/encounter flight. Native enemy entry hooks integrate Medusa's existing cinematic and CoreMorph entry/reassembly, with bounded failure timeout. Entry/skill durations do not redefine engagement lifetime.

The component owns target selection, observation/last-known location, memory, chase/home exits and cleanup. BT/Controller adapters consume its state; they no longer independently discover a world player or clear aggro by their own range. BB conventions: TargetActor, LastKnownPlayerLocation, Lifecycle and Activity. Existing enemy combat subtrees remain separate. Base recovery cancels GAS, clears skill target and stops old movement/focus; creature hooks clean their own combat/intro state. Unpossess, invalid/dead/replaced player and enemy death release target state.

Memory preserves the last observed location. Shared/Medusa/Void ground queries can approach that position during Search; ordinary BT combat skills require an observed target. Humanoid retains its existing search/scan presentation during recovery. Void ground return uses a direct wall leap or one validated preparatory ground hop, with bounded retries; unsupported return does not teleport and times out. Specific movement families retain their executors.

## Pursuit and diagnostics

NavigationStatus is independent of lifecycle. Unreachable records a path failure and retries through existing BT scheduling; it never fabricates a walk path or immediately clears the engagement target. Void has an explicit ground pursuit branch and updates its owned path without the former periodic StopMovement/task-restart cadence. It yields to a newly available Far skill or melee. Return-to-wall does not preempt a distant ground pursuit. Void spawns that fail to attach to a wall now enable gravity/falling and naturally enter walking on landing.

TestMap's prior 34 x 34.5 m navigation volume covered only a tiny part of the existing terrain. The volume now encloses its 49 loaded landscape tiles plus 500 cm margin; static navigation rebuilt and saved. The existing unrelated deleted external actor is not restored.

Dev Panel range/position controls include VoidGazer. Actual component rules supply detection/LOS geometry, chase radius and target/lifecycle/activity/reason/navigation text. Visualization never changes activation. Required verification includes cold asset references, native Engagement.Lifecycle and VoidGazer.GroundCombat, real first-person traversal past the former nav boundary, per-family entry/cleanup, and profile override isolation. Evidence and current pass/fail status are in archive/FEAT-234-enemy-activation-lifecycle.md.

FEAT278 stationary preview supersedes earlier test-soldier sight/BT configuration: BP_Phantom_WeaponTest now references native HumanoidAIController (null BehaviorTree), startsPatrol, uses local Hit-only activation override, RetainTargetUntilInvalid=true, ChaseRadius/HomeLeashRadius0 and ReturnHome=false. Preset has no patrol points. Enables original hit-to-Aim lifecycle and focus without automatic BT navigation; shared Phantom profile/controller and formal BP_Phantom unchanged. Both legacy test bypass flags remain false.


## FEAT-282 lifecycle tree execution
The Engagement component remains the sole state/target authority. AEnemyAIController runs one shared main BT with lifecycle decorators and real family-specific dynamic subtrees. State changes abort the old branch and enter the new branch; creature entry hooks and base recovery cleanup are not duplicated in the tree. CoreMorph's dormant flight is owned by its Ambient subtree; activation hands the live flight pose to entry/reassembly before cancelling flight. Dead and unpossessed enemies stop the brain. See arch/11-enemy-ai.md and archive/FEAT-282-enemy-lifecycle-tree.md.
