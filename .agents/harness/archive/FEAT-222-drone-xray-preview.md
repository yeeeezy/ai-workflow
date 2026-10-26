# FEAT222 Drone deployment XRay preview

Status: done (2026-09-29)

User explicitly approved changing the deployment preview to current enemy scan transparent style, keeping valid blue and invalid red.

## Implementation and verification

FEAT222 completed2026-09-29: BP_TheExecutive.DroneDeployment.PreviewHologramMaterial now selects owner-local M_TheExecutive_DeploymentXRay, AssetTools duplicate of current enemy M_Infiltrator_ScanGoldXRay. Original Fresnel exponent3.5, visible-rim/occluded-fill shader, bias2cm/feather3cm, opacity.8/glow3 retained. Inner/outer RGB normalized to gray peaks.32/1.25, alpha.06/.8 preserved, then tinted by existing HologramColor(.008,.12,1) / InvalidColor(1,.008,.016) with ConflictEnabled. All slots share the existing runtime MID; whole valid blue/invalid red policy preserved. No production C++ change. Preview has no custom-depth writer, so enemy-specific stencil/internal-depth cull bypassed for output; copied unused cull nodes remain disconnected. Old placement material retained as rollback/native fallback; enemy assets unchanged. Live material compile/save and Blueprint open/compile/save passed; actual TestMap EnhancedInput42checks plus supplemental blue/red screenshots passed. Independent cold62checks passed, exact enemy occlusion HLSL verified, every active expression local and no custom-depth sample in output chain.240 protected packages/config byte-identical including all scanned enemy/Drone assets, user rifles and version. Only existing production package changed is Executive BP plus one new material. Final editor30936 on freshly detected primary (no secondary), SpaceStation,noPIE/dirtymaps/content. Checkpoint5ceb7539 preserves priorFEAT221. Evidence D:/UnrealWork/DroneXRay including BlueRecovered.png and WholeRedPreview.png. No map/animation/retarget/package/version edits, final commit or push. Subjective visual acceptance pending.

## Intermediate issues

Three attempts to delete the existing placement material expressions triggered the MaterialEditor !IsRooted assertion before any asset save; reducing Python-held references and snapshot iteration did not resolve it. No proven root cause beyond expression deletion. Original target material remained byte-identical. Final workflow avoids deletion: AssetTools duplicates the source material, reconnects output and sets an explicit BP material override. Final live/cold logs contain no shader errors. Original scan and old preview materials were never saved.

The first BluePreview.png was captured after the immediately queued obstruction, so it shows red; do not use it as blue evidence. Supplemental visual run delays scene changes and confirms BlueRecovered.png correctly blue. Main42checks already passed real valid/red/back-to-blue parameters, no-hit hiding, E release, reentry, rejected LMB audio/shake, successful confirmation, transform match, half-scale, construction audio completion and original final turret materials.
