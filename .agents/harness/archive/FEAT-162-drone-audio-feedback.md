# FEAT162 Drone audio and rejection feedback

Status: done. Closed2026-09-26. User selected drone laser survey (not Infiltrator), requested100835-Laser_scan_07-SFXBible-ss05993 intro+sustain; then249205-Game_Error_4 on rejection and whole invalid preview red, superseding FEAT161 local mask. Checkpoint8fc68b9. Evidence D:/UnrealWork/DroneAudio.

Implementation milestone: Development Editor build succeeds. Imported owner-local intro .34s, crossfaded .16s sustain loop and unchanged1s error source; Cue wrappers use fixed pitch/volume (recognition/loop policy exception), scan3D150+1850cm falloff/concurrency2 per owner, error2D/concurrency1. Consumer Blueprint defaults saved and cold validated. Native attack-to-loop .24s/.10s blend, completion .18s fade, abort owner-stop; error shares .18s shake cooldown. Full-preview red shader supersedes local intersection query; old graph nodes remain disconnected from compiled shader.

Editor DeleteAllMaterialExpressions asserted on rooted object; no material save occurred. Relaunched and safely edited the existing Custom node inputs/code; material compiled and saved, pixel instruction count526->217. Original deployment and preview structure preserved. Actual rendered whole-red preview inspected. Initial recordings were silent because editor was unfocused; test-only command-line Engine.Audio UnfocusedVolumeMultiplier=1 enables audio capture without changing project preferences. ActualDeploymentDebug.wav records5.65s of nonzero scan audio, startup/sustain/end silence; error recording nonzero. One initial end-of-fade assertion overlapped a HighResShot render stall; removed end screenshot and poll bounded completion rather than assume audio mixer advances while screenshot stalls. Final verification completed below.


## Completion evidence

- build-final.log: Development Editor Win64 Succeeded, no new code warnings. Final change after original build was whitespace only.
- final-cold.json independently reloads all3 Cue consumers/leaf waves, loop flags and stable modulation, attenuation/concurrency; editor compiles Executive and DroneTurret Blueprints. Material active validity inputs are exactly Original/Enabled.
- runtime.json33 checks pass; abort.json26 checks pass. Actual E/LMB rejects with selected error SoundCue+shake, restores blue after obstruction removed, nearest-enemy facing/confirmed spawn/half scale preserved, scan attack and loop active at correct stages, no per-frame voice stacking, fade and owner-unpossess cleanup verified.
- recordings.json: RejectedClick.wav peak.707916; ActualDeployment.wav5.312s peak.05905 RMS.00998 and silent last.1s; AbortedDeployment.wav peak.04303 and silent last.15s. Test-only command-line unfocused audio override removed by final normal relaunch; user settings untouched.
- WholeRedPreview.png manually inspected: entire ghost red with original line/fresnel/opacity character retained. Valid BluePreview.png captured.
- Optional cleanup of disconnected old graph nodes hit the same rooted-object engine delete assertion; no save occurred. Retained inactive nodes safely. This does not affect compiled217-instruction shader or full-red behavior. Final normal editor cold-load and PIE pass.
- No persistent fixtures/map saves. Final editor PID32316 on verified secondary monitor with Executive TestMap and deployable drone. ready.json/window.json. Original WAV downloads unchanged.
- Formal consumer validator: Scripts/Audio/validate_drone_scan_audio.py. Audio-policy/character/GAS docs updated. Changes uncommitted after8fc68b9; no push/package/version increment.
