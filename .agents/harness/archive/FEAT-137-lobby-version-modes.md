# FEAT-137 Lobby version and display modes
Status: done
Updated: 2026-09-19

User authorized removal of Fullscreen, retaining Borderless and Windowed; clarified v0.1 -> v0.2 -> v0.3 on successful builds, displayed on all lobby pages. Checkpoint9fc13df protects completed136. Unrelated gun/map edits preserved.

## Implementation
Two display options use explicit enum mapping, avoiding index confusion after removing Fullscreen. Legacy exclusive fullscreen drafts as Borderless until Apply. Global editable VersionLabel moved to PresentationCanvas above Settings overlay; duplicate Settings footer removed. Native initialization reads ProjectVersion. Editor/Game postbuild hooks increment the integer revision through version.py. package.py compiles, increments once, then cooks with managed hook suppression. Locking and atomic config updates; compile failure leaves version, cook failure retains successful build version. Harness policy and arch01/02 synchronized.

## Verification
- D:/UnrealWork/LobbyVersion/build.log: Development Editor Win64 Succeeded; postbuild v0.1 -> v0.2. No C++ edits after this build.
- Scripts/Build/test_version.py: six tests pass: sequence including v0.10, invalid formats, concurrent lock, compile failure, cook failure, one increment before cook. Packaging subprocesses mocked; no actual full package requested/performed.
- pie.json: twelve checks pass across main/character/weapon/settings visibility, hierarchy, two options, legacy mapping, defaults and no duplicate footer.
- Asset authoring initially emitted GUID bookkeeping ensures when adding/removing named widgets. Subsequent compile.py cleanly compiled/saved both widgets with CLEAN_COMPILE_OK, no errors.
- Actual standalone physical clicks on secondary: only BORDERLESS/WINDOWED visible; Windowed -> Borderless applied and kept at secondary1920x1080. Return to Windowed produced confirmation countdown, then automatically restored Borderless. Screenshots D:/UnrealWork/Settings/Version-Modes.png, Version-Confirmed.png, Version-Revert-Before.png, Version-Revert-After.png. Initial physical scripts missed clicks on foreground acquisition and are not evidence of successful switching; corrected consecutive-click runs above verified the UI.
- editor-final.json: PIE stopped, zero dirty content/maps. Editor27376 and standalone16692 on secondary; no positioning loop. Standalone left on lobby main menu, borderless1920x1080, audio100%.
- git diff --check passed. Current config v0.2. No final commit/push/package; old FEAT135 remote LFS budget block untouched.

FEAT-131 remains active for its unresolved silver-gun appearance scope. Do not rerun weapon authoring scripts or modify framing as part of this feature.

## 2026-09-21 user correction: package only
Earlier interpretation was wrong: user requires one increment per packaging, not per compilation. Removed both Target.cs postbuild hooks; version.py direct execution is now a no-op (also protects stale receipts). package.py remains the sole increment caller. Current v0.18 retained rather than renumbering historical work. Ordinary Editor compilation actually compiled the new recording mode and linked successfully, version still v0.18. Six packaging orchestration tests pass (mocked build/cook); direct version.py with and without managed flag leaves config byte-identical. No real package generated. Evidence D:/UnrealWork/ScorpionRecording/recording-build.log. Arch01 and harness policy corrected.

## 2026-09-21 authorized version reset
User explicitly requested restoring version to0.1. Changed only ProjectVersion=v0.18 to v0.1 in DefaultGame.ini; other config bytes unchanged. Package-only increment rule retained, next package reserves v0.2. No build/package or editor restart; existing running editor may retain cached v0.18 until restarted.


## 2026-10-10 UTC milestone version scheme (supersedes numeric increment)

2026-10-10 dated package versions implemented: V<phase>-yyMMdd-HHmmss using one UTC timestamp per invocation. Milestone dates from screenshot 2026-10-10 163120.png inclusive; phase increments at following UTC midnight: V1 Sep26, V2 Nov14, V3 Dec19 2026; V4 Feb20, V5 Mar20, V6 May1, V7 May12 2027. Current phase V1. Compile success stamps DefaultGame.ini; all quality variants share it; compile failure leaves old value; cook failure retains stamped value. Legacy v0.N accepted only as migration input. Required read-only configs are p4-edited before compilation using caller workspace; failures abort early. No auto sync/submit/revert.17 orchestration tests PASS including all boundaries/timezone/checkout/same-second protection; actual project dry-run previews V1. No real compile/cook/package, no production version stamp; current saved version remains v0.10 until next package. Final scripts opened for P4 edit, not submitted; no final Git commit/push. Checkpoint cbd8138a. Evidence D:/UnrealWork/PerforcePortability/dated-version-tests.log. Shared usage guide Scripts/Build/README.md. Separate-machine same-second collisions remain possible; no globally unique ID claimed.


2026-10-10 user requested submission: Perforce119522 submitted and server exact4file list verified (package.py/version.py/test_version.py/README.md); opened list empty. No Git result commit/push. LobbyPresentationWidgetBase.cpp and DebugMenuWidgetBase.cpp already read ProjectVersion from GGameIni without old-format parsing; no UI source/asset edit required. Current DefaultGame.ini remains v0.10 because no package was run; next canonical package stamps dated V1 and game UI reads it. Evidence D:/UnrealWork/PerforcePortability/dated-version-submitted.json.

2026-10-11 user asked why Lobby version is old. Read-only diagnosis: DefaultGame.ini still ProjectVersion=v0.10; Lobby and Debug UI both read GGameIni directly. New dated rule is package-only and no package has stamped it yet. Explained distinction; no version/config/package change.


2026-10-11 packaging follow-up: user reported checkout failure. Root cause was missing P4CONFIG selection, causing global junhangm/DreamCatchers settings; project zying242/TheManIInvented ticket was valid. version.perforce_environment selects existing project .p4config via child-process-only P4CONFIG; absent file preserves caller environment. No hardcoded identity/global setting changes. Real read-only login/info via same helper verified correct project user/client/root. Output folders and delivery README/menu use TMII; folder format TMII-V<phase>-yyMMdd-HHmmss-Win-<configuration>[-High|Medium|Low]. Win label describes existing Win64 target only; no Mac build support added. Default output parent remains D:/UnrealBuilds/TheManTest. Dry-run includes exact delivery paths.19 orchestration tests PASS including local-config selection/environment isolation/fallback, naming, existing-delivery protection, version and quality recovery. No real build/cook/version stamp. Scoped checkpoint9c57037f; four Scripts/Build files opened for P4 edit, final changes uncommitted/unsubmitted. Evidence D:/UnrealWork/PerforcePortability/local-config-tests.log.


2026-10-11 packaging compile correction: hidden validation editor PID30752 and LiveCodingConsole33480 remained alive. Bridge confirmed zero dirty packages, normal quit completed and both processes exited. Failed package log also identified23 own-header-first violations. User authorized repair; moved each matching own header to first include, exactly23 insertions/23 deletions, no gameplay changes. Actual Build.bat TheManTestEditor Win64 Development succeeded20.46s and TheManTest Win64 DebugGame succeeded26.77s. Evidence D:/UnrealWork/PerforcePortability/build-header-editor.log and build-header-debuggame.log; file list header-order-files.txt. No cook/archive/package/version stamp performed. Checkpointb48d564f; final source changes not committed/submitted. Editor left closed for user packaging.
