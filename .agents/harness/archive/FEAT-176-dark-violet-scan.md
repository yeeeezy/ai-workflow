# FEAT176 Dark violet scan

Status: done. Closed2026-09-26. User requests palette matching latest screenshot2026-09-26 205008: near-black violet atmosphere, restrained purple and violet-white highlights. Followup explicitly disables enemy smoke for now; keep assets/palette configuration for restoration. No new VFX systems, enemy highlight or sound changes. Native smoke enable switch defaults false; expose Niagara dot/line colours. Material instance palette only. Verify in SpaceStation. Evidence D:/UnrealWork/ScanDarkViolet.

## Implementation and verification

Editor Win64 Development build succeeded14.70s. BP_Infiltrator compiled/saved. Parent/MI9palette values cold-read exact;16 cold SpaceStation checks passed: GAS start/sustain/reverse,editable Niagara palette,original audio and cleanup,zero smoke actors/targets,zero dirty maps/content. Startup/Sustain/Ground/Return PNGs visually inspected. MI-only changes left blue during startup; aligning parent vector defaults corrected partial postprocess blending. No map/exposure/audio changes or smoke-asset deletion. Evidence D:/UnrealWork/ScanDarkViolet. Checkpoints c4d53df,2fec3f2;final changes uncommitted;no push/package/version increment.

Palette linear RGBA:
```json
{
  "Base-Color RGBA": [
    0.18,
    0.12,
    0.29,
    1
  ],
  "Background-Color RGB": [
    0.01,
    0.007,
    0.018,
    0.5
  ],
  "Flash-Color RGBA": [
    0.4,
    0.25,
    0.65,
    0.08
  ],
  "Outline-Color RGBA": [
    0.36,
    0.24,
    0.58,
    0.14
  ],
  "Outline-ColorBoost RGBA": [
    0.64,
    0.43,
    1,
    4
  ],
  "Fresnel-Color RGBA": [
    0.1,
    0.055,
    0.18,
    0.15
  ],
  "Fresnel-Color Boost RGBA": [
    0.7,
    0.5,
    1,
    4
  ],
  "Dots-Color RGBA": [
    0.28,
    0.18,
    0.45,
    0.12
  ],
  "Dots-Color Boost RGBA": [
    0.65,
    0.44,
    1,
    3.5
  ]
}
```

GridDotsColor=(.45,.34,.65,.06);GridLinesColor=(.25,.16,.40,.10),both axes;Z transparent. EnableEnemySmoke=false skips discovery/spawn and clears prior owned smoke. No new enemy highlights. UE vector setter returned false although assignment readback succeeded; actual saved/cold/render values verified. Source screenshot2026-09-26 205008.
