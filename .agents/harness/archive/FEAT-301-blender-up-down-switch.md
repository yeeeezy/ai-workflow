# FEAT-301 Blender up/down toggle switch

Status: review pending. Created 2026-10-20.

## Request and scope

User requested an arbitrary up/down switch modeled in Blender. This is a standalone external Blender project, with no Unreal import or project Content edit.

## Deliverables

- `D:/Blender Projects/UpDownSwitch/UpDownSwitch.blend`: editable plate, mounting screws, labels, lamps and lever parts parented to `Lever_Pivot__rotate_X_0_up_86_down`.
- `D:/Blender Projects/UpDownSwitch/UpDownSwitch.glb`: portable export with 25 meshes and one lever animation. Source animation frames 1–40 cover UP, DOWN and return to UP; position lamp emission is keyed in the Blender source.
- `UpDownSwitch_Preview.png` and `UpDownSwitch_Down.png`: 960x1120 rendered views of both positions. Both were inspected; the UP preview was opened in Windows Photos on the only connected primary monitor.
- `build_switch.py` and `build.log` in the same external project directory reproduce the model and renders.

## Verification and handoff

Blender 5.0 background build exited 0; GLB JSON validation found 25 meshes, 26 nodes and one animation. `.blend`, `.glb` and both PNG files exist. No UE project asset, Blueprint or Content path was changed. Visual review remains with the owner. WIP Git checkpoint before harness edits: `refs/wip/before-blender-up-down-switch` (`f9fc6104cf139002a4721347efae6b2c819c3951`).
