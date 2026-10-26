# FEAT-302 Scan-gated world interactables

Status: done, 2026-10-20.

## Request and behavior

- E is reserved for world interaction on every character. F activates the current character ability: Infiltrator scan and Executive drone deployment. Q Blink/remote behavior is unchanged.
- `AInteractableBase` owns a configurable `InteractionRange` Box, `bRequiresScan`, persistent per-actor `bDiscovered`, `InteractionLabel`, and a BlueprintNativeEvent `Interact` hook. The base interaction hook is empty.
- The Infiltrator scan marks interactable actors discovered when its wave reaches them. Scan highlight still clears on scan retract; discovery remains until the actor is reset or destroyed.
- `ATheManPlayerController` tracks Box overlaps for its possessed pawn, chooses the nearest eligible actor, checks range again on E, and sends `[E] <label>` to the existing Combat HUD. It clears candidates/prompt on unpossess and rechecks overlaps after possession.
- `BP_ToggleSwitch` uses the Blender modeled switch, is immediately interactable, and has no switch action yet.
- `BP_Level01_RobotVent_Cover` uses `SM_Level01_RobotVentCover`, requires scan, and opens once by rotating its cover about the top-edge hinge.

## Assets and source

- Native: `Source/TheManTest/Public/Actors/InteractableBase.h`, `RobotVentCover.h`, controller, HUD, scan presentation, input/GAS character bindings.
- `/Game/Core/Input/Actions/IA_CharacterAbility` mapped to F in `/Game/Core/Input/Contexts/IMC_Default`; existing `/Game/Core/Input/Actions/IA_Interact` remains E.
- `/Game/Actors/ToggleSwitch/Blueprint/BP_ToggleSwitch`, `/Game/Actors/ToggleSwitch/Meshes/SM_UpDownSwitch`, and nine `M_UpDownSwitch_*` materials under `/Game/Actors/ToggleSwitch/Materials`.
- `/Game/Actors/Interactables/RobotVentCover/Blueprint/BP_Level01_RobotVent_Cover` references the existing Level01 vent mesh. The Blueprint moved here on 2026-10-21 from `/Game/Maps/Level01/Blueprint/` during the actor folder cleanup.
- Blender source remains `D:/Blender Projects/UpDownSwitch/UpDownSwitch.blend`; FBX export helper/source intermediate is external under `D:/UnrealWork/FEAT302/`.
- At initial FEAT-302 completion, no persistent level actor placement or level save was made; the MainGame placement is recorded in the follow-up below.

## Verification

- `Build.bat TheManTest Win64 Development ... -NoHotReload` passed after final hinge change.
- Live Coding passed in the open editor. Standard Development Editor link was blocked by two open editor processes holding the original DLL. `Build.bat TheManTestEditor Win64 Development ... -ModuleWithSuffix=TheManTest,302 -NoHotReloadFromIDE` passed and updated `Binaries/Win64/UnrealEditor.modules` to the new DLL without closing either editor.
- A separate cold `UnrealEditor-Cmd.exe -run=pythonscript` run exited 0. It read both GA native triggers as `Input.Character.Ability`, E/F mappings as `IA_Interact E` / `IA_CharacterAbility F`, switch `requires_scan=False`, vent `requires_scan=True`, both saved mesh references, and the final vent hinge at `(65,0,130)` with roll `-95`.
- Final transient PIE check after the hinge correction: F made the vent CoverMesh custom-depth highlight true; explicit scan retract returned it to false; E still opened the discovered vent, and CoverHinge reached roll -95. No persistent map actor was saved.
- Editor validation loaded all five key assets successfully. Both Blueprints compiled and saved; switch mesh readback showed 9 material slots at the final Materials paths and a 34 x 28.2 x 48 cm bounds.
- Transient Lvl_MainGame PIE: the switch Box overlapped the pawn; a fresh vent did not rotate on E before discovery; manual scan wave reached it and E opened it after highlight lifetime. After refreshing the live editor's stale pre-patch native ability CDO in memory, actual F input activated Infiltrator scan (radius > 0), and E opened a fresh scanned vent. The stale CDO was confined to the already-running editor; cold commandlet confirmed the saved native constructor has the correct F tag.
- WIP checkpoint before edits: `refs/wip/before-scan-gated-interactables` (`6569a27ddec9efd05d97867411e7e1e4b72cbebd`). Existing unrelated worktree state was preserved. No Perforce submit was attempted.

