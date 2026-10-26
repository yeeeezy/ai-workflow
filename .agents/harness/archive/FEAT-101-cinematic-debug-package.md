# FEAT-101 Cinematic Windows DebugGame package

Status: done (latest package delivered2026-09-17; known material fallback below)

User requests highest-quality debug package. Installed UE supports DebugGame game configuration (not Debug engine). Preserve symbols, use Cinematic scalability and 100% resolution. Cook existing three maps and validate packaged runtime. Checkpoint e6c8e40 preserves red scan material. Output/evidence external to project.
`nBuild blocker: UHT game target cannot resolve editor-only UControlRigBlueprint in public helper UFUNCTION. Interface now accepts runtime UBlueprint and validates the exact subtype inside WITH_EDITOR; editor behavior preserved.
Second blocker: PhantomFeatureTests included editor headers in DebugGame. Added WITH_EDITOR guard consistent with all other PIE test files. Editor target already compiles successfully.
Milestone: Development Editor and Win64 DebugGame compiled successfully after the two editor-boundary fixes. Full cook is running (DX12 SM6 and configured DX11 SM5 fallback).

## 2026-09-17 packaged-exit investigation

- `D:/UnrealWork/DebugPackage/package.log` confirms BUILD SUCCESSFUL / AutomationTool ExitCode=0 at 00:53:58 local. Archived package: `D:/UnrealBuilds/TheManTest/DebugGame-Cinematic-20260917/Windows`; DebugGame executable and PDB exist.
- `Lobby.log` loads LobbyMap successfully at 00:56:57 and confirms Cinematic quality groups=4, screen percentage=100. Log stops at 00:56:59 during rendering, without fatal error, assertion, exit status or callstack. No Lobby-result.json or captured screenshot exists; packaged startup validation is incomplete. Do not infer a GPU crash from the final rendering messages.
- SmartObject reports `A valid space partition class name is required` before successful map loading; causal connection to the exit is unproven. CrashReportClient.ini/Session CrashGUID alone does not prove a crash. Latest CrashReportClient log belongs to editor regression and records ExitCode=0; EnemyBaseHealthBar regression passed.
- User reported apparent crash. Read-only investigation found no matching packaged crash dump/report in inspected project, output or validation directories. Retried existing external verify.py Lobby, but it failed at secondary-monitor detection (`assert ms`) before launching the game. Resume rendered reproduction when a non-primary monitor is available; capture exit code and crash evidence before choosing a fix. No gameplay/code/config/assets changed in this investigation.

## 2026-09-17 latest repackage

User explicitly requests highest-quality package after FEAT-106. Existing Cinematic groups4/native100% defaults retained. Reuse Win64 DebugGame with symbols; new timestamped output preserves old package. Latest saved RepairGun Blueprint included without editing its parameters. Build/cook/stage and runtime checks pending.

Packaging blocker found and repaired: cook failed ExitCode25 because WBP_PauseMenu retained a WidgetVariableNameToGuidMap entry for removed PausedWorldBlur. UE5.7 WidgetBlueprintCompiler automatically removes stale entries while reporting ensure. Loaded/compiled/saved only the affected WidgetBlueprint in hidden editor to persist the engine cleanup; no layout, visual material or behavior edits. Re-running full cook. Evidence pause-repair.json/repair-pause.log; initial failure retained in package.log.

## Latest package delivery

- package-final.log BUILD SUCCESSFUL / ExitCode0. Output D:/UnrealBuilds/TheManTest/DebugGame-Cinematic-20260917-103443/Windows (2.51 GiB including symbols). Lobby,SpaceStation,Test each ran20 seconds with DX12 offscreen rendering, loaded correct world, all ten sg groups4 and r.ScreenPercentage100, exited0 without timeout; validation.json ok=true. WBP_PauseMenu stale PausedWorldBlur GUID removed by engine compile/save to unblock cook. Runtime M_UE4Man_Body default-material fallback in gameplay maps and SmartObject warnings remain; no interactive UI/playthrough acceptance claimed. Evidence D:/UnrealWork/CinematicPackage-20260917-103443
- New package preserves original earlier folder. Checkpoint98636f6 contains FEAT-106 and latest saved RepairGun Blueprint; packaging only changed WBP_PauseMenu stale editor metadata plus harness. No final commit/push.
- Prior user-reported visible-window exit was not reproduced by this offscreen smoke, but its historical cause remains unproven. No visible editor/game window opened.
