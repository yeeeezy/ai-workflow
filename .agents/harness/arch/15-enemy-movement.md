# Enemy movement: selection, requests and execution

FEAT-246, 2026-10-06. Implementation and verification status: archive/FEAT-246-enemy-movement.md. This document defines movement ownership. FEAT234 is now authorized and implemented; engagement rules are defined in16-enemy-engagement.md.

## Contract

Behavior trees choose actions and schedule requests. Point queries select destinations and validate reachability without starting movement. Execution owns path following/collision/velocity. Animation consumes actual movement and action state. Different movement families share this responsibility model, not a universal XYZ-only destination or a universal movement solver.

Only one request may drive a creature at once. Ground tasks retain their path request ID; abort only cancels that request, not a newer skill request. Unpossess/death stops the brain and owned movement. Failed queries clear MoveLocation. No NavMesh is a failed ground query, never a silent direct-movement fallback. Failed selection branches wait before retrying.

## 当前通用 Task 与选点入口（2026-10-15）

改 Enemy 的游走、追击、保持距离、跳跃或落点前，先核对实际保存的 BT/`PhaseSkillSets`，再从以下入口选择。战斗中被随机选中的位移动作由 GA 持有动作生命周期并调用合适的查询/motor；普通 Ambient 或持续移动可由预设 Task 直接调度。选点、请求和实际到达分别验证，不因 Task 名称相似而另造节点。

| 入口 | 读入／写出 | 选择与执行边界 |
|---|---|---|
| `BTTask_GroundMovementPreset` | `RandomPoint`；`KeepPlayerDistance` 读 `TargetActor`；`FollowActor` 读 `FollowActor` 或组件配置 | `UEnemyGroundMovementComponent` 选可达点、验证完整 NavMesh 路径、发出/取消自有 `MoveTo` 请求并等待结果；Follow 不自动退化成追玩家。 |
| `BTTask_WallMovementPreset` | `RandomPoint` 选当前墙面的可爬点；`AuthoredPoint` 读 Blackboard `MoveLocation` | `UWallCrawlMovementComponent` 检查整段支撑/间隙、执行爬行并报告结果；不负责跨墙跳跃或回墙。 |
| `BTTask_FindGroundLocation` → `BTTask_MoveGroundLocation` | 前者按 Approach/Retreat/Strafe 写 `MoveLocation`，失败清除；后者读点并发地面 `MoveTo` | 保留既有拆分式策略；普通随机游走/保持距离/跟随优先用地面预设。Move Task 等真实路径结果，选点成功不等于到达。 |
| `BTTask_FindWallLocation` | 通过墙爬组件与 `FEnemyWallPointQuery` 写 `MoveLocation`，失败清除 | 只选同墙受支撑点，不发移动请求；必须另有执行动作消费该点。 |
| `FEnemyGroundPointQuery`／`FEnemyWallPointQuery`／`FEnemyAirPointQuery` | 地面 NavMesh 可达点／墙面受支撑点／有界 3D 点与直线路径检查 | C++ 查询接口，不是 BT Task；空中查询由 CoreMorph 专属 `BTTask_CoreMorphAction` 的 AmbientFlight 随机预设使用，不代表通用 3D 绕障寻路。 |

特殊 Leap、Dash、传送的目标和轨迹由对应 GA/组件验证。若现有入口不能表达新策略，先在功能记录里写清缺失能力、真实使用者、请求令牌/中止归属和失败/重试条件，再新增专属 Task。技能选择链及距离档见 [11-enemy-ai.md](11-enemy-ai.md)。

## Existing consumers

