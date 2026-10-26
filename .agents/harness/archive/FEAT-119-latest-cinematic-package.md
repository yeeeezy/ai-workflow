# FEAT-119 Latest Cinematic Windows package

Status: done
Closed: 2026-09-17

User explicitly requests highest-quality package while asking about continued editor use. Existing Cinematic groups4/native100percent/no dynamic resolution retained; latest saved user Station map and Infiltrator rifle included. No code/config/assets changes for packaging. Normal editor browsing/PIE may continue; user informed not to save changes or compile source during cook. Existing113 patrol remains unresolved and is not part of packaging repair.

Initial UAT failed before build due active editor Live Coding. Using official -skipbuildeditor to reuse latest successful editor binaries while building separate Win64 DebugGame target; no disabling guard or closing user editor. New timestamp output preserves old package.

Output: D:/UnrealBuilds/TheManTest/DebugGame-Cinematic-20260917-140300
Evidence: D:/UnrealWork/CinematicPackage-20260917-140300

Package completed BUILD SUCCESSFUL/ExitCode0 in105.07s. Lobby/SpaceStation/Test offscreen20s startup runs each exited0 before user requested no more checks and manual inspection. No further verification. Existing M_UE4Man_Body fallback and SmartObject messages retained from prior package;113 unresolved. Saved source settings/gun/Station hashes unchanged across packaging. User editor untouched.

2026-09-17 followup: user explicitly requests another highest-quality debug package. Existing Cinematic4/native100/no dynamic resolution retained. BuildCookRun Win64 DebugGame build/skipbuildeditor/cook/stage/pak/iostore/archive succeeded ExitCode0. Output D:/UnrealBuilds/TheManTest/DebugGame-Cinematic-20260917-143018; log D:/UnrealWork/CinematicPackage-20260917-143018/package.log. Includes saved450lm and FEAT120 gun damage/ammo/Station actors. No runtime launch checks per prior user preference. User editor untouched.
