# FEAT-287 Enemy development and file standard

## Scope

User requested an Enemy standard for review. The team-facing draft is `Documentation/Enemy/EnemyDevelopmentStandard.md`. It is based on current architecture and asset layout, with no code, Blueprint, map or animation changes.

## 2026-10-14 draft

- Read the harness, cross-system guide, current asset/AI/movement/engagement/GAS/animation/audio architecture notes, and actual Enemy asset and source paths.
- Wrote the standard covering ownership and names, shared lifecycle tree and engagement authority, three movement shapes, GAS/AbilityTask/Cue/component boundaries, external animation retargeting boundary, and verification.
- Asked for review of two policy choices in the document: required shared lifecycle tree for new formal enemies, and required delivery checklist without retroactive bulk migration.
- UTF-8 document validated; no replacement characters. Perforce `opened` was empty before review. Draft has not been submitted.

## 2026-10-14 onboarding example update

- User requested a concrete example for incoming programmers: what a new Enemy needs and exactly where each file belongs.
- Added a hypothetical ground humanoid `ScrapGuard` example to the draft. It separates new assets from existing shared AI/BT/blackboard/movement files and optional specialized BT/controller/native files, then gives an editor wiring and test order.
- Checked the shared file paths against the current workspace and used Phantom only as a real reference. No `ScrapGuard` assets or C++ files were created.

## 2026-10-14 real example update

- User requested a concrete existing Enemy example. Added formal `BP_Phantom` with actual native class, engagement profile, mesh/ABP, shared AI/BT/BB, shooting ability/projectile and hit/death config paths, plus an ordered reference trace.
- The prior hypothetical example was absent from the current document at turn start despite its archive record; restored it before adding Phantom. The earlier visible Notepad session may have saved an older copy, but the cause was not verified.
- Phantom's `OriginalRifle` historical resources and `BP_Phantom_WeaponTest` test variant are explicitly excluded as new-enemy templates.

## 2026-10-14 HTML review page

- User found the Markdown difficult to read and requested an HTML version.
- Added `Documentation/Enemy/render_standard.py` using Python's standard library and generated `EnemyDevelopmentStandard.html` as a self-contained offline page. The Markdown remains the content source.
- HTML places the real Phantom example first and the hypothetical ScrapGuard example second, with side navigation, readable tables, responsive layout and print styling; all later standard sections remain included.
- HTML parser verified anchor uniqueness/navigation targets and four tables. Headless Edge screenshot was visually inspected after the first layout pass; path table width and line breaks were improved for the final pass.
- No UE code or assets changed; no Perforce submission pending user review.

## 2026-10-15 shared task and skill-set clarification

- User pointed out that earlier Void routing bypassed the intended `PhaseSkillSets` action selection and that the Enemy standard did not enumerate shared skill/point-selection tasks. The standard was clear on file ownership but incomplete on behavior wiring.
- Added a shared Task and point-query catalog with inputs, outputs, owners and failure boundaries: lifecycle subtree, combat skill selection, ground/wall presets, split ground find/move, wall find, and ground/wall/air queries. CoreMorph flight remains an explicit specialized consumer.
- Documented the current phase/distance-pool selection chain, fixed auto-range thresholds (500/800 cm), shuffled activation attempts, GA eligibility checks, retry/post-skill delays, Void's configured action categories, and a verification item for invalid candidates and apparent idle time.
- Updated the offline HTML renderer/navigation and regenerated the review page. Renderer also now recognizes the real Phantom example heading. Documentation only; no UE build or Perforce submission.

## 2026-10-15 Enemy architecture workflow correction

- User clarified that preventing the next Enemy workflow mistake matters more than the reader-facing standard. The initial response overcorrected by placing detailed Enemy rules directly in harness `AGENTS.md`. User then clarified the intended structure: `AGENTS.md` only routes Enemy work to relevant architecture documents, and those architecture documents own the detailed rules. Reverted all edits made to harness `AGENTS.md` in that correction; its existing Enemy trigger map remains.
- `arch/11-enemy-ai.md` now requires checking saved BP skill pools/BT, recording a stage/distance/surface eligibility matrix and failure/wait conditions in the active feature archive, and reusing `BTTask_UseCombatSkill` unless a documented exception is needed. `arch/15-enemy-movement.md` catalogs shared Task/query/motor entry points and their execution boundaries.
- Added the AI+movement read order to `arch/cross-system-guide.md`. Added a current skill-selection and shared-task section to `arch/11-enemy-ai.md`, fixed its interrupted family table, and moved stale 450/200 cm assumptions out of the current guidance. Added a current point-selection Task catalog to `arch/15-enemy-movement.md` and corrected the Void consumer/old Combat wall preset description.
- Existing unrelated edits in harness `AGENTS.md` and the reader-facing documentation update were captured in scoped local WIP checkpoint `f07dfd44` before these changes. The worktree version of `AGENTS.md` now matches that checkpoint. No gameplay code or assets changed; no UE build required. HTML anchors, five tables, UTF-8 and JSON parse validated.

## Next

Keep Enemy implementation rules in the system architecture documents reached through the existing harness trigger map. Continue user review of the separate team-facing standard, then mark that standard effective and submit only the approved document to Perforce. Do not infer approval from silence.