## MainGame placement and smooth opening follow-up (2026-10-21)

At the owner's request, `ARobotVentCover` now eases its hinge from its current rotation to the configured -95 degree angle over 0.65 seconds using a cubic smoothstep curve. The one-time interaction also disables its overlap range and removes the E prompt. `BP_Level01_RobotVent_Cover` was compiled and placed in `Lvl_MainGame` under the existing `RobotVent_Cover` label and `Level01/UpperWarehouse/RobotVent/HangingCover` folder, replacing `StaticMeshActor_236` at the same location, rotation, scale and CoverMesh world transform. The Level01 cover mesh asset stayed in place. The opened cover left the Spider Tank vent route unobstructed in a live PIE capsule sweep.

## Scannable interaction separation (2026-10-21)

The owner clarified that ordinary interactables must not receive scan highlight, and every scannable object must be interactable. `AInteractableBase` now owns only E interaction and Box range. Optional `UScannableInteractableComponent : UHighlightComponent` gates `CanInteract`, stores one-time discovery and supplies the temporary object highlight. On first wave contact it unlocks E and calls the actor's BlueprintNativeEvent `OnScanCompleted(AFPSInfiltrator*)`; each child Blueprint may implement different discovery logic. The controller's existing eligibility check then exposes the E prompt. Discovery persists after scan retract while the highlight ends. The component rejects non-`AInteractableBase` owners at runtime.

`ARobotVentCover` now owns the scan component. `BP_InteractableBase`, `BP_ScanCube`, and `BP_ToggleSwitch` have no scan component, remain directly interactable and do not receive scan object highlights. The enemy gold overlay stays on its separate scan path. Four affected Blueprints were compiled and saved in the editor; the vent Blueprint was also opened and compiled after the final native build. Final Development Editor build `D:/UnrealWork/ScanGateRefactorCheckpoint/EditorBuild5.log` and Development Game build `GameBuild.log` succeeded. The actual TestMap PIE `TheManTest.Player.Scan.GoldHighlightLifecycle` passed, including pre-scan E lock/no prompt, vent discovery/highlight, ordinary object no highlight, post-scan E prompt and action, highlight cleanup and persistent discovery. A first test placement was outside the Box range, so its E assertion failed; the fixture was moved into range and the final run passed. WIP Git ref `refs/wip/before-scannable-interactable-separation` preserves tracked pre-write state; external checkpoint holds the untracked vent source/Blueprint. No Perforce submit or Git push.

## MainGame missing outline fix (2026-10-21)

The owner's `204723` system screenshot shows the placed rotating vent cover during the Infiltrator scan with no object outline. The prior TestMap PIE test only verified the scan component and CustomDepth; it missed MainGame's level postprocess. `Level01_PostProcess_Exposure` in `/Game/Maps/Lvl_MainGame` had no weighted blendables. Added the existing `/Game/Characters/Infiltrator/Gameplay/Abilities/Scan/Materials/M_Infiltrator_Highlight` at weight 1.0, matching the material already used by TestMap and SpaceStation. Saved and reloaded the map in a cold editor commandlet; the active editor readback confirmed the reference. In MainGame PIE, `ScanComponent.start_highlight` changed `CoverMesh.render_custom_depth` from false to true and stencil from 0 to 1. A rendered screenshot from that PIE did not frame the cover, so final on-screen outline appearance remains to be checked in the normal Infiltrator camera. No code, material, or Blueprint changes in this follow-up. Pre-write checkpoint `refs/wip/before-maingame-scan-highlight` and map backup `D:/UnrealWork/ScanMainGameHighlight/Lvl_MainGame-before.umap`. No Perforce submit or Git push.