| Consumer | Point selection | Request scheduling | Actual movement |
|---|---|---|---|
| Phantom combat | BTTask_FindGroundLocation, Strafe mode | Humanoid tree parallel attack/movement branch; BTTask_MoveGroundLocation | AIController MoveTo -> PathFollowing -> CharacterMovement |
| Phantom patrol | BTTask_HumanoidRoute FindPatrolPoint -> current/nearest configured APatrolPoint, shared reachable check | PatrolLeg consumes the selected projected MoveLocation; character retains turning, arrival and wait presentation; no autonomous chaining while tree scheduled | NavMesh/path following/CharacterMovement |
| Phantom search | last perceived location, existing search policy | Search task requests path and waits through SearchRush/SearchScan | NavMesh/path following/CharacterMovement |
| Phantom weapon dummy | existing configured patrol points | Existing character scheduling retained for plain-controller noncombat test variant | Same ground navigation; no combat tree |
| Medusa approach/retreat | BTTask_FindMedusaMoveLocation delegates to shared FEnemyGroundPointQuery | Existing action decorators -> selection -> BTTask_MedusaMove; retains distance completion/retry policy | PathFollowing/CharacterMovement; GAS dash stays separate |
| Void Gazer | Saved Ambient uses WallMovementPreset for random crawl; combat wall roam, retreat, pounce, return and transfer select/validate their own destinations inside GA/components | Saved Combat BT selects movement/attack GA through PhaseSkillSets and one UseCombatSkill node; Ambient wall preset owns its request token | WallCrawlMovementComponent for supported crawl; GAS-owned VoidWallLeapComponent for transfer/ground hops |
| CoreMorph Scorpion | FindApproachLocation retains existing distance/fan-of-clearance-probes policy | Existing CoreMorph BT task explicitly queries destination and passes it to TickMovement | Existing eight-leg terrain motor; no NavMesh/global route guarantee |
| CoreMorph Manta | Assigned Spline/FlightPath and existing encounter trajectory | Existing flight/entry GAS lifecycle and CoreMorph tree/encounter gate | Existing FlightComponent and body FlightMotion. Already separated; no rewrite required |

Ground common implementation: Enemy/_Shared/AI/EnemyGroundPointQuery.{h,cpp}, BTTask_FindGroundLocation and BTTask_MoveGroundLocation. The actual consumers are Phantom and Medusa. Medusa's query task is an adapter for its combat settings; it does not duplicate ground geometry/nav logic.

FEAT-284 ground stage (2026-10-14): `UEnemyGroundMovementComponent` is a default subobject of `AEnemyBase` and is active only when `UBTTask_GroundMovementPreset` starts a ground request. The task selects `RandomPoint`, `KeepPlayerDistance`, or `FollowActor`; the component selects and validates a complete NavMesh route, calls `AIController::MoveTo`, tracks its `FAIRequestID` and request token, replans live targets, and cancels only its owned path. PathFollowing and CharacterMovement still move the pawn. Random selection uses a bounded reachable-point search around the position captured at request start. KeepPlayerDistance uses the Blackboard `TargetActor`, a configurable min/max band and the engagement last-known position when that combat target is not observed. FollowActor reads Blackboard `FollowActor` if available, otherwise the component's level-instance `ConfiguredFollowActor`; there is no player fallback. Continuous Keep/Follow requests remain active while holding position, so the BT branch can interrupt them. Humanoid Combat now uses KeepPlayerDistance; Humanoid Ambient tries configured FollowActor, then its authored route/search, then a random leg plus wait. Existing Medusa and Void ground BT adapters still run their original policies. Existing `MoveLocation` remains for those routes and Humanoid authored patrol. The editor migration entry point `EnemyMovementAssetLibrary.UpgradeHumanoidGroundPresets` is idempotent for the two Humanoid trees.

FEAT-284 wall/air stage (2026-10-14): `EEnemyMovementRequestStatus` and `FEnemyMovementRequest` define the shared token/result lifecycle used by ground, Void wall crawl and CoreMorph flight. A new request supersedes the old token; an abort may cancel only its own token. Selection failures report `Failed`, death/unpossess/branch abort report `Interrupted`, and physical arrival reports `Succeeded`. `UBTTask_WallMovementPreset` selects bounded random positions in the current wall plane or an authored blackboard point. `UWallCrawlMovementComponent` validates each candidate with its existing whole-segment capsule clearance and five-point support checks, then executes and reports the crawl. Disabled `bPointSelectionEnabled` rejects random patrol. Saved Void Ambient still uses three wall preset nodes; the former five Combat wall preset nodes were replaced by PhaseSkillSets selected wall-roam GA. Leap and ground return remain GAS actions. The editor migration entry point is `UpgradeVoidWallPresets`.

