# FEAT-297 Explosion gun target interactions and breakable props

Status: implementation complete, local review pending. Created: 2026-10-17; resumed 2026-10-20 after user approval.

## User intent

- The ExplosionGun dart currently attaches to ordinary surfaces and detonates after its fuse. Targets without an explicit explosion-gun interaction should make the dart fall to the ground without a delayed explosion.
- A reusable glass-window Blueprint should be breakable by the ExplosionGun but should not itself produce an explosion.
- The existing Cube and the new glass window should each be a concrete class with its own break presentation. Retain the existing `AChaosDestructibleCube` / `BP_ChaosDestructibleCube` rather than creating a duplicate Cube class.
- Different targets may use Chaos, mesh replacement, spawned shards, Niagara/audio, or other Blueprint presentation. Do not require every breakable to be a Geometry Collection.
- The user asked to record this as a new feature and explicitly defer execution.

## Read-only baseline

- `AExplosionGunBullet::ProcessHit_Implementation` currently attaches after almost any processed hit and starts `ExplosionDelay` (default 2 s). `Detonate` applies damage, physics impulse, Chaos, cue and conditional bullet time. `TriggerChaos` directly applies strain/impulse to every nearby Geometry Collection and calls the special `ATestInteractionProp::Fracture` path. Ordinary static-mesh walls do not fracture, but they still cause the dart's delayed explosion.
- `AChaosDestructibleCube` already owns a `UGeometryCollectionComponent` root, exposes `FractureAsset` and `Toughness`, and uses a pre-fractured `GC_DestructibleCube` made by an explicit editor-only helper. Its break is caused by the dart's area Chaos operation; the Cube itself does not detonate.
- `ATestInteractionProp` is a separate TestMap fixture that swaps static meshes for pre-fractured Geometry Collections. Preserve its existing test behavior when implementing a general prop contract.

## Proposed architecture for review when resumed

1. Add an opt-in explosion interaction component for eligible props. Keep target eligibility, dart handling and target break presentation separate. Targets without the component should take the non-detonating dart-drop path. Existing Enemy behavior needs an explicit compatibility path.
2. The component delivers a once-only `OnBreak` event with hit/blast context. Each concrete Blueprint owns its presentation. The existing Cube class keeps Chaos; a new reusable glass-window Blueprint owns its intact pane and glass-break presentation. Child Blueprints can vary size, material, shards and effects.
3. Replace the projectile's unconditional nearby-Geometry-Collection fracture loop with explicit eligible-target dispatch. Retain existing damage, enemy hit reactions, bullet time, cues and test-prop behavior only where the selected response calls for them.
4. Keep projectile detonation separate from target breakage. Glass breakage must not create an additional explosion. Specify the direct-hit and nearby-blast responses before implementation, then test Cube, glass, ordinary wall and Enemy paths in actual PIE.

## Acceptance outline

- Unconfigured static wall: dart visibly falls, no fuse or second explosion; wall unchanged.
- Cube: existing class, fracture asset, strength and visual result remain configurable; its intended explosion-gun response still breaks it.
- Glass-window class: responds with its own break presentation, with no explosion caused by the glass.
- Nearby eligible targets respond according to their configured mode; an arbitrary Geometry Collection is not automatically destructible merely because of component type.
- Development Editor build, saved Blueprint compile, cold asset load and focused real projectile/PIE regression checks pass. No destination-project animation retargeting.

## Implementation, 2026-10-20

- `UExplosionInteractionComponent` is the explicit opt-in contract. Direct dart attachment and nearby blast response are independently configurable. It delivers one `OnBreak` event with origin, radius, strain, impulse, angular speed and direct-hit context.
- `AExplosionGunBullet` checks the hit actor and its parent actor, so a pane inside a ChildActorComponent resolves to its window. A configured target or Enemy retains attachment, fuse, damage, blast, cue and bullet-time behavior. An unconfigured surface makes the dart stop, detach into a physical fall and expire after eight seconds, without a fuse or delayed explosion.
- The dart dispatches once to its direct eligible target and then to nearby opted-in targets. Geometry Collection type alone no longer grants breakability. `AChaosDestructibleCube` retains its class, asset, toughness and Chaos fracture, now owning that presentation through the component. `ATestInteractionProp` retains its existing mesh-swap/Geometry Collection behavior through the same contract.
- `BP_ExplosionGlassWindow` is the placeable reusable Blueprint. Its native `AExplosionGlassWindow` owns the interaction component; its vertical Child Actor uses `BP_ExplosionGlassPaneRuntime` / `AExplosionGlassPane`. The pane configures Tempered Glass 1.2's 1x2 data asset, intact/cracked materials, skeletal shard simulation and vendor one-shot Niagara. Glass break only handles its own presentation; it does not launch a second explosion.
- The vendor sample Blueprint's radial-damage graph terminated during PIE on this installation. The project native pane calls the plugin's `ApplyRadialDamageWave` directly. The vendor's empty EndPlay override also caused PIE cleanup ensure; the project pane restores the actor lifecycle call. Vendor files were not modified.
- TemperedGlass 1.2 is installed under the UE 5.7 engine, not inside this project. `TheManTest.uproject` explicitly enables it and `TheManTest.Build.cs` adds the runtime module. Collaborators and build machines need the compatible engine plugin installed. Team handoff: `Documentation/Dependencies/TemperedGlassIntegration.md`.
- Project assets are `/Game/Actors/ExplosionGlassWindow/Blueprint/BP_ExplosionGlassWindow` and `BP_ExplosionGlassPaneRuntime`. No map or external actor was saved.

