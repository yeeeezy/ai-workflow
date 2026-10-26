# USC Perforce workspace

LATEST HANDOFF: 2026-10-10 first Perforce upload completed and verified: submitted change119513 (server renumbered pending119509),6069files,7403430973bytes. Server describe exact depot-path set/all-add matches reviewed SHA256 manifest; client opened0; original placeholder.txt#1/change119315 retained. Includes current project source/assets and recent disintegration work; generated/cache/local MCP/Git/agent state excluded. No Git commit/push or gameplay changes during upload. Evidence D:/UnrealWork/PerforcePortability/upload-verified.json. Custom Void debris remains paused separately.

Current status (2026-10-09): portability cleanup and English packaging prompts implemented and verified; first Perforce upload is still pending. The historical dependency-review items below are superseded by the completion entry at the end.

USC Perforce setup - 2026-10-09
Server: ssl:perforce.sca.usc.edu:1666
User: zying242
Workspace: zying242_LAPTOP-4EDDIURP_TheManIInvented
Root: D:\Unreal Projects\TheManTest
Mapping: //CTIN_594_20263_20271/TheManIInvented/... //zying242_LAPTOP-4EDDIURP_TheManIInvented/...
Host: LAPTOP-4EDDIURP
Options: noallwrite noclobber; SubmitOptions: revertunchanged
Detailed exclusion mappings: client-final.txt; add-ignore rules: project .p4ignore.
Global defaults and three existing user workspace specifications verified unchanged.
No add/edit/delete opened, sync, move/copy, or submit. Existing server placeholder.txt#1 left untouched.

UPLOAD PREVIEW: 5979 files, 6.874952 GiB
Content 5421; Source 504; Config 6; Plugins 11; Scripts 29; SourceArt 2; root 6.
Full path/size/type list: upload-manifest.csv / upload-manifest.json.
Root files: TheManTest.uproject, PackageGame.cmd, .p4ignore, .vsconfig, .gitignore, .gitattributes.
Source plugin BBBAimIK runtime/editor source and descriptor included. Engine plugins use UE5.7 (harness specifies5.7.4).
Includes maps, BuiltData, __ExternalActors__, __ExternalObjects__, audio and opening movie. Build folder contains logs only and is excluded.
No Git LFS pointers in .uasset/.umap. Server typemap already applies binary+l to Unreal assets/maps; no typemap edits.
Excluded: Binaries, Intermediate, DerivedDataCache, Saved, plugin generated output, logs/temp/bytecode, .git repository, agent/MCP/IDE local state, local P4 connection/launcher.

DEPENDENCY REVIEW BEFORE FIRST SUBMIT:
1. TheManTest.uproject enables external McpAutomationBridge and absolute AdditionalPluginDirectories D:/Unreal Plugins/Unreal_mcp-v0.5.30/plugins. Plugin is not in project upload. Recommend optional editor-tool dependency/local setup before collaborators download. No descriptor edits made.
2. Config/DefaultGame.ini contains a local MCP settings section alongside required gameplay/package config. Recommend remove only this section from shared defaults in the approved portability cleanup. No existing config edits made.
3. Editor-only Medusa authoring helper mounts D:/Unreal Projects/UE58Blank/Content/ParagonSevarog. That authoring command requires external source resources; normal runtime does not call the helper.
4. Preview reflects current on-disk project, including existing uncommitted edits and unfinished GroundImpact assets/source from paused FEAT272. Nothing was reverted or committed to Git.

LOCAL CONNECTION:
.\P4Local.cmd info
.\P4Local.cmd opened
Launcher uses setlocal and explicit project connection/workspace arguments. .p4config is local-only; global default workspace remains DreamCatchers. No persistent p4 set performed.

Await user approval of upload scope and portability preparation; do not add/submit yet.

## Portability cleanup and English packaging prompts — 2026-10-09

User explicitly approved splitting machine-specific MCP settings from shared project configuration and requested English packaging scripts. Selective checkpoint `9c996ff8` preserves known prior harness/Void work, current v0.8 config and .p4ignore; unrelated user blink/rifle/external-actor-deletion/cache edits were excluded and remain unchanged.

