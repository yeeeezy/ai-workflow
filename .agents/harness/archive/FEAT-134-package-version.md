# FEAT-134 Date-based project/package version

Status: done. Closed: 2026-09-19.

## User request and delivery

User authorized YYYY.MM.DD.NN date-plus-sequence numbering, adding it to debug UI, automatic update on packaging, and harness rules. Initial ProjectVersion is 2026.09.19.01 in Config/DefaultGame.ini. WBP_DebugMenu has an editable muted VersionLabel below the controls. DebugMenuWidgetBase.NativeConstruct reads GeneralProjectSettings.ProjectVersion from GGameIni; displayed text is VERSION 2026.09.19.01. No layout construction at runtime and no new module dependency. Tab/Escape/God Mode behavior is unchanged.

Canonical packaging command: `python Scripts/Build/package.py`. Default Win64 DebugGame; --configuration supports Development and Shipping. --engine and --output-root override default UE5.7 and D:/UnrealBuilds/TheManTest roots. This is an actual wrapper around RunUAT BuildCookRun build/cook/stage/pak/iostore/archive, not merely a written reminder.

The wrapper validates project/engine, locks the packaging operation, increments local-calendar YYYY.MM.DD.NN before build/cook, starts a uniquely named delivery folder, and records BuildVersion.json with status and exit code. Same-day sequence increments; new date resets to01; clock rollback is rejected; failures retain reserved numbers. Ordinary Editor builds, saves and play sessions do not increment. Harness requires this wrapper on every future package; direct UE Package Project/raw UAT bypass it and must not be used. No native editor packaging hook is claimed. After future full packaging, verify the actual packaged UI against the manifest as required by the rule.

## Verification

Evidence D:/UnrealWork/PackageVersion:

- build.log: Development Editor Win64 succeeded.
- script-tests.json: nine isolated checks including config update before UAT call, date rollover, success/failure manifests, launcher exception, collision prevention, clock rollback, concurrent lock and unrelated config preservation. UAT runner simulated; project version not incremented by tests.
- process-test.json: actual Windows subprocess runs a fake .bat with space-containing paths and propagates intentional exit13; no real full package.
- runtime.json: real PIE reads VERSION 2026.09.19.01, opens/reopens/closes correctly and restores pause.
- UI00001.png: visually inspected clean runtime version display on secondary monitor. UI00000.png has transient existing startup messages and is superseded.
- cold.json / cold.log: independent commandlet loads and recompiles the saved widget successfully.
- editor-final.json: affected widget/controller compiled/saved; PIE ended, no dirty packages/maps. secondary.json confirms editor placement.

Authoring initially used an unavailable Python TextBlock.get_font API; corrected to get_editor_property. Initial compilation repaired the newly added widget's missing GUID through the engine compiler fixup. Subsequent editor compilation/save and independent cold compilation passed without that ensure. Final saved asset is verified.

## Handoff

Checkpoint af720d5 protects preceding Debug UI work; unrelated weapon/map edits excluded. This batch remains uncommitted, no push. No full game package was produced or claimed. Version is still 2026.09.19.01; next successful invocation on the same day reserves .02 (failed attempts also consume their number). Packaging rule is in harness/AGENTS.md and implementation details in arch/01-build-config.md; UI ownership in arch/02-core-framework.md. FEAT-131 remains pending broader appearance acceptance. Visible operations remain on secondary monitor, with no recurring placement lock.
