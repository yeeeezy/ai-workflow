# SpaceStation entry hitch diagnosis — 2026-09-26

Scope: user authorized further investigation only. No production code, assets, map or rendering configuration changed. Two rendered direct-PIE captures in existing UE5.7 editor PID18316; loaded SpaceStation without saving. SelectedCharacterID empty -> BP_TheExecutive. Zero InfiltratorScanSmoke instances in both runs. UE58Blank editor remained open, so these are local editor measurements, not isolated packaged performance.

## Confirmed measurements

- StartPlayInEditorSession: 891.42 / 836.04 ms. World duplication: 333.30 / 327.23 ms. Audio device init: 95.31 / 94.26 ms. These timings are within startup, not additive extra stalls.
- UWorld::BlockTillLevelStreamingCompleted: 223.10 / 220.05 ms. Nested FlushAsyncLoading: 138.08 / 137.92 ms; registration/update streaming contributes roughly another 81 ms in run1.
- The log's 112 QueuedPackages are requests, not 112 unique assets: 69 AddPackage plus43 UpdatePackage. Unique assets:46 LandscapeStreamingProxy,22 InstancedFoliageActor external packages and1 /Engine/EngineMaterials/LineSetComponentMaterial. All43 duplicate requests concern that same engine material. Asset Registry names/classes cross-checked; no scan assets in this flush. SplineComponent.cpp initializes both line and point materials to this path.
- First game-thread frame after startup:145.31 /141.40 ms. First run next frames42.2,54.8,125.9,132.6 ms; second run43.4,27.1,25.3 ms, then roughly15-17 ms. Steady game-thread median16.64ms with60FPS cap. No claim of uncapped headroom.
- Run2 first-frame GameThreadWaitForTask100.08ms aligns with rendering startup. Render visibility wait39.89ms; worker FPrimitiveSceneInfo_CacheMeshDrawCommands55.78ms includes DeferredFinalizeMeshDrawCommands38.78ms. Initial landscape TickEdgeFixup27.11ms includes HeightmapsNeedingEdgeSnapshotCapture26.80ms. These scopes overlap/nest and must not be summed.
- GPU frame intervals that span startup contain idle/submission gaps; do NOT interpret ~715-730ms GPU Frame as shader execution cost. RHI waits similarly are not active work. No evidence that VSM overflow is the dominant entry stall; existing overlap warning remains a separate rendering concern.

## Interpretation and next work

The repeated entry hitch has CPU-side editor world duplication, synchronous nearby landscape/foliage streaming, then scene/mesh draw-command and landscape initialization before smooth frames. This is reproducible with Executive and no scan simulation. Scan hard-reference/cold-load cost outside this measured flush is not universally excluded.

Any implementation should distinguish direct PIE startup from actual lobby-to-game/packaged entry. PIE duplication is editor-only. Profile actual travel separately before changing game loading policy; investigate scene warmup/streaming scope and mesh draw-command work before reducing lighting quality. Preserve prior FEAT144 terrain under station/NPC safety guarantees. No optimization applied in this investigation.

## Evidence / state

D:/UnrealWork/StationEntryAudit: entry1.utrace,entry2.utrace; events1.csv,events2.csv,render1.csv,firstframes-all.csv; runtime.json; packages.json/package-classes.json; analysis.json; final-state.json. Original detailed log D:/UnrealWork/InfiltratorGridScan/editor.log. Insights exports run headlessly with NoUI; local UE engine source consulted. Editor window verified on secondary monitor [2572,12,4468,1042]. PIE ended; SpaceStation open, no dirty map/content packages. Prior temporary scan-review actors disappeared with PIE teardown. Other editor unchanged.

Documentation-only checkpoint50a20ca protects prior FEAT163 progress/index. Final diagnostic documentation uncommitted; no push/build/package necessary or performed.

## Regression follow-up after user chronology correction

The earlier measurements identify existing costs, NOT the cause of a new regression. Historical logs prove direct SpaceStation PIE startup was already0.891s at2026-09-26 15:48UTC (before grid-smoke),and0.862s at2026-09-23 07:15UTC. Old first SBT creation also occurs about228-235ms after PIE total-start logging,similar to the recent log. This does not disprove the user's newly perceived hitch, but invalidates attributing its onset to these unchanged baseline costs.

Confirmed new cross-character impacts: Config/DefaultEngine.ini gained r.CustomDepth=3 (engine default was1). Asset Registry confirms hard-reference chain BP_TheManGamemodeBase -> DT_CharacterRoster -> BP_Infiltrator -> nine direct new grid/smoke assets and their dependencies. Executive and zero smoke simulation therefore do NOT rule out new resource-loading cost.

Four reversible in-session A/B runs,mode1/3/1/3,4s settle after each mode change,4sPIE: logged startup0.880/0.867/0.839/0.829s. Post-start Slate callback long interval113.1/109.9/115.7/112.6ms. No reproducible degradation from mode3 in this warm test. This is not a clean cold before/after asset/build comparison. Original mode3 restored,no saved config change,PIE ended,no dirty packages; ab-final-state.json.

Environment confound: during user's21:01-21:03UTC separate TheManTest PID36648 test,our PID18316 remained in TestMap with active two-enemy smoke preview,plusUE58Blank PID31468.RenderOffscreen source editor. These sharedGPU workloads could affect user-visible hitches; not measured enough to assert causality. Our scan preview stopped during diagnosis; no duplicate TheManTest currently. Source editor untouched.

Optional user clarification pending: hitch at first visible character withoutscan,firstE scan,or lobby START travel. Next causal work should match that exact moment and compare identical cold/warm and concurrent-editor conditions. Do not present startup baseline or light warnings as established new-regression root cause. No implementation authorized/applied in this followup; documentation checkpoint0289e87.
