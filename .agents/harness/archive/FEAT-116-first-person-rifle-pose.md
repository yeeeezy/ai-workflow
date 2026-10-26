# FEAT-116 First person rifle animation pose revision

Status: in_progress
User explicitly authorized parallel implementation. Existing follow-up checkpoint660450e applies; root owns harness/build/target integration.

- Match desktop WeChat reference right-low side-visible gun pose via arm animation
- Adjust excessive gun size if needed; keep central view clear and hands attached
- Retarget only external TMIIR/FPSShooter1; target finalized assets only
- Verify firing reload movement ADS and transition poses

User clarification: every adjusted Sequence/BlendSpace/AnimBP must be independent owner assets, no shared action mutation. Executive keeps its owner folder. Infiltrator gets /Game/Weapons/InfiltratorLaserRifle with BP child of base BP_LaserRifle; material and changed animation refs independent. Worker original LaserRifle unaffected. Public interface/native GAS/ADS behavior may remain inherited.