## Verification and handoff

- Development Editor and Game builds succeeded. Both glass Blueprints cold-loaded, compiled and saved. Rendered TestMap PIE `TheManTest.Player.Weapons.ExplosionInteraction` passed: ordinary wall dart falls; Cube breaks only after fuse; an opted-out nearby Cube stays intact; nearby glass produces simulated physical shard bodies; direct glass still breaks when nearby response is disabled; a real flying dart attaches to and breaks vertical glass. Final retest also passed direct glass with zero nearby-break radius (`interaction-pie-zero-radius-fixed.log`; 26 valid shard bodies, 25 simulating). Before/after screenshots: `D:/UnrealWork/FEAT297/GlassBefore.png` and `GlassAfter.png`. The test map was not saved.
- Rendered Chaos ground regression and PIE damage reaction, radial damage, physics, outcome bullet time and shake regressions passed. `StickyExplosionAndBlood` still has four presentation timing/audio/Niagara assertions failing in rendered PIE; gameplay assertions were unaffected. `ExplosionVisualCapture` lacked its configured weapon. These are not counted as passed.
- Git WIP snapshot before changes: `refs/wip/feat-297-before-implementation` (`9d57da188987ab479dace304f914c427c06db349`). The existing worktree had unrelated FEAT-299 and other changes; they were preserved. Project Perforce ticket expired, so this feature remains local and unsubmitted. Owner visual review and scoped Perforce submission remain.
- No animation retargeting occurred in TheManTest.

## Glass blast VFX follow-up, 2026-10-20

- User clarified that a direct glass hit should play the glass break effect without the ExplosionGun blast visual. The explosion Gameplay Cue also owns audio and camera shake, so skipping the whole Cue would remove unrelated feedback.
- `UExplosionInteractionComponent::bSuppressBlastVFXOnDirectHit` defaults false; `AExplosionGlassWindow` sets it true. The dart sends `Data.Explosion.SuppressBlastVFX` only for a direct hit on a component with this flag. `UGCN_ExplosionGunExplosion` omits its generic Niagara for that context while retaining sound and shake. Nearby glass broken by a Cube blast still sees the Cube's normal blast Niagara.
- Editor and Game Development builds passed. Rendered TestMap PIE `TheManTest.Player.Weapons.ExplosionInteraction` passed (`D:/UnrealWork/FEAT297/glass-no-blast-vfx-pie-final2.log`): Cube still spawns ExplosionGun blast Niagara, a real direct glass hit has no generic blast Niagara near the pane, and glass shards still appear. The ordinary-wall dart fell from Z=5000 to Z=2573.9.
- The first follow-up PIE run exposed a test fixture issue: a temporary `AActor` gained its root after spawning and remained at world origin, while the test assumed it had been placed at Z=2000. The fixture now explicitly sets its transform after registering the root and checks the fall after enough physics frames. Production wall-drop behavior was unchanged.
- Pre-follow-up Git WIP snapshot: `refs/wip/feat-297-before-glass-vfx` (`c2cb235752f826c023bf953975422a5687977ae5`).

## Editable glass dimensions, 2026-10-22

- The vendor 1x2 pane lies in local XY and the placed window pitches it upright. Previously Actor Scale Z changed thickness, while X changed visible height. `AExplosionGlassWindow` now compensates on the child actor during construction and Editor transform edits: Actor Scale Z sets height, Y sets width, X sets thickness. At scale (1,1,1) the pane is about 100 cm tall and 200 cm wide; (1,1,2) is about 200 cm tall.
- The mapped child scale applies to both intact and fractured meshes. Focused rendered `TheManTest.Player.Weapons.ExplosionInteraction` PIE passed at twice height, with 26 valid physical shard bodies all simulating (`D:/UnrealWork/FEAT297/glass-height-rendered-test.log`). A NullRHI run's legacy Niagara visibility assertion failed because rendering was disabled; the rendered run passed.
- Transient Lvl_MainGame Editor details-property checks confirmed immediate Z resizing and combined X/Y/Z scaling. Both glass Blueprints compiled and saved. The transient actor was deleted and the map reloaded clean without saving. Development Editor build passed. Pre-change Git WIP checkpoint: `10427493`. No Perforce submit.