## Vent interaction reach and closed-to-open pose (2026-10-21)

The owner confirmed scan must unlock E, and E must rotate the cover. The placed cover had been moved 227 cm away from the opening and its small interaction Box was too high for a player standing on the floor. Restored the actor to `(5177.592224, 1734.775894, 555)` and enlarged only this level instance's editable `InteractionRange` Box: relative center `(-245, 40, 115)`, half extents `(280, 130, 150)`. A real MainGame PIE pawn at `(5055, 1735, 285)` overlapped the Box; scan completion unlocked the cover, simulated E opened it, and the Box disabled itself.

The old native hinge default was visually already open at 0 degrees and rotated farther away to -95 degrees. Moved `CoverHinge` and `CoverMesh` origins to the cover's bottom-left corner, made hinge roll 110 degrees the initial closed pose, and made 0 degrees the open target. The same 0.65-second smoothstep remains. Saved Blueprint CDO and placed actor readback both showed these defaults. MainGame PIE closed/open screenshots, `Saved/Screenshots/WindowsEditor/vent-final-closed.png` and `vent-final-open-framed.png`, were visually inspected: first covers the hole, then swings to the left-side open pose. Development Editor build succeeded, Blueprint opened/compiled/saved, editor finished on MainGame with no dirty content/maps. The Infiltrator's actual F key was not replayed in this final visual test; earlier GoldHighlightLifecycle PIE verified the scan gate. Checkpoint `refs/wip/before-robotvent-range-fix`, pre-write map/source/Blueprint copies under `D:/UnrealWork/ScanMainGameHighlight/`. No Perforce submit or Git push.

The `SpiderTankVent` route test now verifies that the closed cover blocks its entrance, opens the transient PIE cover, and then checks the complete robot capsule route. Final Development Editor rebuild and `TheManTest.Player.Infiltrator.SpiderTankVent` automation succeeded; evidence `D:/UnrealWork/ScanMainGameHighlight/EditorBuild-final.log` and `SpiderTankVent-final.log`.

## Powered SpaceStation door and conditional prompts (2026-10-22)

The user requested a powered door and context-sensitive interaction text. `AInteractableBase` now exposes `GetInteractionPromptState(Viewer)` with label, visibility and E-enabled flags. Its default follows the old `CanInteract` scan gate, so the vent remains hidden until discovered. `ATheManPlayerController` reevaluates the prompt even while focus remains on the same actor and only adds `[E]` for enabled actions. The door's native `APoweredDoor` uses `ATheManGameStateBase::IsPowered()`: off shows `No power`, on shows `[E] Open door`. An instance can allow Both, Front (+X), or Back (-X); an excluded side has no prompt and cannot open it. Seven panels from the existing SpaceStation door meshes slide/rise with smoothstep on E, then disable panel collision and interaction overlap. New Blueprint `/Game/Actors/Interactables/PoweredDoor/Blueprint/BP_PoweredDoor`; the original `BP_door_01` and its 22 SpaceStation instances were untouched. MainGame placement was not made because the merged map remains locked by another user. Team setup: `Documentation/Interaction/PoweredDoor.md`.

The user also specified English UI text, including prior Infiltrator Spider Tank warnings. All three existing runtime warnings were translated to English. Development Editor/Game builds and focused TestMap PIE `TheManTest.Interaction.PoweredDoor` passed, including live HUD changes while standing at the same door, both-side selection, seven mesh assignments, one-time opening and final collision. Existing `GoldHighlightLifecycle` PIE also passed, confirming the vent remains scan-gated. Evidence `D:/UnrealWork/PoweredDoor/`. WIP ref `refs/wip/powered-door-before`. No Perforce submit or Git push.

## MainGame powered door and spawn interaction route (2026-10-22)

