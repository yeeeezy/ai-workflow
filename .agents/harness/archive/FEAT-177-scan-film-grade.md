# FEAT177 Scan film grade

Status: done. Closed2026-09-26. User requested Control reference screenshot210416: soft vignette,faded/nostalgic texture; restore missing original TMIIR orange lines shown in210607/210559. Explain flattening from previous uniform-violet palette and source stable-luminance adaptation. Restore warm/cool contrast without reverting unstable scene-colour extrapolation; add lightweight custom output grade to existing scan material, exposed MI parameters. No separate full-screen pass,capture,smoke or audio changes. Smoke stays disabled. Evidence D:/UnrealWork/ScanFilm.

## Preview ready, waiting for user editor close

2026-09-26: new graph compiled and actual SpaceStation preview lifecycle passed16checks; EnemySmoke remainsfalse. Preview images Startup/Sustain/Ground/Return and preview-audit.json. Scan-only film scalar controls: Strength1,Saturation.72,Vignette.42,Fade.015,Grain.008; no new pass/capture. Original warm outline/dots and gold grid restored; grey-violet base.

Asset save FAILED with Windows Error32 sharing violation. User-owned TheManTest PID38208 locks M_Infiltrator_ScanGrid; never force-close. Asked user asynchronously to save/close it; no reply yet. Owned validation editor PID46296 remains open with3dirty packages M/MI/BP_Infiltrator, no dirty map. Changes currently preview-only, NOT completed or cold-verified. Do not quit owned editor before saving after lock release. UE58Blank PID31468 unrelated. Queue D:/UnrealWork/ScanFilm/command.py with resume-save.py after user close; current globals mat,mi,bp,colors,params and queue alive. Source scripts are reproducible. Checkpoint8211a34.

## Final correction and completion

User explicitly rejected scene-complexity diagnosis because same test scene also lacks source texture. Do not attribute difference solely to scene. Inspection confirmed FEAT165 replaced source base with bounded monochrome luminance,changing chroma/contrast. A/B uncompressed luminance alone only modestly changed appearance. Final restores source Desaturation0 -> Multiply1.A,BaseDesaturation3 and source basecolour,removes old custom compression. Hue control is separate film output,not further base rewiring. Source normal/Fresnel functions remain. This supersedes initial plan to retain bounded base.

All assets saved after lock released; fresh editor cold validates original base link,absence of compressed-luminance node,source Base-Desaturation3,film scalar/vector controls,restored warm grid and disabled smoke. SpaceStation16 lifecycle checks pass; startup/sustain/return PNGs inspected. Material and BP compile successfully; no C++ changes or build required. No dirty map/content packages; owned editor closed. Actual texture comparison in same SpaceStation camera: DetailA_Compressed,DetailB_LinearLuminance,DetailC_LinearWithFilm; removing compression alone was modest,not proven sole cause. Final source base processing restored; artistic equivalence remains subjective. Evidence D:/UnrealWork/ScanFilm; checkpoint8211a34; final changes uncommitted,no push/package/version change.

Final author manifest:
```json
{
  "ok": true,
  "colors": {
    "Base-Color RGBA": [
      1,
      0.226637,
      0.286385,
      1
    ],
    "Background-Color RGB": [
      0.025,
      0.021,
      0.034,
      0.5
    ],
    "Outline-Color RGBA": [
      1,
      0.385808,
      0,
      0.2
    ],
    "Dots-Color RGBA": [
      1,
      0.75,
      0,
      0.2
    ]
  },
  "film": {
    "Film-Strength": 1.0,
    "Film-Saturation": 0.25,
    "Film-Vignette": 0.42,
    "Film-Fade": 0.008,
    "Film-Grain": 0.008
  },
  "film_tint": [
    1.12,
    0.91,
    1.27,
    1
  ],
  "base_desaturation": 3,
  "base_connection": "MaterialExpressionDesaturation_0 -> MaterialExpressionMultiply_1.A",
  "stats": "<Struct 'MaterialStatistics' (0x0000015D905110E0) {num_vertex_shader_instructions: 153, num_pixel_shader_instructions: 114, num_samplers: 3, num_vertex_texture_samples: 3, num_pixel_texture_samples: 18, num_virtual_texture_samples: 0, num_uv_scalars: 2, num_interpolator_scalars: 0}>"
}
```

File-lock resolution: user confirmed saved/closed; owned preview had exited before pending queue save. Remaining user project window received normal WM_CLOSE (no forced termination) and exited. Recreated authoring from external script,successfully saved then cold-verified. Earlier blocked/preview-only statements are historical. No file lock remains. No original smoke asset deleted.


## Follow-up visual acceptance

2026-09-26 TestMap visual acceptance FAILED: actual GAS PIE screenshots show extremely faint/thin yellow grid lines, while prominent floor squares are the pre-existing floor material. Current scan raises shadow appearance and looks washed out. Same-camera Film-Strength=0 transient comparison restores saturation but grid remains faint; film grade contributes to grey appearance, not sole cause. No production asset/code/map changes. Two PIE passes successful, no smoke, clean shutdown; dirty content/maps zero. Evidence D:/UnrealWork/ScanTestMapReview/{Before,Ground,FilmOn,FilmOff}.png. Earlier parameter/lifecycle pass does not establish visual acceptance.