CoreMorph's `UBTTask_CoreMorphAction` now exposes `AmbientFlightPreset`: `AuthoredRoute` remains the saved default and preserves spline/reference flight and the live-pose encounter handoff. `RandomPoint` starts a three-dimensional flight request through the existing flight GA/component. `FEnemyAirPointQuery` samples bounded XY distance and vertical offsets, rejecting blocked goal spheres and swept straight routes against world/static/dynamic/pawn collision. The flight component rechecks each movement segment and fails if a new obstacle appears; it never silently crosses a blocker. This is a validated direct route, not a global 3D path around obstacles. Arrival ends the GA and the BT task may dwell before retrying. An encounter approach invalidates the patrol token while retaining the current body pose and velocity. Void wall transfers and CoreMorph authored encounter trajectories remain creature-specific. FEAT-284 PIE `WallAirPresets` covers wall arrival/failure/old-token cancellation/death, air blocked routes/BT-started arrival/token handoff/unpossess. Existing `WallLeap`, `GroundPresets` and `SharedTreeRouting` regressions passed.

Ground navigation uses the pawn's agent properties and navigation data, projects candidates and rejects incomplete/unreachable paths. Strafe queries try preferred then opposite side; retreat queries try radial then two diagonal candidates. Bounded ground move tasks fail/yield at their replan deadline; a deadline is not reported as arrival.

FEAT-292 Phantom repair: `FEnemyGroundPresetSettings::bStrafeWhileKeepingDistance` is optional and defaults false for direct component callers. BT ground-preset tasks enable it, so the humanoid combat branch selects reachable lateral goals at a 350 cm step, approaches or retreats diagonally, and continues moving within its distance band. Random ground-point selection requires enough travel beyond MoveTo acceptance where the configured radius permits it. Phantom PIE tactical and shared GroundPresets regressions passed after this change.

Humanoid controller still owns perception, focus and aim presentation, plus compatibility tuning/public pure calculation. It no longer issues recurring combat movement or fakes AlreadyAtGoal for actor MoveTo. Combat tree movement parameters are editable on the selection/execution nodes. The existing three-second skill recovery and extra .5-second wait remain. An unavailable skill waits .5s before retrying, allowing parallel locomotion to continue. Ground movement runs concurrently with the skill task. TakeCover holds its GAS lifetime until its owned path finishes/times out; ordinary selection yields while it is active.

Humanoid attached WeaponMesh never participates in navigation geometry (constructor and BeginPlay enforce this even for serialized Blueprint defaults). Its gameplay collision remains unchanged. A moving weapon previously carved the owner's path, causing repeated replanning and near-zero locomotion on dynamic navigation.

Phantom native AnimBP uses Root Motion from Montages Only: ordinary locomotion is driven by CharacterMovement. Its previous Everything mode overrode navigation velocity with small animation root drift; the change preserves montage root motion and all sequence assets.

Void mechanics remain restricted to static approximately vertical planar walls. Goal contains the wall-relative surface context owned by the component; no ground NavMesh calls. Every segment checks capsule clearance and five-point support along its length, then continues checking during motion. It stops at unsupported edges/obstacles; it cannot route around holes or corners. Legacy AttachToWall/SetWallInput/StopWallCrawl entry points remain; disable automatic patrol and stop the brain before externally driving a demonstration. Animation selection and 180ms blending remain actor presentation. No retargeting changes.

The wall component is currently owned by Void's directory because only Void consumes it. It has no dependency on Void animations or skeleton. Move it to the nearest shared directory only when a second actual enemy uses it.

## Planned capabilities (not implemented)

| Capability | Proposed selection/planning | Proposed execution | When to implement |
|---|---|---|---|
| Additional wall/air policies | Add target-distance/follow modes only when a specific wall or flying enemy needs them; current wall supports random/authored points and CoreMorph flight supports random/authored routes | Existing medium-specific executors and the FEAT-284 token/result lifecycle | Future consumer request |
| Random ground roaming | Implemented in FEAT-284 ground component as a bounded reachable-point query around the request origin | Existing ground path following/CharacterMovement | Humanoid Ambient fallback active; FlyingBug2 remains retired |
| Complex wall navigation | Wall surface graph or authored regions/connections; support holes, corners and changing normals explicitly | Wall component follows surface-aware route segments | A concrete enemy/level requires it |
| Complex free-space flying | Build a finite waypoint search around obstacles if a level requires routes beyond the validated direct air segment | Existing CoreMorph flight component or another creature's motor | A concrete blocked-route case needs detouring |
| Shared spline flight | Route provider emits desired position/tangent/speed independently of visual flight model; authored splines can satisfy a patrol request when configured | Consumer-specific motor; preserve CoreMorph encounter choreography | FEAT-284 should assess extraction against real consumers, not turn authored splines into universal pathfinding |
| Swimming/burrowing/teleport | Medium-specific destination validation and policy | Appropriate movement mode or GAS ability | Concrete enemy introduced |

