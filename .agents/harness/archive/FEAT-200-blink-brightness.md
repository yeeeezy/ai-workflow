# FEAT-200 Blink brightness

Status: done; closed2026-09-27

User finds FEAT199 too dark and explicitly requests unchanged scene brightness. Neutralize brightness/contrast/vignette only, retain radial blur, selective violet accents and desaturation. No native/BP/timing change. Evidence D:/UnrealWork/BlinkBrightness.

## Completion

FEAT200 completed2026-09-27: user requested unchanged scene brightness after FEAT199 too-dark feedback. Existing Blink M/MI Brightness1, Contrast1, Vignette0 replace.2/1.6/.4, removing global darkening and added vignette. Existing blur.24, saturation.08 and selective violet accents retained; shader graph/code unchanged.600cm/.26s/.28tail/native ownership and scan unchanged. Live material compile/save, actual TestMap18 GAS/cleanup/scan coexistence/pause checks and cold24parameter/reference/shader checks passed. BlinkMid.png visually confirms restored base brightness. Only2material packages changed, no C++/BP/map writes or build required. PIE ended, dirtycontent/maps0. Checkpoint ec268e7f preserves dark version. Evidence D:/UnrealWork/BlinkBrightness. Final saved/uncommitted; no push/package/version change.