After syncing MainGame depot map #5, placed `Spawn_PoweredDoor` at `(1600, 590, 0)` in the entrance near PlayerStart, scale `(1, 1.28, 1)` to fit the approximately 320 cm doorway, and `Allowed Side = Both sides`. The player begins on the door's -X/back side. `Spawn_MainPowerSwitch` and `Spawn_ToggleSwitch` flank the door; three powered flames and a tagged PointLight lie beyond it. All seven new actors are in `Level01/SpawnInteractions`. The existing RobotVent cover remains at its vent opening and is still scan-gated. The generic switch has its static mesh set on the map instance but no action implementation. Setup: `Documentation/Interaction/MainGameSpawnInteractables.md`.

Cold map reload passed with 1967 actors. Live MainGame PIE verified the door's initial unpowered state, then directly invoked `Interact` after the breaker powered GameState: `is_open()` became true, `is_opening()` false, and both door mesh collision states became `NoCollision`. The final rendered spawn screenshot is `Saved/Screenshots/WindowsEditor/spawn-final.png`. The map is opened for Perforce edit, the setup document for add; neither was submitted. Backup and logs: `D:/UnrealWork/MainGameSpawnInteractables/`.

## Original SpaceStation door motion clarification (2026-10-22)

The owner asked whether the powered door uses the original door's built-in opening action. It does not: `BP_PoweredDoor` inherits native `APoweredDoor` rather than `BP_door_01`, and reuses only its seven SpaceStation mesh assets. `APoweredDoor::Tick` drives an independent 0.8-second smoothstep animation, moving four side panels outward and three middle panels upward before disabling their collision. The original `BP_door_01` and its placed instances remain untouched. `Documentation/Interaction/PoweredDoor.md` now states this explicitly. No door code or asset change in this clarification.

## Original door motion reproduced in the powered door (2026-10-22)

The owner approved matching the original open/close motion without inheriting its Blueprint. Read-only inspection of `BP_door_01` found a 2.5-second Timeline with five cubic, zero-tangent 0-to-1 tracks: `top1` 0–1 s, `top2` 0.5–1.5 s, `top3` and `bot` 1–2 s, and `side` 1.5–2.5 s. Graph links confirmed `door_mid_01/02/03` rise 50/60/65 cm, while each matching bottom/side left/right panel slides +50/-50 cm. Native `APoweredDoor` now reproduces those timings, easing and distances. Power plus E still starts opening; one second after the last player leaves the interaction Box, the five tracks reverse. Reentry while closing reopens the door; once fully closed another powered E is required. The Box stays enabled so exit/reentry is detectable, while open panel collision is disabled and restored when closing.

Development Editor build passed after the final C++ change. The expanded focused `TheManTest.Interaction.PoweredDoor` headless PIE test passed: staggered positions at 0.5 and 1.5 seconds, final seven-panel pose, occupied hold, one-second exit delay, reverse motion, reentry reversal, full closure and collision restoration. In visible MainGame PIE, the placed door opened to the expected 50/60/65 cm and ±50 cm offsets, then returned to zero after the player left; the open viewport was inspected at `Saved/Screenshots/WindowsEditor/door-original-motion-open.png`. `BP_PoweredDoor` compiled and saved in the editor. Logs: `D:/UnrealWork/PoweredDoor/original-timeline-tangents.log`, `original-motion-test.log`. Scoped Git pre-write checkpoint `bd23af77`; source, Blueprint and team documents were opened for Perforce add, with no submission.


## Powered door access card and non-visual backpack items (2026-10-22)

The owner approved storing the door's no-model, non-switchable access card in the existing EquipmentManager component, separate from its weapon Actor array. Added Blueprintable UInventoryItemDefinition and FInventoryItemStack; EquipmentManager now initializes and queries non-visual item counts. BP_AccessCard lives at /Game/Core/_Shared/Items/Blueprint/BP_AccessCard. BP_TheExecutive initial items contain exactly one card, and BP_Infiltrator has none. BP_PoweredDoor and the placed MainGame Spawn_PoweredDoor require the card. Off shows No power; on without card shows Access card required and rejects direct Interact; on with card shows [E] Open door. Opening does not consume it, and original door motion/closing remain.

