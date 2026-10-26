# FEAT-164 Manual scan smoke palette

Done2026-09-26. User approved replacing surface RGB sampling with editable black/blue palette,retaining shared effect and range/occlusion lifecycle.

AEnemyBase now owns a native UEnemyScanSmokeComponent named ScanSmoke under Enemy/Effects. No tick or gameplay/highlight logic. Editable per Blueprint or placed enemy: PrimaryColor(.002,.003,.006),SecondaryColor(.01,.16,1),SecondaryRatio(.3),Brightness(1). Native inheritance makes it available on Phantom and other enemies without Blueprint-only component copies. Existing smoke actors read live settings each simulation step.

Two existing materials updated in Unreal: Dye advects primary/secondary density in R/G,procedurally distributing the injected silhouette density according to SecondaryRatio; captured RGB never enters colour. Composite uses total density for opacity and density-weighted palette for colour,so zero/black colours remain visible occluding smoke. Brightness changes colour,not opacity. Capture still provides the enemy shape. Solver,mesh capture,range/through-wall behavior,highlight eligibility and cleanup unchanged. No extra per-enemy material/system assets.

Verification: Development Editor Win64 build-fixed.log Succeeded,no new compiler warnings. First build had TObjectPtr auto deduction error,fixed with Get(). Unreal material graph intermediate edits emitted missing-input compile warnings until all new pins were connected; final independent cold.log recompiles cleanly without material errors/warnings. BP_Phantom opened/compiled and native component defaults verified in author.json and cold.json. No BP save needed for inherited native defaults.

Rendered TestMap:13 existing path checks plus7 palette checks plus21 lifecycle checks all pass. Independent pure-primary/pure-secondary dye density sums5265/12.6 and35/5250;blue density channel excluded when ratio0,primary excluded when ratio1 after advection settles;dye B always0. Live PrimaryColor/Brightness changes read back from actual MID. BlackBlue.png/ScanSmoke.png inspected: black and blue flowing wisps including wall-occluded enemy. Outside/reentry,held-scan newcomer,real GAS death,destruction,second toggle/unpossess cleanup pass. Original opt-in highlight unchanged,no automatic enemy highlight.

Palette probe initially used MaterialEditingLibrary MIC-only getter on MID; corrected to MID getters,all checks pass. Cleanup.json zero dirty maps/content. PIE ended; no permanent fixtures saved. Cold editor successfully reloaded/saved material graphs and component defaults then exited normally after its script; no background scan preview remains. Verification window was on freshly detected secondary[2572,12,4468,1042]. Source projects untouched.

Evidence D:/UnrealWork/ManualScanSmoke. Scoped WIPb7584b3 protects preceding implementation;archive-only WIP4870321 protects FEAT163 history. Final feature uncommitted,no push/package/version change.
