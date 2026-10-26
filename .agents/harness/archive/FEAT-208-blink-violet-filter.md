# FEAT208 Blink violet filter

Status: done
Closed: 2026-09-27

User requests purple filter added to current blink. Add editable violet colour balance with luminance normalization, no extra desaturation or exposure change; native strength owns blending/cleanup. Preserve Flash four muzzle trails, phase gun/arms, radial blur/random centre/vignette. Checkpointe3d92e15.

## Completion

FEAT208 completed2026-09-27: added blink-only violet colour balance to existing filter after radial sampling and before vignette. New Blink-Tint=(1.05,.60,1.40,1) and Blink-TintStrength=.7. Multiplies scene RGB by interpolated tint and normalizes back to pre-tint Rec709 luminance (epsilon guarded); no independent desaturation, exposure change, highlight selection or additional texture samples. Display gamut/clipping may still affect perceived brightness. Existing16tap radial.12/random stable centre, vignette.35, native Strength/tail and four Flash muzzle trails/phase gun+arms preserved. Only two M/MI packages changed;132other Infiltrator gameplay assets byte-identical including BP/Flash/Phase. No native/BP changes or build required. Live material compile/save and actual unsaved TestMap22checks pass including active tint, radial blur, phase slots/restoration, muzzle offsets,600cm,neutral after tail and trail cleanup. Final screenshot visibly violet. Independent cold exactHLSL/sixscalars/vector/location validation passes. At start target editor had been closed externally; launched UE5.7 on detected secondary, PID32864 with existing BlinkViolet Python bridge. Restored SpaceStation and normal editor UI after immersive preview; no PIE, dirtycontent/maps0. Checkpointe3d92e15; evidence D:/UnrealWork/BlinkPurpleFilter. Final saved/uncommitted; no push,packaging,version or map changes. Subjective tint strength pending user feedback.