Never create unused movement subclasses/assets just for symmetry. Ground locomotion, wall crawling and flying keep their own destination data and collision rules. Their shared layer is the patrol request and lifecycle contract, not a single XYZ-only geometric solver. Special displacement (dash/leap/teleport) can remain GAS abilities; only extract trajectory utilities when real skills need them.

## 历史实现与后续修正

以下 FEAT 记录保留实现经过，部分阈值和 BT 路由已被后续修正覆盖。新改动以本页“当前通用 Task 与选点入口”、实际保存的资产和源码为准；不要从旧日期段落复制当前规则。

## FEAT247 Void wall transfer

VoidWallLeapComponent owns a private plan (landing component/point/normal, source/end capsule transforms and cubic Bezier controls). BT FindLeap requests a bounded outward fan query; it never starts movement. Leap activates the saved BGA_VoidGazer_WallLeap default ability. UGA_VoidWallLeap and its ticking AbilityTask exclusively execute Start/Air/Land and release ownership on completion/cancel. No player chase or damage is added; automatic wall patrol is the consumer. Disable WallLeap.bAutomaticLeap to retain crawl-only patrol.

Current tuning: 250–1400cm center separation, 850cm/s nominal flight speed, 100cm control-point lift, 8s cooldown; failed searches retry no faster than 1s. Landing requires five supported probes on the same static nearly vertical component. An enclosing capsule-radius sphere (half-height+1cm) conservatively validates the entire route and covers interpolated capsule rotation; actual root movement is also swept. Revalidate at activation/takeoff and during flight. There is no obstacle routing or guarantee every valid geometric wall is found by the sampled fan.

Crawl pauses while the skill is active and detaches only at takeoff. Successful landing restores the wall basis and disabled/gravity-free CharacterMovement. Interrupted airborne movement stays at the last safe point, becomes upright and falls under gravity; it does not return by teleport or automatically seek a wall. Windup cancellation retains attachment. BT abort, unpossess and death cancel the owning ability; death does not enable falling. WallCrawlMovementComponent::Detach explicitly clears attachment for transfer. Animation/root motion never drives the trajectory.

## Verification policy

Live TestMap fixtures are transient and never saved. Verify real selection plus actual displacement, no-nav/unreachable rejection, interruption, target loss, death/unpossess and preservation of per-creature animation/attack behavior. Compare serialized tree graph and runtime nodes after cold load.

## FlyingBug2 retirement
User explicitly retired FlyingBug2 on 2026-10-06. Remove its 16 owner assets (14 original plus 2 intermediate tree assets), native class/controller and dedicated crawl/slope tests. The old six-leg animation work is no longer an active feature. No other consumer references were found by Asset Registry before deletion. Final deletion verification is recorded in FEAT246.


FEAT248 extends VoidWallLeapComponent with static nearly-horizontal ground destinations (normal Z>0.98), five-point support/capsule enclosure clearance and upright landing. Ground plan source remains attached wall. Ground recovery enables Walking/gravity and native ground idle; no ground navigation/chase/return-to-wall. Ground candidates are chosen near blackboard player, locked before0.8s windup, with warning radius/damage snapshot and single impact at arrival. Revalidation/cancel retains existing no-teleport policy; warning cleans up on completion/cancel/death/endplay. Ground attack planning FindGroundLeap is separate from patrol FindLeap, which only accepts vertical wall hits.


## FEAT249 shared wall selection

Enemy/_Shared/AI/FEnemyWallPointQuery (EnemyWallPointQuery.h/cpp) selects a supported same-wall point without issuing movement. BTTask_FindWallLocation finds UWallCrawlMovementComponent on its pawn (no AVoidGazer cast), clears the configurable Vector ResultKey on failure and writes the selected center on success. Default key MoveLocation; StepOverride=0 uses component PointSelectionStep, otherwise the node override. Component bPointSelectionEnabled gates the task. Void synchronizes its existing bAutomaticWallPatrol/WallPatrolStep into those settings at attachment and tick. Four wall-local cardinal directions and existing support/capsule route validation are preserved. This is local point selection, not global wall navigation.

BT_VoidGazer now uses FindWallLocation -> existing VoidWallAction Move. Legacy serialized FindPoint delegates to the shared query for compatibility. FindLeap/FindGroundLeap/Leap, GAS trajectories, animation and movement executors remain creature-owned; jump extraction explicitly deferred by user. Component source path stays unchanged until a second production consumer exists.

