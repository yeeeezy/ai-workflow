# FEAT-238 Medusa introduction and Dark Veil

Status: done. Created/closed 2026-09-30.

Latest user correction supersedes moving camera: use original LevelStart with black fades into introduction and back to combat, no push-in/zoom/low orbit. Fixed shot retains player camera position/FOV and turns toward Medusa only under full black. First activation once per game retained. ActivationRadius explicitly800cm (8m), supersedes2500 and proposed1250. Checkpoint3f3b532b preserves prior implementation. Revised fade timings0.4s, original animation5s, combat resumes after return fade clears; runtime/cold verification pending.

User approved first activation, once per game. Push in from player camera, low orbit during original Sevarog introduction, return control before combat. Darkness ability renamed Dark Veil; Nightmare remains the enemy family. Shared activation FEAT234 remains deferred.

Checkpoint 8c5bdc0c preserves FEAT237. Original LevelStart is 5 seconds, nonadditive, no root motion. Copy native sequence using existing identical-skeleton validated helper; no retargeting. Evidence: D:/UnrealWork/MedusaIntro.

Implementation: world-owned MedusaIntroSubsystem gates existing Medusa decision service on first acquired target. Real-time tick advances the original sequence through DefaultSlot while world pause protects player/round timers. Camera push-in0.8s, front low orbit85degrees, return0.7s. Native configurable distance/height/focus/FOV and intro enable/sequence on BP_Medusa. Preserve/restore input ignore stack, actor hidden state and viewport widget visibility. Death/EndPlay/unpossess/view replacement cancels. No shared activation framework changes.

Dark Veil: native GA and scene subsystem renamed, Medusa properties/functions renamed with exact CoreRedirects. AssetTools renamed BGA_Medusa_Nightmare to BGA_Medusa_DarkVeil and updated phase Background references. Duration8/cooldown12/max25/activation2500 retained. Intro asset AS_Medusa_Intro references existing Medusa native skeleton only.

Development Editor Win64 build2.log succeeded5.88s, no compiler warnings. Initial build discovered protected PlayerController full-tick flag; removed access, camera updated directly by owning subsystem while paused. BP_Medusa and BGA_Medusa_DarkVeil opened/compiled/saved. PIE verification ongoing. Initial external probes used unavailable Python subsystem access; switched to ObjectIterator without modifying gameplay APIs.


## Final verification

Final user-approved implementation is the fixed shot and black fades described in the correction above; earlier orbit implementation is superseded. Native/BP ActivationRadius800cm. Build-fade2.log succeeded4.95s, no compiler warnings. BP_Medusa compiled/saved; prior renamed BGA compiled/saved.454 actual PIE frames:850cm outside no intro;700cm inside starts intro; paused world advance0; camera position constant; view/input/visibility restored; new Medusa in same world does not replay. Original sequence reaches blend-out at4.63s. FixedIntro.png visually inspected: full character visible. Destroy-during-intro test clears pause/input/view/fade; InterruptedRestored.png shows normal first-person view. Renamed GAS Dark Veil activation and cancel/mesh restoration pass. Cold185assets: native class/background references correct, duration5s/radius800, no source-package dependencies or redirectors. Final editor31416 SpaceStation, noPIE, no dirty content/maps.

External test script issues corrected: native Python cannot access world subsystems through the unavailable library; ObjectIterator used. First repeat probe read a destroyed actor; corrected probe454frames passed. Early Dark Veil probe accessed BrainComponent before first AI tick; final isolated unpossessed-actor GAS test passed. These did not require gameplay changes.13-game-flow.md is legacy non-UTF8, new ASCII documentation appended preserving original bytes.

Architecture06/07/10/11/13 synchronized. No retargeting/maps/package/version/push changes; unrelated config/rifle assets preserved. Final changes uncommitted after checkpoint3f3b532b. Evidence D:/UnrealWork/MedusaIntro.
