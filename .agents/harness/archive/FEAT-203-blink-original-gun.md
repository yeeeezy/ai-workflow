# FEAT203 Original gun material during blink

Status: done
Closed: 2026-09-27

User identified whole gun purple. Root cause: legacy CloakMaterial replacement remained enabled independently of new Trail21. Disable production BP CloakMaterial only; retain all trail and filter settings. Checkpoint87a709e6. Evidence D:/UnrealWork/BlinkOriginalGun.

Disabled BP_Infiltrator.BlinkPresentation.CloakMaterial=None and compiled/saved. First save was blocked by duplicate same-project editor PID5660 holding package open (Error32). Read-only injected Python confirmed duplicate PIE=False/dirtycontent0/dirtymaps0; gracefully closed duplicate, then saved intended BP in owned editor and closed it cleanly for build. Unrelated UE58Blank left alone. Updated existing native test to assert no cloak and exact original gun material DURING traversal. Cold Development Editor Win64 build24.03s succeeded. All75 ability assets including filter and trails byte-identical to checkpoint.

## Completion

FEAT203 completed2026-09-27: user correctly identified whole-gun purple as legacy blink CloakMaterial substitution, not Trail21. BP_Infiltrator.BlinkPresentation.CloakMaterial cleared to None, compiled and saved in editor. Production gameplay C++ unchanged; existing native test now asserts no cloak and exact original material pointer during traversal. Development Editor Win64 cold build24.03s Succeeded. Actual TestMap23checks passed, including equality of every gun material before/during/after blink, six trail/support components, all four live ribbons,600cm travel, tail cleanup, scan coexistence and pause/resume. BlinkMid.png confirms gun no longer uniformly violet. Native Blink and BlinkKeyboard Success. Independent cold BP/filter/trail checks pass. All75 existing ability packages including filter/trails/scan byte-identical to checkpoint87a709e6. Duplicate same-project editor caused Error32 save lock; verified no PIE/dirty assets/maps in duplicate via read-only Python, gracefully closed it, saved owned BP, then cold built and reopened single target editor on secondary. UE58Blank untouched. Final PIE ended, dirtycontent/maps0. Only production asset change is BP; test assertions updated. Evidence D:/UnrealWork/BlinkOriginalGun. Changes saved/uncommitted; no push/package/version change.