FEAT254 grab victim displacement is creature-owned in VoidTentacleComponent, not enemy locomotion/navigation. CharacterMovement stays disabled while captured; actor root capsule sweeps constrain world-vertical lift and accelerated downward slam, preserving XY. Ground query uses player collision responses and slightly inset capsule to avoid ceiling/floor contact start penetration; actual movement uses full capsule. Missing floor or blocked nonwalkable descent cancels. Escape/cancel restores falling for prior walking mode, never teleports to original capture position. Monster remains in existing ground/wall stance.

FEAT255 supersedes FEAT254 vertical-only immediate lift: stationary4s escape Hook, then Whip follows a subdivided capsule-swept upward curve (450cm up,45cm away from monster,90cm side-arc amplitude). Cubic ease-out launches fast and decelerates; .12s apex turn precedes existing accelerated downward Slam. Floor is checked at activation and throughout motion; capsule remains upright. Controlled camera pitch/yaw/roll changes do not rotate capsule. Whip/Slam own counted look-ignore; all release paths balance only their own movement/look locks. Component ticks PostPhysics before camera manager update. No monster navigation/BT scheduling change.

FEAT255 follow-up tuning: WhipCameraShake1.5deg applies bounded multi-frequency pitch/yaw/roll vibration during launch and1.4x during descent, fades before apex/contact. GetWhipViewRotation provides the same final angular shake to camera and strand anchors; controller aim is not shaken, camera has no extra positional offset.

Final FEAT255 user correction: BP_VoidGazer.WhipCameraShake=0 disables all added airborne vibration.450cm height, authored whip orientation and original landing impact feedback retained. Native shake capability remains optional; actual production BP opts out.


FEAT259: Void now also consumes the existing ground point query/path executor through BTTask_FindVoidGroundLocation and BTTask_VoidGroundMove policy adapters. Selection and execution remain distinct BT nodes; action guards keep wall crawl/leap/grab/melee ownership exclusive. Ground approach/short strafe uses navigation, no direct collision-only fallback. Short wall reposition follows successful skills and transfer; no automatic look interruption inside a selected crawl. See arch/11-enemy-ai.md for scheduling and arch/06-animation.md for actual-velocity animation. Ground-to-wall traversal is implemented by FEAT260 below.


### FEAT260 current Void behavior (2026-10-07)

FEAT260: Void ground policy now uses shared Approach query only (no random side flips),140cm/s slow-aware movement, faces travel, stops in melee range. VoidWallLeapComponent FindWallFromGround probes12directions at350/500/650cm heights after4s ground dwell, retries1s; PlanTo supports a grounded source with vertical lift, validated static wall support and unobstructed capsule/sphere swept path. BeginLeap and Start accept grounded takeoff; same GAS movement ownership, cancellation/falling and AttachToWall landing are reused. Ground-start initial sweep uses upright capsule until above floor, then conservative enclosing sphere. No teleport attachment or wall re-entry from a script; real BT cycles verified.


### FEAT261 wall transfer polish (2026-10-07)
FEAT261 wall-only FlightRotation holds initial10% of flight then SmoothSteps to final wall basis by reach entry. Cubic path, collision sweeps/support checks, destination, GAS ownership, cancellation and ground variants unchanged. Pre-contact Land playback no longer waits for P3; ContactAnimationTime continues into recovery. Existing GroundTakeoff/GroundSlam untouched.


FEAT266 ground descent only: FlightRotation holds initial6%, SmoothSteps to upright at62% flight; native GroundAir samples authored1x looping time; Land transition45ms (native200ms recovery retained). Wall transfers/ground takeoff keep prior timing. Original assets/mesh/tentacles unchanged.


## FEAT268 directional wall animation
Wall-only transfers select StartLeft/Right from departure delta in source wall tangent basis, and LandLeft/Right from source-to-destination position in destination wall tangent basis. Center path retains existing Start/Land. Selection is snapshotted on accepted BeginLeap; cleanup clears active clip pointers. Side push-off detaches at4/30s and continues through first1/6s of flight; side reach uses authored0.3..0.6s before contact, then recovery. Conservative swept route, support validation, damage, cooldown and ground variants unchanged. Wall attack outward recovery is part of presentation and pauses existing wall movement through IsLooking. Corner/edge-over traversal remains unimplemented.

