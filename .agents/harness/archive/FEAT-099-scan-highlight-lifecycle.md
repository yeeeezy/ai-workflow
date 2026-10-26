# FEAT-099 Scan highlight lifecycle

Status: done. User requested detected objects remain lit until scan cancelled, no highlight duration, and a scannable cube near PlayerStart.

- User-edited current SpaceStation preserved as external backup D:/UnrealWork/ScanHighlight/SpaceStation-before.umap, excluded from initial checkpoint. Ordinary Highlight interface retains timed compatibility; Scan removes its HighlightDuration setting, requests indefinite highlight and stores actual interface recipients to stop on retract/restart/EndPlay.

- Native build succeeded. BP_ScanCube (InteractableBase parent, engine Cube/BasicShapeMaterial, queryable collision) authored and compiled at /Game/Actors/ScanCube/Blueprint. Placed Station_ScanCube forward450/right180cm relative to current PlayerStart, ground-probed to rest on terrain; all1810 prior actor transforms preserved. Current user PlayerStart(-226.24,-426.91,149.08) retained. Added existing Mat_Outline_library/M_Highlight to station PP blendables, preserving existing grading/settings. Saved map and cold loaded in PIE.
- First rendered PIE passed real BGA_InfiltratorScan activation, initially unlit cube, highlight on wave arrival, retention beyond12s and immediate second-toggle off. On/off screenshots inspected: restored red scan and cube outline visible on, neither remains off. External cleanup test call omitted required Python destroy_component caller argument; corrected test only, final rerun pending.

- Closed2026-09-17: final rendered PIE ok=true, including restart and component EndPlay extinguishing targets. Evidence D:/UnrealWork/ScanHighlight/{build.log,author.json,pie.json,ScanOn00001.png,ScanOff00001.png}. Existing obsolete MPC reference/body-material startup warnings remain outside this lifecycle change; original red scan visibly restored. No final commit/push.
