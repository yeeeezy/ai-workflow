# FEAT-133 Tab debug menu

Status: done. Closed: 2026-09-19.

## Scope and final behavior

User requested the debug UI and Tab toggle first, with God Mode as its first planned feature, then supplied D:/Perforce/zying242_LAPTOP-4EDDIURP_DreamCatchers as the visual reference. Final steering explicitly requires Tab to open AND close, preserving Escape and operating visible windows on the secondary monitor. This supersedes the initial Escape/close-button draft and earlier primary-screen preference.

Editable /Game/UI/Debug/WBP_DebugMenu follows DreamCatchersDebugPanelWidget.cpp: dark full-screen translucent background, 48-unit padding, upper-left cyan DEV DEBUG, muted Tab to close, cyan PLAYER section, disabled God Mode: OFF button. God Mode has no invulnerability logic in this UI-only batch. Reference project was read only.

ATheManPlayerController owns IA_DebugMenu, DebugMenuWidgetClass and the transient instance. /Game/Core/Input/Actions/IA_DebugMenu maps Tab once in IMC_Default; BP_TheManPlayerController supplies the references. Open is local combat only, cancels ADS through the existing release event, pauses, collapses combat HUD, shows cursor and focuses UIOnly. Close restores GameOnly, pause, cursor and HUD, flushing held keys. Debug and existing pause menus cannot overlap. Unpossess and EndPlay clean up.

UDebugMenuWidgetBase handles Tab in NativeOnPreviewKeyDown, ignoring repeats; it has no Escape handler. Escape while debug is open leaves it open. After Tab closes debug, the original Escape pause/resume works. Lobby input is unchanged.

Layout stays in the Widget Blueprint. Editor-only UDebugMenuAssetLibrary initializes an empty asset or the known initial DebugCanvas draft, and refuses overwriting the completed DebugBackground layout. No runtime layout construction.

## Verification

Evidence: D:/UnrealWork/DebugMenu.

- build-final.log: Development Editor Win64 succeeded.
- cold.json: saved widget hint, controller reference and single Tab mapping verified.
- runtime.json: all 19 PIE checks pass, including modal exclusivity, frozen world time, repeat toggle, HUD/cursor/input restoration and unpossess/repossess cleanup.
- Physical-EscapeUnchanged.png, Physical-TabClose.png and Physical-Pause.png: real window Tab/Escape input and resulting UI visually reviewed. Final-Secondary.png records reference-style UI on secondary monitor.
- editor-final.json: affected Blueprints opened, compiled and saved; PIE ended, no dirty packages/maps.
- secondary.json: connected secondary detected and all three editor windows verified inside its work area. Owned standalone test was stopped after validation.

Temporary test scripts were corrected for UE Python API names, generated-class duplicate object lookup and secondary-monitor capture/window selection. Initial save lock from the owned standalone process was resolved by stopping that process; final compile/save passed. No unresolved implementation blocker.

## Handoff

Checkpoint 8db4b68 protects previous scoped-rifle ADS fix and documentation. Final debug changes remain uncommitted; no push/package. Preserve unrelated user edits and current silver-only initial loadout. FEAT-131 remains open for broader appearance acceptance. God Mode behavior requires a subsequent requested implementation. Future visible operations must use the secondary monitor when available; do not add a recurring window-position lock.
