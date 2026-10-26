# FEAT-197 Highlight occluder culling

Status: done; closed2026-09-27

User accepts FEAT196 appearance but reports highlighted objects hide enemy. Graph audit confirms source internal-triangle cull compares pixel depth with shared CustomDepth without stencil ownership. Object stencil1 in front causes zero enemy opacity. Fix will apply cull only when nearest custom-depth writer is enemy254. Checkpoint ae7570f6 protects accepted previous version. Evidence D:/UnrealWork/GoldHighlightOccluder.

## Completion

FEAT197 completed2026-09-27: fixed highlighted objects erasing enemy XRay. Original internal-surface cull incorrectly treated stencil1 object CustomDepth as enemy front surface. Existing M_Infiltrator_ScanGoldXRay now samples CustomStencil and supplies original depth only when stencil254; other stencil values supply PixelDepth to preserve enemy opacity. Scene-depth visible-rim/occluded-fill selection, palette/glow, native lifecycle and handmade object material unchanged. Actual same-fixture TestMap PIE before/after reproduced central enemy completely absent behind highlighted cube before, translucent gold enemy visible after while cube highlight remains. Both real GAS32checks pass incl original body materials, enemy254/object1, stop/restoration and exposure0/blur. Independent cold19checks plus saved graph export pass; live material compiled/saved with no shader errors. Sole changed production asset is XRay master;47other scan/BP packages SHA256 preserved. No C++/BP/map changes or build required, no push/package/version change. PIE ended, dirtycontent/maps0. Checkpoint ae7570f6; evidence D:/UnrealWork/GoldHighlightOccluder including before/GoldOcclusion.png and after/GoldOcclusion.png. Results saved/uncommitted.

Shared CustomDepth only stores the nearest writer. When a highlighted non-enemy owns that pixel, enemy inner-surface culling is deliberately bypassed; overlapping enemy surfaces can contribute additional translucent detail there. No extra depth buffer or render pass added; one CustomStencil sample.
