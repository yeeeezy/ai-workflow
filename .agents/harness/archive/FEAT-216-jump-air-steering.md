# FEAT-216 Jump air steering

Status: done (closed2026-09-28)
User approved shared airborne WASD steering with inertia and unchanged vertical jump. Checkpoint8341fda1 preserves FEAT215. Inspecting movement defaults and live behavior; evidence D:/UnrealWork/AirSteering.

## Implementation and checks
Three playable Blueprint CharacterMovement defaults: AirControl .05 -> .65; AirControlBoostMultiplier2 ->1 for consistent acceleration from rest. Preserve jump420cm/s, gravity1, falling friction/braking0, each role max acceleration and ground speed. No native source or animation edits. All three BPs opened, compiled and saved live. Independent commandlet cold load confirmed exact saved values and all other inspected movement settings unchanged. Actual PIE movement-input trials:24cases and123checks pass across three roles, four directions, coasting, reverse braking and old/new steering comparison. Enhanced Input end-to-end check now running.

Intermediate test issues: editor simulate creates spectator instead of possessed gameplay pawn; switched to normal PIE. AssetEditorSubsystem uses close_all_editors_for_asset. Controller switching uses existing DevPanel handlers. EditorAssetLibrary.load_asset returned None for IA_Move during PIE, so test now uses the actual controller MoveAction reference. No production workarounds.

## Final validation
Enhanced Input using controller-owned IA_Move:24cases/123checks passed. At0.4s from550cm/s forward speed, right input gives roughly394cm/s lateral velocity versus38cm/s with old settings. Opposite input slows forward speed to~52cm/s, no-input preserves550cm/s. All roles land, root motion inactive throughout. No vertical/ground defaults changed. No C++ changes or build needed; live BP compile and commandlet cold load succeeded. Only three gameplay BP assets changed; no map saves, animation/retarget, config, package/version or push changes. Final changes remain uncommitted. Subjective player feel still for user review.
