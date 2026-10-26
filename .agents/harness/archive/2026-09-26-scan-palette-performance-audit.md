# Scan palette and moving performance audit

2026-09-26. Read-only source/screenshot diagnosis; implementation pending proposal confirmation.

Original07 MI export and pre-FEAT165 screenshot show grey-blue background, pale violet/pink surfaces, cyan boost and warm fine accents. FEAT165 bounded-luminance base and purple-family parameters produce stronger uniform purple. Restore original palette/art direction while retaining return animation, enemy black-blue palette, range checks and corrected air contours; compare exact original base mixing against a softer stabilised variant before choosing.

Smoke source: each front-facing enemy runs SceneCapture at30Hz and20 material-to-RT draws per step, including16 pressure iterations. Eight RGBA16f RTs at256. Captures use SceneColorHDR though current dye reads silhouette alpha only. Only five rendering show flags explicitly disabled; do not assert specific lighting costs until profiling. Per-frame mesh enumeration/show-only rebuild; forward hemisphere test only, no full-frustum culling. Legacy ScanEffect performs static/dynamic/pawn SphereOverlapActors every scan-active frame. These are code-derived workloads, not measured timings or proof of moving-specific cause.

Proposed measurement: same camera path/resolution/enemy count with scan off, full scan static/moving, grid-only, smoke-only; collect game/render/GPU timings and peaks. Optimisation candidates: cache mesh membership, eliminate unused capture shading after silhouette parity check, frustum bounds including smoke expansion (retain through-wall visibility), stagger enemy update phases, lower pressure iterations only with visual comparison, throttle legacy broadphase while preserving opted-in highlight behaviour. Do not blindly lower global rendering quality or promise FPS gains.

User editor PID23336 running normal project; UE58Blank PID31468 also remains from prior work. Python remote discovery returned no nodes. Did not open a competing rendered editor or alter the user session. Actual runtime profile remains outstanding, explicitly report this limitation. Production files/assets unchanged. Handoff docs only; checkpoint4f3025b.