Development Editor build passed. All three Blueprints were compiled and saved in the Editor. Cold readback verified both role defaults and the placed MainGame door requirement at D:/UnrealWork/DoorAccessCard/cold-readback.json. Focused TheManTest.Interaction.PoweredDoor PIE passed Success with zero errors, checking no-card denial, live HUD update after adding the card, non-consumption, Executive default count, and existing motion/reclose at D:/UnrealWork/DoorAccessCard/TestReport/index.json. Team documentation: Documentation/Interaction/PoweredDoor.md and MainGameSpawnInteractables.md. Local Git WIP checkpoint e8176ba3; Perforce CL 121864 submitted (13 files); server readback shows no opened files.


## 2026-10-22 proximity outline for all interactables

Owner approved white highlighting on proximity for all AInteractableBase objects regardless of power, card, scan discovery, prompt visibility or E availability. The local controller requests proximity on every overlapped interactable; AInteractableBase arbitrates that request with scan yellow (yellow wins) and restores each mesh's prior CustomDepth/stencil when both end. UScannableInteractableComponent forwards scan start/stop. M_Infiltrator_Highlight selects white for stencil 2 and gold for stencil 1, sampling nearby stencil pixels so the outer outline changes color too. MainGame, SpaceStation and TestMap all cold-read with the postprocess blendable at weight 1. Development Editor build; PoweredDoor and GoldHighlightLifecycle PIE tests passed in NullRHI, plus a rendered-RHI PoweredDoor run. Material graph cold readback: D:/UnrealWork/ProximityHighlight/material-readback.txt. Team setup: Documentation/Interaction/InteractableHighlight.md. Local pre-write Git checkpoint 76d43db4; 10 files opened in Perforce CL 121873, not submitted.

## Robot vent toggle and persistent proximity outline (2026-10-22)

Owner requested closing the previously one-shot vent and noticed white proximity outline vanished after opening. Root cause: Interact disabled InteractionRange and unregistered the actor. ARobotVentCover now toggles bOpened, keeps the overlap Box enabled, updates Open vent/Close vent prompt, and eases between the existing 110-degree closed and 0-degree open angles. A mid-motion press reverses from the current hinge angle. Scan discovery remains required before E; yellow still has priority over white. Development Editor build and GoldHighlightLifecycle plus MainGame SpiderTankVent PIE passed, including open white highlight, close prompt and restored robot-route collision. Vent source and SpiderTankVent test opened in P4 CL 121873; no submit.

## Perforce delivery (2026-10-22)

White proximity highlight, yellow scan priority, toggleable RobotVentCover and associated interaction tests/docs were submitted as CL 121896 (13 files; previously pending CL 121873). The shared SpiderTank vent test file traveled in this changelist. Perforce reports no files open on this client.

## Powered door front-card/back-auto access (2026-10-22)

Owner changed the door policy: local +X front requires power, configured access card and E; local -X back auto-opens for a player in InteractionRange when powered, without card or E prompt. Legacy AllowedSide enum/instance property was removed because both sides now have fixed roles. Closed-door Tick remains enabled so a player already at the back opens when power turns on. During closing, reentry reverses only if power is on and the player is on the back or carries the front card; open-door occupancy still holds the doorway while someone is in range. Existing seven-panel animation, collision, close delay and white/yellow highlights remain. MainGame's placed Spawn_PoweredDoor faces world +X, placing the spawn on its automatic back side. Development Editor and Game builds passed. Focused `TheManTest.Interaction.PoweredDoor` NullRHI PIE passed Success, including front card/no-card, back no-card/no-power, power-on while already at back, door pose and reclose. Evidence: D:/UnrealWork/PoweredDoorSideAccess/automation-final.log and TestReportFinal. Team docs PoweredDoor.md and MainGameSpawnInteractables.md updated. Git safety checkpoints 6d8c090c and 04de95d3. Five files opened in dedicated P4 CL 121899, not submitted.

## Perforce delivery (2026-10-23)

The owner requested upload. The five powered-door source, test and documentation files from pending CL 121899 were submitted as CL 122042. Server readback confirmed submitted.