## Ineligible-target dart ricochet and fuse-length cleanup (2026-10-22)

The owner requested a dart hitting a non-explosive target to rebound without embedding, then disappear after the same delay configured for eligible-target detonation. AExplosionGunBullet::DropDart now positions the mesh outside the impact plane using its bounds, reflects the incoming direction with an outward bias, starts physics/gravity with a consistent outward velocity, and sets LifeSpan from ExplosionDelay (zero maps to next-tick expiry). It never attaches or starts a detonation timer. Eligible UExplosionInteractionComponent targets and Enemies retain their attachment/fuse behavior. Development Editor build and rendered TestMap PIE ExplosionInteraction passed. The expanded test covers a synthetic and a real flying ordinary-wall hit, outward position/velocity, falling, 1.5-second cleanup, zero-delay cleanup, no delayed blast, and existing Cube/glass behavior. Log/report: D:/UnrealWork/ExplosionDartBounce/interaction-flight-final.log and InteractionReportFlightFinal. StickyExplosionAndBlood was also run: new expiry assertions passed, while its four previously documented audio/shake/Niagara presentation assertions still fail. Team document: Documentation/Dependencies/TemperedGlassIntegration.md. Pre-write local Git checkpoint af62a352; scoped P4 CL 121880 pending (4 files), not submitted.

## Ineligible dart natural fall follow-up (2026-10-22)

Owner found the 600 cm/s outward rebound too strong. DropDart still places the mesh outside the impact face but now clears linear velocity after physics begins, so gravity drops it naturally. ExplosionDelay expiry and no-fuse/no-blast behavior remain. Updated synthetic and real-flight assertions; Development Editor build and rendered ExplosionInteraction PIE passed. Scoped change remains in P4 CL 121880, not submitted.

## Dart ground-collision correction (2026-10-22)

Owner noticed the supposedly falling dart did not settle on the ground. Read-only editor asset audit found SM_ExplosionGun_Projectile has no simple collision primitives (zero box/sphere/capsule/convex elements); the previous DropDart simulated its visible BulletMesh, while the root CollisionSphere was disabled. The earlier PIE fixture had no floor. DropDart now shrinks the existing root sphere to the mesh's narrow radius, places the actor outside the impact face, enables PhysicsActor blocking and gravity on that root, and keeps BulletMesh attached as visual only. Zero rebound velocity, ExplosionDelay cleanup and non-explosive behavior remain. The updated PIE fixture has a blocking floor: at 0.7 s the dart still falls (Z 4996.7, vertical velocity -65.3 cm/s), then rests at the exact expected contact height Z 4996.10 with vertical speed 0 before its 2.5 s test lifespan expires. Development Editor build and rendered ExplosionInteraction PIE passed. Evidence: D:/UnrealWork/VentToggleFall/GroundRestReport and Saved/Logs/TheManTest.log. No static-mesh asset edit. Local pre-write Git checkpoint 99b34c16; three scoped delivery files remain in P4 CL 121880, not submitted.

## Gentle rebound correction (2026-10-22)

The owner asked for a small bounce instead of zero rebound. On non-explosive impact, DropDart now gives the root collision sphere a 100 cm/s reflected velocity with an outward normal bias, then physics gravity and floor collision take over. This is one-sixth of the prior 600 cm/s ricochet. The dart still never embeds or detonates and expires after ExplosionDelay. Development Editor build and rendered ExplosionInteraction PIE passed: initial outward velocity was 100 cm/s, the dart moved about 6.7 cm outward after 0.7 s, settled on the fixture floor at Z 4996.10, and expired at the configured delay. Report: `D:/UnrealWork/VentToggleFall/LightBounceReport`. The four delivery files remain in pending P4 CL 121880; no submit.

## Team project plugin (2026-10-22)

Tempered Glass 1.2 was copied from the local UE 5.7 Marketplace installation to `Plugins/TemperedGlass` so Perforce teammates can sync it with the project. The project plugin contains its descriptor, 12 source files, 564 content assets, a resource icon and 72 optional Blender source files. Generated Binaries/Intermediate are ignored. Development Game and Development Editor builds passed, including both `TemperedGlass` and `TemperedGlassEditor` project modules. `Documentation/Dependencies/TemperedGlassIntegration.md` now describes project-local use. Dedicated P4 CL 121895 submitted 651 files (650 plugin files and the guide); server head and no-opened state verified. The guide also included the already pending ExplosionGun dart paragraph from CL 121880; the remaining dart source/test files stay pending there. Local Git pre-write checkpoint: 1580de8a.

## Perforce delivery (2026-10-22)

The remaining ExplosionGun dart source and focused tests from pending CL 121880 were submitted as CL 121897 (3 files). The related guide paragraph had already shipped with project plugin CL 121895. Perforce reports no files open on this client.
