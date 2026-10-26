# FEAT-277 Configurable shared enemy hit reactions
Status: done
Closed: 2026-10-12
User authorized implementation and clarified that hit bone only selects the animation; no local bone masking requested. Preserve FEAT235 Animation/ControlRig/Disabled provider contract, existing damage ownership, no-restart and death cleanup. Eight directions are configuration capacity, not a claim that JKMotion supplies a complete eight-way/body-region matrix.

JKMotion read-only audit: 96 reactions + idle, repeated for UE4/UE5 and root/no-root (388 actual sequences). Inspected sampled skeletal poses for all97 UE5 root clips. Nominal four categories contain varied diagonal/torso motion, but no verified complete eight-way or authored bone mapping. Evidence D:/UnrealWork/JKHitReview. No retargeting or asset migration performed.

Implementation: request source/severity/local source direction/point/bone/strength/in-place. Rules filter source, severity, eight-way direction, exact bone or descendants; explicit priority then bone specificity then other filters, array-order tie. Empty/incompatible animation rules fall through; explicit Disabled suppresses. Legacy Projectile/Explosion fallback preserved. Per-rule animation pool/mode/timing/root movement. No new Control Rig graph supplied.
Checkpoint95b0ff3b preserves previous electrical component changes; unrelated dirty workspace remains untouched.

Native implementation compiled (build-final4.68s; build-verified4.54s). First test pass: ExplosionDamageReactions and AttractionProjectile Success. New rule tests passed matching/pools/provider switches but live-postprocess assertion missed the initial .2s fixture window; extended fixture cap to1s and expiry wait1.1s, retest pending. Build first attempt after normal quit raced Live Coding shutdown; subsequent confirmed-exited build passed. No production asset edits.

## FEAT277 configurable shared hit reactions (2026-10-12)

This supersedes the historical September5 four-direction/removed-mode descriptions. FEAT235 provider contract remains authoritative. UEnemyHitReactionComponent::RequestReaction(FEnemyHitReactionRequest) is the single presentation entry. Existing ReactToHit / ReactToExplosion / ReactToElectricalHit are adapters; no damage is applied by this component. Projectile defaults Light, explosion Heavy, electric first Medium and subsequent twitch Light. Strength is blend intensity, independent of severity. Direct custom callers may set source/severity explicitly.

Per-enemy UEnemyHitReactionConfig.Rules selects by Source, Severity, eight-way Direction, and Bones. LocalSourceDirection is actor-local pointing TOWARD the incoming source (+X Front,+Y Right), horizontal45-degree sectors; unknown/vertical direction only matches Any. Bones are exact mesh names; IncludeChildBones allows descendants. Empty list means any bone; invalid/absent hit bones do not match a specific bone rule. Bone matching chooses the clip, not a playback mask; existing stationary/full-body and moving/falling upper-body blending remains.

Precedence: larger Priority, then nearest matching bone ancestor (exact beats group, group beats any), then count of specific source/severity/direction filters, finally earlier array index. Empty/incompatible-skeleton animation pools are skipped. A matching Disabled rule suppresses presentation and does not fall through. Animation pools filter valid same-skeleton clips, deduplicate and avoid repeating the last clip when alternatives exist. Per-rule mode override selects Animation/ControlRig/Disabled; false inherits Config.Mode. Rules own PlayRate/BlendIn/BlendOut/MaxDuration(0=whole clip)/ApplyRootMotion. In-place requests always prevent root movement. Root consumption stops at capped duration and restores movement.

If no usable rule matches, existing Projectile/Explosion four-way/default data remains fallback (electric/melee use Projectile). Legacy unconfigured electrical reactions retain .55s first/.24s twitch timing; matched rules own their timing. No-restart, minimum interval and death cleanup preserved. Mode selects/switches only the configured postprocess provider, never the locomotion AnimBP. Missing ControlRig provider fails safely; no finished rig graph supplied.

Current Phantom and test soldier both reference /Game/Enemy/Humanoid/Phantom/Data/DA_Phantom_HitReaction. Add named entries under Rules to configure; a head rule uses Bones=[head], optional Source/Severity/Direction filters, and appropriate finalized clips in Animations. Existing saved assets keep Rules empty and their established animations. JKMotion audit does not prove complete eight-way/body-part coverage; no JKMotion retarget/migration in this feature. Retargeting remains external-only.

Verification: TheManTest.Enemy.HitReactionRules, TheManTest.Player.Weapons.ExplosionDamageReactions, TheManTest.Player.Weapons.AttractionProjectile. Tests use transient TestMap PIE fixtures; no map saves.

## Final verification
Development Editor Win64 build-verified.log Success4.54s. Actual TestMap PIE: HitReactionRules Success05:59:53, AttractionProjectile Success06:00:01, ExplosionDamageReactions Success06:00:07 UTC. Cold saved config preserved; final.json zero dirty maps/content.
Initial live-pose failure was resolved by a1s fixture window; no production timing workaround. New matcher tests cover eight directions, exact/descendant/invalid bones, priority, source/severity, empty-pool fallback, Disabled, non-repeating pools, legacy fallback, no restart, timed completion, live postprocess and provider switch Animation→ControlRig→Animation. Existing AimIK/Nav warnings present in old fixtures; no new compile warnings.
Main editor41844 open TestMap, PIE ended, throttle restored, DA_Phantom_HitReaction opened for user configuration. Rules empty in saved assets; no false body-region assignments, no JKMotion import or retargeting. No map/asset writes, final Git commit, P4 submit or push. Evidence D:/UnrealWork/HitReactionRules.
