# FEAT-256 ? Cursor and Dev maps

User authorized original downloaded SciFiCursorPack and individual map buttons. Scope: four existing maps, retain current character, no level content edits. Source: https://abstergodesign.itch.io/sci-fi-cursor-pack .

Implementation in progress. Recording map requires camera-only loading readiness.
# FEAT-256 cursor and maps
Native build passed. Imported two original PNG textures and authored pointer/hand WBP assets; WBP_DebugMenu compiled and saved with four map buttons. Recording readiness now recognizes its map GameMode and fixed camera. Refinement in progress: place MAPS directly below TESTING; validate real travel and cold cursor configuration. No map content changed.
# Verification update
Cold editor launch loaded the SoftwareCursors configuration. Actual mouse hover shows original pointer and hand artwork. Actual Dev Panel clicks successfully traveled TestMap -> SpaceStation -> TestMap; Executive pawn selection retained and TestMap returns unpaused with loading finished. MAPS is now directly after TESTING. Remaining: lobby/recording completion and final saved palette consistency.

## Completed
- Original pointer/hand artwork imported as UI textures; editable 36px software cursor widgets use offset matching click tip. Config loads both on cold editor start.
- Four separate map buttons in MAPS immediately below TESTING; original TestMap button relocated, no duplicate. Palette copied to match other controls.
- Final Development Editor Win64 build succeeded (build-delivery.log). Widgets opened/compiled/saved in live editor.
- Actual mouse clicks: TestMap -> SpaceStation -> TestMap -> LobbyMap; Lobby normal start -> SpaceStation -> ScorpionRecording. Logs show all transitions finish. Executive retained on combat travel; TestMap and Recording unpaused; Recording correctly has no pawn and fixed camera. All source cursor assets load, pointer and hand visible in hover screenshots.
- saved.json confirms dirty content/maps empty. No level edits, packaging, version change, retargeting, final commit or push. Checkpoint aa3b4236 preserved prior authorized work.
- Existing Dev Panel remains combat-controller-only: Lobby uses its normal menu; ScorpionRecording is a camera-only map. No new global input or recording-map player added.

## Main Game shortcut (2026-10-22)

The Dev Panel MAPS section now includes a fifth `Main Game` button in its own full-width first row. `UDebugMenuWidgetBase` binds `Button_MainGame` to the existing `ATheManPlayerController::EnterDebugMap` route with `/Game/Maps/Lvl_MainGame`; `UTheManGameInstance` adds that package to its non-shipping allowlist. `DebugMenuAssetLibrary::AddMapButtons` updates the existing editable `WBP_DebugMenu` idempotently and keeps MAPS after the CHARACTER buttons. The widget was compiled and saved in the editor. Development Editor build passed. Rendered PIE automation `TheManTest.UI.Debug.MainGameTravel` succeeded after clicking the actual widget button and arriving in `Lvl_MainGame`; a NullRHI attempt timed out after initiating loading because that presentation path requires rendered viewport frames. Scoped originals: `D:/UnrealWork/DevPanelMainGame/checkpoint/`. No Perforce submit or Git push.