- Removed AdditionalPluginDirectories absolute path from .uproject. McpAutomationBridge remains Enabled with Optional=true and TargetAllowList=[Editor]. Other plugins/modules unchanged.
- Created local `Plugins/McpAutomationBridge` directory junction to `D:/Unreal Plugins/Unreal_mcp-v0.5.30/plugins/McpAutomationBridge`; plugin source remains external. P4 already excluded this path; Git now ignores it too.
- Moved only the MCP settings section from DefaultGame.ini to ignored Config/UserGame.ini. UE Game config hierarchy explicitly reads this override. All seven original values, including localhost-only connection, port3000 and project instructions, are retained. Gameplay cues, movie/packaging configuration and version v0.8 unchanged.
- Added Config/UserGame.ini to .p4ignore and .gitignore; Git also ignores the existing .p4config and P4Local.cmd. `git check-ignore` and local `p4 ignores -i -v` confirm exclusion; no server changes or uploads.
- Converted interactive package.py quality/configuration menus, prompts and invalid-input messages to English. PackageGame.cmd now prints its own English close prompt and suppresses the OS-localized pause prompt. Underlying compiler/OS output retains its native language. Packaging behavior unchanged.

Verification evidence: `D:/UnrealWork/PerforcePortability`.

- Local Development Editor Win64 build succeeded75.98s; build-local.log. Actual editor settings read-back confirms every original MCP value, dirty content/maps0; local-settings.json. MCP HTTP initialize and tools/list pass,23tools and project instructions verified; mcp-check.json. Sole primary monitor detected; editor window placement checked, then editor closed cleanly.
- CleanProject is a separate source/config/plugin copy excluding MCP and UserGame.ini. Its Development Editor Win64 build succeeded69.07s; build-clean.log. Commandlet initialized without MCP, loaded TheManGameInstance and TestMap, dirty content/maps0; clean-load.json/editor-clean.log. Content was linked read-only in practice (no saves requested); engine reported3 junction-path capitalization warnings and0errors. Build logs contain existing first-header-order diagnostics despite Succeeded; no unrelated source changes made.
-13 existing packaging orchestration tests pass; English interactive dry runs cover High/DebugGame and All/Shipping. No real build package/cook/stage, no version increment; remains v0.8. git diff --check passes.

Operational caveat: plugin settings UI calls SaveConfig and can reintroduce MCP settings into DefaultGame.ini. Edit local UserGame.ini directly and restart editor for local settings; inspect shared config diff before submission. The external Medusa authoring-only source resource requirement is unchanged and does not block normal runtime. Prior5979-file upload preview predates this cleanup; regenerate before upload. Current on-disk scope still includes paused FEAT272 unfinished assets/source. No add/sync/submit/push; final changes remain uncommitted pending user instruction. FEAT272 remains paused.

## Upload attempt and user pause

User requested Perforce upload. Initial read-only info failed with WSAETIMEDOUT to68.181.26.43:1666, before authentication. Requested interactive login windows used explicit zying242/project client; no passwords or tickets read or recorded. P4V also launched with explicit connection after clarifying UI versus CLI. Existing global defaults remain user junhangm/client DreamCatchers. DreamCatchers historical records confirm ordinary login under junhangm (September17 and25); its saved editor configuration matches the same SSL server. Read-only login-status checks directly in that original directory and with explicit TheManTest arguments both timed out. DNS resolves correctly, USC public HTTPS connects, target1666 does not. Cause is not isolated; VPN requirement is unproven. User accepted connection issue and explicitly paused work.

External prepare_upload.py prepares a current local manifest with ignore checks and hashes; it is not a replacement for server reconciliation. P4 ignores output needs parsing of the trailing ' ignored' and case-insensitive Windows paths; failed intermediate preparation attempts opened/submitted nothing. Local P4Local.cmd uses relative P4IGNORE=.p4ignore with explicit -d project root. On resume verify connectivity/login as zying242, inspect opened/head state, refresh reconcile and upload manifest, then submit within the already-authorized project scope. No new request for upload permission is needed unless scope changes. FEAT272 remains paused separately.

2026-10-10 connectivity recheck requested by user: DNS68.181.26.43, TCP1666 connected in0.01s, explicit zying242/project-client p4 info succeeded over SSL. Yesterday network timeout is resolved at this check. p4 login -s and opened both report Perforce password (P4PASSWD) invalid or unset; authenticated operations require login. No credential values read, no login attempt/add/sync/submit; this request was status diagnosis only.


## First upload completed - 2026-10-10

