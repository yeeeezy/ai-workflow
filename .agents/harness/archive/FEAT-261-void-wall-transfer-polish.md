# FEAT261 Void wall transfer polish

User authorized implementation after free-animation research. Retain native creature; polish wall-to-wall only. Checkpoint a29f3c5a preserves FEAT260; unrelated config/rifles/externalactor/caches excluded. Source animation preparation only TMIIR. Evidence D:/UnrealWork/VoidWallPolish.

Implementation: wall-only airborne turn holds first10% then finishes at reach entry (last min(.20s,28% flight)); bounded16deg upper-spine/neck lead. Wall Air samples authored1x looping time rather than stretching to path duration. Existing Land begins during approach, advances first1/30s before contact, then continues same asset/time through recovery. Ground variants untouched. No new assets or retargeting required for this timing pass.

Build initially exposed pre-existing unity collision between anonymous Enemy helpers after FEAT260 checkpoint changed adaptive unity membership. Renamed only WallAction helper to WallActionEnemy; Development Editor20.21s succeeded. Native WallLeap regression now also verifies pre-contact reach and continuous grab playback at attachment.

## Complete implementation ? 2026-10-07
Visual review found dangling human-fall legs; added smooth72%-peak blend toward existing native wall Idle frame0 in mid-flight, releasing before reach. Final Development Editor build17.75s Succeeded, no compiler warnings. Initial four native tests GroundCombat/TentacleGAS/TentacleGrab/WallLeap Success; final tuck revision WallLeap rerun Success. Existing cancellation/death/crawl continuation assertions retained. No Blueprint/asset edits.

Live normal-speed reciprocal transfers: initial4passes, final2passes; final clean recording samples show.30s Start,.93s Air,.97s recovery, accurate wall attachment. Real30fps desktop recording WallTransfer.mp4 and viewport crop WallTransfer-Viewport.mp4 inspected at launch/midair/reach/recovery. Midair knees visibly tucked. Earlier console Shot captured editor viewport instead of PIE; those pose*.png are NOT validation. MCP game_viewport frames can force extra drawing and are not used as timing evidence. Final clean recording avoids those requests during flight.

Preview setup corrections only: console-summoned walls inherited camera yaw; explicitly zeroed while movable. Immediate BrainComponent access delayed until next tick. No production spawn workaround. Removed inactive preview cameras. Handoff.json shows one live Void, running real BT, automatic crawl/transfer enabled, no dirty maps/content. Editor30556 on re-detected sole primary; transient TestMap two walls, camera and actor atZ20000, runtime cooldown4, health invincibility. Runtime-only r.MotionBlurQuality0 for inspection; engine config unchanged. Stop PIE clears fixture. Final code/docs uncommitted; no asset/map saves, retargeting, packaging, version bump or push.

Limits: heuristic reach timing for existing Grab clip, not authored per-limb contact events; no per-tentacle wall contact IK or new custom animation assets. Subjective naturalness still requires user review; delivered as first wall-transfer polish pass.
