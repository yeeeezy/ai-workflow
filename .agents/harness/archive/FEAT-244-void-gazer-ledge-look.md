# FEAT-244 Void Gazer ledge look

Status: done
Closed: 2026-10-06
User approved stop/look-at-player/resume with suitable LedgeAnimationSet motions. Inspect and retarget only in TMIIR, finalized assets only to target. Existing Medusa untouched. Checkpoint3b8fc13e preserves FEAT243; user config/rifles/external actors excluded.

## Implementation progress
- Confirmed original IP_Ledge_Lean_L/R/Out each Start1.0667s/Hold1.6667s/Stop1.0667s. Source mannequin hangs vertically; separate source-only Ledge rigs retarget spine and neck/head onto Void. Nine final clips preserve native tentacles and bake pelvis rotation-90deg around meshX plus reference offset to match wall-crawl coordinates. Original vendor clips unchanged. Final source finite-pose/dependency checks passed; AssetTools migrated nine clips with existing dependencies skipped.
- Added timed native look phases and left/right/back selection based on target position in wall basis. Automatic player check every6s within12m and line of sight. Crawling input preserved during look; explicit stop/death/support loss clears it. Nonloop entry/exit and looping hold use existing pose-transition proxy. No navigation/combat or Medusa changes.
- Source helper build2.26s and target Development Editor15.24s succeeded. Initial source build refused while source editor was still exiting; repeated after process exit. Target visual/runtime verification underway.

## Verification
- Target live BP opened/compiled/saved. Cold38packages and nine BP clip references pass,0errors/0warnings. Source final pose samples finite and dependencies restricted to own semantic directory.
- Actual TestMap SIE1292 assertions (16distinct primary assertions plus per-frame no-motion/speed guards) pass for Left/Right/Back, non-restarting requests, full Start/Hold/End/Crawl phases, and resumed up movement. Left/Right/Back screenshots captured; Left/Back visually inspected.
- Normal TestMap PIE15lifecycle/automatic checks pass using actual possessed playable character: explicit stop, target destruction, resume, range exclusion, behind-wall exclusion, automatic player glance, support removal cancellation, reattachment and continued crawling. SIE originally had editor spectator rather than gameplay character; switched to normal PIE instead of changing production behavior to accommodate fixture.
- Source bAuto property readback/EditorPerformanceSettings Python exposure and get_pawn naming issues were script-only; fixed by reacquiring compiled CDO, reflected class path and real PIE controller. Earlier target-error.txt is stale; final runtime.json/extra.json are successful.

## Handoff
Editor21088, sole primary display, TestMap SIE temporary wall/monster/target with crawl-left/back/right/back glances and resumed square movement. External demo.py/target-command.py bridge. Stopping simulation removes fixtures; all real assets saved, no map writes. Native tentacle motion retained but contacts/intersections still intentionally deferred. Looking chooses an authored directional clip once at onset; it does not continuously aim individual bones at a moving player. Changes saved/uncommitted; no final commit/push/package/version changes. User config/rifles/TestMap external actor changes preserved.

Final final-editor.json confirms dirty content/maps both0; desktop capture verifies visible preview on primary monitor. No later native/asset changes after passing verification.

## User-reported recoil diagnosis (2026-10-06)
Read-only evaluated33poses per clip, evidence recoil-assets.json. All14crawl/idle/look clips have root translation0 and EnableRootMotion=false. However crawl pelvis retains full travel: Up end-startY+84.82cm, DownY-85.20cm, LeftX+97.73cm, RightX-98.04cm. Resetting root alone did not make these locomotion clips in-place; pelvis resets on looping/switching, causing visual recoil despite actor travel being independently driven. This is a missed animation-adaptation defect in FEAT243. Ledge Left/Right Start->Hold->End endpoint pelvis AND neck gaps are0; their End deliberately returns pelvis~24.4cm to hanging reference, Back~10.7cm. Thus ordinary authored lean recovery and defective crawl travel reset coexist. Target demo was stopped by the time runtime observation was attempted (editor log21:32:03 endPIE); no new runtime result claimed. No production assets/code changed during diagnosis. Recommended correction: remove accumulated planar pelvis travel in the four crawl clips in TMIIR while retaining cyclic motion; recheck ledge-to-crawl pose offset/blend separately, migrate corrected finalized sequences only.