2026-10-10 first Perforce upload completed and verified: submitted change119513 (server renumbered pending119509),6069files,7403430973bytes. Server describe exact depot-path set/all-add matches reviewed SHA256 manifest; client opened0; original placeholder.txt#1/change119315 retained. Includes current project source/assets and recent disintegration work; generated/cache/local MCP/Git/agent state excluded. No Git commit/push or gameplay changes during upload. Evidence D:/UnrealWork/PerforcePortability/upload-verified.json. Custom Void debris remains paused separately.

Current manifest adds90paths and removes0 versus historical5979preview. Reconcile preview and opened paths reviewed exactly; Unreal packages binary+l verified before submit. All current files rehashed before submission. No sync or delete; global Perforce defaults unchanged. Submit command returned success; initial post-submit verifier queried old119509 and received no-such-changelist because server renumbered it to119513. Corrected read-only verification against119513 passed; no second submit. Submit log and server description retained externally. Do not rerun upload_current.py: it is a one-shot initial-upload script tied to the old pending change. Future updates must prepare a fresh delta/changelist.


2026-10-10 user requested submission: Perforce119522 submitted and server exact4file list verified (package.py/version.py/test_version.py/README.md); opened list empty. No Git result commit/push. LobbyPresentationWidgetBase.cpp and DebugMenuWidgetBase.cpp already read ProjectVersion from GGameIni without old-format parsing; no UI source/asset edit required. Current DefaultGame.ini remains v0.10 because no package was run; next canonical package stamps dated V1 and game UI reads it. Evidence D:/UnrealWork/PerforcePortability/dated-version-submitted.json.

2026-10-11 live save blockage diagnosis: explicit project p4 info succeeds; login -s reports session expired for zying242. Editor SourceControlSettings has correct server/user/project workspace but Provider=None after failed reconnect. Exactly one dirty package: GunFraming TestMap external actor /Game/__ExternalActors__/Maps/Test/TestMap/A/0F/5J527LSXV08USDIYLMHSIY; actual extension .uasset (dirty-map API classification does not imply .umap), disk read-only. P4V started with explicit project connection, PID28744 title Perforce Password Required; waiting for user local login, no credentials requested in chat. Original package and latest Auto1 (2026-10-11 14:03:54 local) copied to D:/UnrealWork/DisintegrationGrip/SaveRecovery. Frozen target JSON/screenshot under GunFramingTargets/DisintegrationGun-20261011 also preserve location/rotation/0.3scale. No force close, chmod, checkout, sync, revert or submit for this diagnosis. Pose/icon integration paused at user's request; pending built-in image generation may complete independently. After login verify ticket, scoped fstat/opened and checkout/save only this actor, then confirm zero dirty packages before closing if requested.


2026-10-11 SAVE RESOLVED: User logged in; project ticket valid. Scoped p4 edit and UE save_packages succeeded for GunFraming external actor 5J527LSXV08USDIYLMHSIY.uasset. Verified zero dirty packages and preserved location(3,8,-9), Pitch1/Yaw-3/Roll0, scale0.3. Editor subsequently exited normally. Asset remains opened in default changelist; no submit. UE Provider remains None; live reconnect not verified, check connection on next launch. Evidence D:/UnrealWork/DisintegrationGrip/save-restored.json. Underhand support pose and lobby icon integration remain pending.

2026-10-14 user requested all current workspace changes be submitted, including unfinished work. Reconciled nine ground movement paths; combined default and pending 120292 into one 67-file snapshot. Server submitted change **120309** (renumbered from pending 120292): 44 edits, 17 adds, six deletes. Server path set equals pre-submit manifest exactly; client opened zero, reconcile preview zero. This includes Void GAS/VFX and FEAT-284 movement but does not mark unfinished features done. The earlier whitebox document-only change **120307** remains separate. Local empty `Content/Maps/Level01/` cannot be represented in Perforce until it contains a real file or map asset. GitHub was not pushed.

2026-10-14 Level01 placeholder follow-up: user asked to version the pre-created folder structure. Perforce **120313** submitted five `.p4keep` files under `Content/Maps/Level01/{Blueprint,Meshes,Materials,Textures,Data}` and revision 2 of `Documentation/LevelDesign/WhiteboxFileStandard.md`, exactly six paths. The standard now documents placeholder removal when real assets arrive. Server documentation bytes match local; client opened zero. No map asset was created, and GitHub was not pushed.
