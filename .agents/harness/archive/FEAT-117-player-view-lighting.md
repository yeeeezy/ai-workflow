# FEAT-117 Player view lighting

Status: done
Closed: 2026-09-17
User asked to enable remembered reserved player light, specifically gun and forward scene visibility. Read-only Blueprint templates and spawned actor audit found zero light components on all3 playablepawns; add native camera-child WeaponFillLight35lm160cm and ForwardLight900lm1400cm28/58degrees. PercomponentBlueprint tuning, no map/postprocess edits. WIPcheckpoint d72e442. Build/render pending.

DevelopmentEditor/Win64 build.log Succeeded, no new warnings. Initial spot overexposed the gun; added shared M_PlayerForwardLightFalloff Light Function on all3 playable BPs, fading from125cm to225cm camera distance. Native35lm near fill remains. Actual SpaceStation before/after/fill-only/turn capture passed: D:/UnrealWork/PlayerLight/visual.json ok=true; visual-final.log and After-On.png inspected. Blueprint compile/save and component camera-follow assertions passed. Implementation and validation complete; final batch handoff pending.

Implementation and required validation complete. User visual acceptance remains pending.

User follow-up: all ForwardLight defaults now450lm. Native FPSCharacterBase constructor changed900->450; all3 playable Blueprint components compiled/saved/read back450. Evidence D:/UnrealWork/PlayerLight/default450.json ok=true and default450-build.log Succeeded12.20s. Selective checkpoint1dae179 preserves user saved Infiltrator/Executive BP changes before tuning. No repackage requested; latest delivered executable retains its prior lighting values.
