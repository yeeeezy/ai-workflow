# FEAT-300 Main game default and death respawn map

Status: review pending. Created 2026-10-20.

## User request

Use `/Game/Maps/Lvl_MainGame` as the default map and respawn in that map after player death.

## Baseline and decision

- `Config/DefaultEngine.ini` previously booted `LobbyMap` and opened `TestMap` in the editor. `Config/DefaultGame.ini` cooked LobbyMap, TestMap and SpaceStation, but not Lvl_MainGame.
- `UTheManGameInstance::HandlePlayerDeath` previously sent deaths to `LobbyMap`; `bPendingTransition` was cleared only when a user selected a character or used debug travel. Direct death-to-game travel therefore needs a reset after the destination world loads to support repeated respawns.
- Explicit pause-menu ReturnToMainMenu and terminal HandleGameOver remain directed to LobbyMap. Selected character ID and carried round number are retained through death; the map's GameMode/GameState retain their normal spawn/next-round behavior.
- `Content/Maps/Lvl_MainGame.umap` already exists locally and contains a PlayerStart according to the level asset guide. The map is currently untracked; it is not edited by this feature.

## Implementation and verification

- `GameDefaultMap` and `EditorStartupMap` now point to Lvl_MainGame. Added Lvl_MainGame to `MapsToCook`.
- Added an editable `RespawnMapName` defaulting to Lvl_MainGame. Death travel uses it. `OnWorldChanged` clears the death-travel guard when the destination world arrives, allowing a later death to reload again.
- Git WIP snapshot before changes: `refs/wip/before-main-game-default-respawn` (`1b23891cb5dbb262268ab86ca3187e22c3656f5d`). Existing unrelated dirty work was preserved.
- Development Editor build passed after the final changes. Config readback confirms both startup maps and MapsToCook point to Lvl_MainGame; the local `.umap` exists.
- Focused PIE automation `TheManTest.Core.MainGameRespawn.Twice` passed (`D:/UnrealWork/FEAT300/main-game-respawn-verified.log`, exit 0). Both death requests reloaded Lvl_MainGame, retained the same GameInstance and handed over round numbers 1 and 2. Initial attempts exposed a test-only stale PIE world reference and a real request-failure lock issue; the test now reads the active PIE WorldContext, waits for the loading presentation to finish, and `HandlePlayerDeath` releases its lock if `RequestTravel` rejects the request.
- No map content was edited. `Content/Maps/Lvl_MainGame.umap` remains locally untracked, so the map must be included in a scoped Perforce submission with this feature. The project Perforce ticket is expired; no submit was attempted.