## FEAT268 natural wall return / angular admission (2026-10-08)
Historical FEAT268 behavior, superseded by the 2026-10-15 follow-up below: FindGroundReturnApproach used a Reachable NavMesh point420cm in front of a wall and FindWallReturn walked there through AI MoveTo before turning and planning the wall leap. The current implementation has no return MoveTo.
PlanTo independently requires frontal receiving approach <=45deg, wall departure <=60deg and total root rotation <=120deg. Ground launch additionally needs facing <=20deg, horizontal140..650cm and rise180..500cm. Opposing180deg wall flips now reject, rather than using old omnidirectional slerp. Same-wall jumps remain unsupported. Flight duration has a conservative angular-speed floor; ground holds initial18%, rotates to aligned basis by82%. Ground launch path moves forward/up before receiving reach. Retains conservative sweeps/support revalidation and cancellation/falling. BP CrawlSpeed remains170 (prior user doubling); ground speed unchanged.

Latest FEAT268 airtime correction supersedes the angular-duration floor above: no rotation-based prolongation; ground distance/min1000 speed clamped350..650ms. Facing admission5deg to travel/10deg to receiving normal; approach normal threshold5deg. Ground flight commits launch yaw and <=15deg forward pitch. After validated contact, wall basis aligns over180ms; conservative enclosing-sphere support/collision checks cover all orientations. Wall transfer root admission<=95deg; their duration again follows distance/configured speed.

FEAT268 direct pounce supersedes previous delayed takeoff: ground push detaches at min100ms/clip length, then continues first100ms airborne. Ground cubic P1/P2 use1/3 and2/3 displacement plus20cm up; horizontal velocity constant and rise monotonic for admitted180..500cm targets. Short duration/committed heading/contact recovery unchanged.

FEAT268 descent exposes read-only presentation snapshot of locked impact, swing and blend weight; no additional collision/hit/target-selection logic. BeginLeap ground landing expands mesh bounds to at least4x for native strike and Finish restores captured scale (including cancellation/death).


FEAT268 landing knockback: VoidCombatComponent Impact retains existing radius/height/LOS and one pawn/controller damage. After actual Health decrease with survivor, walking/falling CharacterMovement receives LaunchCharacter with editable SlamOutwardSpeed420/SlamUpwardSpeed520cm/s (XY/Z override). Disabled movement/capture, zero damage/invulnerability, lethal/dead, occluded/outside targets do not launch. No new player movement locks or additional damage.


FEAT234: Void ground chase is an explicit BT branch. Shared path executor optionally tracks the engagement target/last-known location and replaces its owned path at each replan without periodic StopMovement. It yields to ready Far/melee skills; wall return does not preempt distant pursuit. Failed navigation is reported separately as Unreachable, with target retained according to engagement policy. Ground spawns that cannot attach enable gravity/falling. TestMap saved nav bounds expanded around existing landscape and static data rebuilt. Component optional home recovery owns and cancels only its own request; no wall/flight teleport fallback.


## FEAT-280 CoreMorph movement ownership (2026-10-12)
ScorpionMovement now owns readiness, target, enabled/BT/wake configuration and Idle/Approach/Face requests previously on ScorpionCombat. It retains existing terrain probes and anatomical gait. Body owns model resources/final pose and tail solver; tail GA supplies goals per pose substep. No attack stages or damage in the movement component. Flight retains existing path/flight motion and delegates piece resource management/final piece transforms to Body.

## FEAT-283 VoidGazer leap status (2026-10-13)

The WallLeap component validates the planned path, collision support and landing, then exposes displacement and animation helpers to GA_VoidWallLeap. The GA owns accepted action lifetime, Start/Air/Land, phase age, outcome and tagged automatic planning cooldown; AbilityTask_VoidWallLeap calls its per-frame advance. The component keeps only a one-second search retry throttle. Direct planned ground return can still run while automatic patrol planning is cooling down. Warning decal lifetime belongs to the EyeCharge Cue actor during a ground slam.

## 2026-10-15 Void ground leap correction

The initial FEAT295 ground return was attempted at any player distance after the ground dwell, provided a supported wall, navigable approach and clear leap existed. The prior FindWallReturn near-range guard was removed. Ground retreat at <=300cm plans a supported landing away from the target without impact damage; ground pounce at650..1150cm plans a supported landing near the target with impact. Both use BT planning and the existing GA-owned Start/Air/Land executor. StopWallCrawl no longer calls ResetLook while already detached: ResetLook forced Falling at ground takeoff and caused the existing ground-to-wall leap to cancel on its next GA tick. Wall return has first BT priority after its dwell; the two ground skills follow it and precede melee/far attacks.

