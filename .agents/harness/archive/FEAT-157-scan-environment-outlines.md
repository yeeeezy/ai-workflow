> Superseded2026-09-26 by user-approved FEAT163. Legacy ScanContours no longer renders for Infiltrator; history below is retained, no further review needed.

# FEAT-157 Scan environment outline correction

Status: needs_review.2026-09-26 user rejected initial smooth-band version: original scan texture must remain.

Correction saved and verified: restored original wave edge mask, RGB separation and scene tint. Relative to the original pre-task graph, ONLY Add_8.B changes from Add_6 to Multiply_30, bypassing constant environment-outline contribution Multiply_10. All other node connections match original graph exactly. Original wave can still reveal environment edges while it passes/holds; RGB separation also retains its original colored fringes. Do not again replace wave with a solid light band or remove grading to claim all environmental edges disappeared.

Scoped checkpoint205e22a preserves rejected revision and its docs. Editor material compiled/saved; independent cold load confirms the single original-graph connection difference (correction-cold.json/log, exit0 zero errors/warnings). Actual Infiltrator GA PIE expansion/hold/toggle pass, CorrectedWave.png and CorrectedHold.png captured; hold image inspected. No C++/Blueprint/map or target-highlight edits; no fixtures needed this correction. PIE stopped. Pending user visual acceptance. Evidence D:/UnrealWork/ScanNoContours/correction*.json and Corrected*.png. Correction uncommitted, no push/package.

## Superseded initial implementation (rejected by user)

# FEAT-157 Disable scan environment outlines

Status: done. Created/closed2026-09-26. User approved removal of ordinary environment red contours, retaining scan expansion, target highlight and toggle lifecycle.

Only gameplay asset changed: /Game/Characters/Infiltrator/Effects/Scan/Materials/M_Infiltrator_ScanContours. Two connections: Multiply_41.A now receives Multiply_30 directly; LinearInterpolate_2.B receives VectorParameter_1 directly. Final output is original scene RGB + smooth spherical scan band * ScanAlpha. Bypasses persistent environment edges, RGB separation/doubled scene tint and wave edge mask. Original flow color, wave intensity/radius/thickness/feathering/timing/MPC controls remain; old unused graph retained. No target-highlight material, C++ or Blueprint changes.

Scoped checkpoint8b188ed preserves preexisting harness files; other existing assets/deletions untouched. ScanContours was clean before task. Final change uncommitted; no push/package/version increment.

UE editor material recompiled/saved. Actual Infiltrator GA rendered TestMap PIE passes11checks: ordinary-object exclusion, target hold, expansion, second-toggle cancellation and restart. BeforeView.png, AfterHold.png, Wave.png and Off.png inspected: ordinary cube/terrain have no colored edge lines, target remains outlined, smooth scan band visible. Temporary unsaved editor fixtures used for PIE duplication, removed after PIE; TestMap reloaded without saving. cleanup.json confirms no fixtures, dirty content/maps or active PIE. No persistent test assets.

External test helper corrections: unavailable deferred-spawn Python API replaced with unsaved editor fixtures; first save refused during PIE, then succeeded after ending PIE; unavailable material-stat API omitted from report. No production-code workaround. Independent cold commandlet verifies saved connections and reachable graph: only PostProcessInput0 scene texture, no scene depth/normal edge detector. Exit0, zero errors/warnings. preservation.json checks all scan assets and TestMap/SpaceStation hashes, only ScanContours differs. No C++ build needed.

Evidence/scripts D:/UnrealWork/ScanNoContours: author.json, pie.json, cold.json/log, cleanup.json, preservation.json, window.json and captures. Editor placed/verified on freshly detected secondary display; remains clean in TestMap with PIE stopped. Existing TMIIR editor untouched.
