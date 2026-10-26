# Unified full-magazine ammo

Status: done (2026-10-08)

User approved removing independent InitialAmmo and setting ExplosionGun capacity2. Checkpoint c6ea8452 preserves preceding engagement changes; unrelated config/rifles/deleted external actor excluded. Existing saved ExplosionGun InitialAmmo2/capacity30 confirmed; reload fills capacity.

## Implementation and verification

Removed InitialAmmo UPROPERTY and spawn override branch. BeginPlay now always uses clamped MagazineCapacity, matching reload. Saved BP_ExplosionGun capacity30->2, other capacities unchanged. Updated existing ammo regression and arch09. Development Editor Win64 build succeeded23.68s. ExplosionGun Blueprint opened/compiled/saved before and after native field removal; cold capacity2 and absent InitialAmmo verified. PIE SavedDamageAndMagazineAmmo Success2026-10-09 02:55:33UTC: spawn2, capacity2, full reload rejected, two shots empty, third rejected, infinite100shots preserves counts, reload2 and spare decremented. Existing damage assertions pass. Final TestMap dirty maps/content empty. Evidence D:/UnrealWork/FullMagazineAmmo.

No new compiler warnings, no editor Error/ensure. Existing test fixture AimIK skeleton warnings, default-pawn fallback and PIE teardown navigation warnings remain unrelated. No map/test fixtures saved. Final changes uncommitted; no push/package/version/retargeting.