2026-10-15 follow-up: wall return no longer issues a NavMesh MoveTo. It first tries the wall leap from its current ground position. When the wall is too far, too near, too oblique, or its actual swept wall path fails, it plans one validated, non-damaging ground hop toward a floor point 420cm in front of that wall. The return BT task owns that first GA hop, waits for a successful landing, turns toward the retained wall point, and plans the second GA leap. A successful preparatory hop skips the usual ability cooldown so the second leap can start immediately; failed/cancelled hops and the final wall leap retain normal cooldown. A failed ground landing or second wall plan ends that return attempt. No walking fallback or repeated staging hops.

2026-10-15 frequency update: BP_VoidGazer inherits GroundRoamDuration 16s, GroundRetreatRange 450cm and GroundPounceMin/MaxRange 500..1400cm from UVoidWallLeapComponent. A successful nondamaging ground retreat has no ability cooldown and can chain into a damaging pounce. A successful pounce uses GroundHopCooldownSeconds 3.5s; wall transfers and failures keep the wall ability's 8s cooldown. Real BT automation now covers retreat, pounce impact and far NavMesh pursuit. Pursuit still requires navigable ground between Void and its target.

2026-10-15 close-only chase correction supersedes the far pursuit statement above: Void's ground approach BT task starts only when the player is within 200cm and stops its owned path when the player exceeds that range. MeleeRange is 220cm, so close melee generally wins before ground approach. Retreat, pounce, ranged attacks and wall return retain their own range and priority rules. The live BT test now asserts no distant ground pursuit.

2026-10-15 melee reach follow-up: Void's inherited MeleeRange is now 160cm. The resulting 160..200cm band lets close ground approach attempt a short correction before melee. Melee GA remains stationary while turning/attacking and rechecks target reach at the hit frame. No ground chase is allowed beyond 200cm; without a valid hop, ranged skill or wall return, stationary ground idle is expected.

2026-10-15 Void combat movement is now activated through GAS entries in `PhaseSkillSets`; the old fixed Return/Retreat/Pounce/Ground/Wall BT branches are no longer in the saved combat tree. Close chase uses the shared `FollowActor` ground preset only inside200cm, wall roam uses `StartRandomPreset(120,180)`, and wall transfer and ground hops use the existing leap component. The return GAS entry retains one validated staging hop and no NavMesh walk. Its initial ground age gate is removed, while the post-slam return retry delay still applies. Ambient movement remains separate.

2026-10-15 correction: ground return has no dwell/retry clock or shared leap cooldown selection gate. It remains a random PhaseSkillSets action subject to supported-wall/path checks. The auto Near pool is chosen through450cm, so ground retreat is available throughout its <=450cm range; Mid/Far pounce begins at450cm. Ground chase remains limited to200cm. The old GroundRoamDuration setting and NextReturnAttempt clock are removed from VoidWallLeapComponent.

2026-10-15 user clarification supersedes the 450cm/200cm values above: auto Near is <=500cm; GroundRetreatRange and GroundPounceMinRange are500cm. GroundChaseAttack can be selected throughout Near when melee is ready, walks toward the player if needed, then directly activates its configured melee GA. There is no 200cm chase cap. The old Void ground-move BT task's guard is also500cm. Return-to-wall remains an all-distance random action without a dwell clock.

## Void wall random movement distance (2026-10-15)

Combat GA_VoidWallRoam uses the BP-editable Void WallRoamMinDistance=250cm and WallRoamMaxDistance=450cm, with 24 supported candidate attempts. The three saved Ambient WallMovementPreset tasks use 250..450cm/24 attempts. The task CDO and upgrade authoring defaults match; the saved Ambient values are explicit and were cold-loaded after save. Request validation still requires continuous wall support and a clear route.

2026-10-15 recording correction: grounded Void CharacterMovement orients actor yaw toward travel at 360 degrees/second, so navigated movement uses forward locomotion instead of sustained lateral clips. Wall travel remains separately oriented by the wall motor. Wall-to-ground slam plans require at least 300cm horizontal displacement and use outward-then-inward curve controls; swept path validation still decides whether the planned slam is clear.
