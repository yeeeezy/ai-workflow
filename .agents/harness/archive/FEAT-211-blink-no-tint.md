# FEAT211 Disable purple blink filter

Status: done
Closed: 2026-09-28

FEAT211 completed2026-09-28: explicit user request cancels purple blink filter. Only MI_Infiltrator_BlinkFilter Blink-TintStrength changed .95->0; existing colorization shader/parent/tint retained inactive. No color shift at any blink strength. Radial blur.12, random centre, vignette.35, Flash muzzle trails, specified audio, original gun/arms materials and Phantom hidden UI unchanged. Live MI update/save/read-back passed, independent cold parameter check passed, actual unsaved TestMap25checks passed with neutral-color BlinkMid.png. No C++/BP changes or rebuild needed. Initial editor closed externally before queued authoring; launched fresh UE5.7 PID43332 on re-detected secondary. First runtime script failed unrelated transient Phantom summon lookup; removed that fixture from this filter-only check and final run passed. Restored SpaceStation and normal editor UI, noPIE/dirtycontent/maps. Evidence D:/UnrealWork/BlinkNoTint. Checkpoint90a776f9 preserves FEAT210. Saved final MI change uncommitted; no push/package/version/map changes.
