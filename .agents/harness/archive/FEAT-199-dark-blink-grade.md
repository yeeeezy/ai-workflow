# FEAT-199 Dark blink grade

Status: done; closed2026-09-27

User rejected FEAT198 bright grade and authorized darker reference match. Apply material-only contrast/brightness curve and radial-sample contrast-gated violet accents, preserving16tap blur and native lifecycle. Safety checkpoint a6fe2262. Evidence D:/UnrealWork/BlinkDark.

## Completion

FEAT199 completed2026-09-27: user rejected FEAT198 bright grade; Blink filter now dark near-black/grey with selective violet contrast accents. Existing M/MI only: Brightness.2, Contrast1.6, Vignette.4, Saturation.08, HighlightThreshold.65; existing16radial samples also track luma range to gate purple accents, avoiding flat bright wall/sky tint, no extra texture fetches. Blur.24 and purple color retained;600cm/.26s/.28tail/native lifetime and all scan assets unchanged. Live material final recompile/save succeeded; intermediate missing-input warning during graph editing resolved and final saved graph inputs connected. Actual TestMap18 real GAS/visual/lifecycle/coexistence/pause checks pass; BlinkMid.png visually much darker than FEAT198, centre scene detail retained with dim purple streaks. Cold24checks confirm graph/params/BP references and duration. Exactly2material packages changed;52other existing ability/BP packages SHA256 preserved. PIE ended, dirtycontent/maps0. No C++/BP/map edits, build required, push/package/version change or performance claim. Checkpoint a6fe2262; evidence D:/UnrealWork/BlinkDark. Final saved/uncommitted; subjective acceptance pending.
