# Deadly 60 - Stage 7 feature batch (record)

Status: **candidate built and statically checked; not runtime-verified. Deadly 60 is not runtime-verified.**
All changes are lab patches (never committed here). This file is documentation only.
Cleanup note (2026-10-18): rows 2, 3, 5, 7, 10, 12 and 14 carry *Update* / *Superseded* annotations pointing to the later stage records; the original Stage 7 wording is kept where it is still historically accurate. No code or build was changed by this cleanup.

## 1. Requirement checklist (from `newplan.md`, judged against the source)

| # | Requirement | Status | How / reason |
|---|---|---|---|
| 1 | 60 days and nights | Implemented (Stages 1-4); synthetic-save loads checked | see earlier stage records |
| 2 | Additional weapons | ~~Blocked~~ **Superseded - implemented in the experimental/test line (not played)** | *Historical verdict at Stage 7: a weapon needs a shop slot (movie clips placed on the Shop timeline, `Shop.as:134` index-aligned) and gun sprites; neither can be added with source-only patches.* **Corrected in `deadly60_stage9_record.md` section 1** (extra shop icons are additive tag patches). Current state: four weapons, ids 31-34 (Wave Cannon, Gravity Gauntlet, Flamethrower, Tesla Rifle), in `S9_SHOP_2`-derived builds and the `FULL_UNLOCK_TEST_*` line only; the `S7`/`S8`/`S9_SAFE` candidates have none. Details: `deadly60_stage9_record.md` section 3b and `deadly60_stage10_test_build.md`. The original has 12 weapons plus blueprint unlocks (`Level.addBluePrint`, player level) |
| 3 | Weapon upgrades | Implemented (code), untested in play | hotkeys U / I / O at the base in daytime from day 30: damage +10 %/lv, reload time -10 %/lv (floor 12 frames), magazine +20 %/lv; max 3 levels each; cost `400 x (level+1) x artTier` (artTier = the weapon's 1-based tier: 1-4 for the original rifle/machine-gun/shotgun families, see `Controller.d60WeaponCost`); press twice to buy (first press shows the cost) |
| 4 | Barricade repair | Already in the original | `SafeRoom` hold-to-repair for scrap, fence upgrade levels 1-5; left unchanged |
| 5 | Defensive structures | Partly | existing: boxes, small/big turrets, fence upgrades. New: **spiked fence** (contact damage to zombies attacking the fence, from night 31, fence level >= 2, 3 x (level-1) per hit). New sprites for traps/other structures: blocked (art). *Update (Stage 8): plating, barbed wire and mines were added without new sprites, see `deadly60_stage8_features_record.md` section 2.* |
| 6 | Resources | Implemented | dawn survival bonus (night-30)x40 scrap, supply drops on nights 35/40/45/50/55 (health pack + ammo) - Stage 6 design kept |
| 7 | New zombie types | Partly | three **variants** of the existing four zombie types (swift, armored, elite), shown by colour tint; introduced swift from night 31, armored from 36, elite from 41; excluded on nights 56 and 58 (approved B6/C8 balance rows untouched). New sprites (ranged, explosive, boss): blocked (art). *Update (Stage 8): Bloater (explosive), Spitter (ranged) and Brute (boss) were added as tinted variants of existing sprites; still no new zombie sprites.* |
| 8 | Attack behaviours | Partly | armored = +40 % damage (pressures the fence), swift = faster / fragile, elite = +25 % damage. Wind-up animations: blocked (art) |
| 9 | Gradual difficulty | Implemented | variant chance 8 % at night 31 rising 0.3 %/night; wave data from Stage 1 unchanged |
| 10 | Special night events | Implemented (3 at Stage 7; **6 as of Stage 8**) | night 42 Elite Hunt, 47 Swift Night, 52 Armored Night: banner tag + 30 % themed variants; pure function of the night number (nothing to save, cannot repeat). *Update (Stage 8): nights 37 Bloater, 45 Brute and 49 Spitter were added, see `deadly60_stage8_features_record.md` section 2.* |
| 11 | XP and rewards | Implemented | existing XP/level system extended: dawn survival XP `30 + 3 x (night-30)` for nights 31-59 |
| 12 | Unlockables | Existing system kept; **extended in the experimental/test line** | player-level blueprints unchanged. *Update: the ids 31-34 weapons unlock by survived night (31 / 36 / 42 / 48) and are bought in the shop; Mk II (Stage 8) needs all three upgrades at level 3 plus night >= 40. See `deadly60_stage9_record.md` section 3b.* |
| 13 | Character upgrades | Implemented, limited | perks J Endurance (-6 % damage taken per level) and K Scavenger (+10 % dawn scrap per level), max 3, cost 1500 x (level+1) |
| 14 | Preserve original | Honoured | nothing changes before day 30; upgrades/perks locked until day 30; no new menu *(true at Stage 7; the experimental/test line later added a NEW GUNS page button in the shop, see `deadly60_stage10_test_build.md`)* |
| Extras | Quality of life | Implemented | in-game help text for costs (H shows weapon levels and perk levels) |

## 2. Save compatibility
- `saveString` and the checkpoint structure are **unchanged**.
- One additive, guarded SharedObject key `d60` (string `"<13 packed weapon levels>|<endurance>,<scavenger>"`) is written next to `saveString` in `Level.addCheckPoint` and read in the Continue path; a missing key (all old saves, all synthetic saves) means level 0 everywhere. The original game ignores unknown keys (same pattern as `lvl` / `achievedGuns`). Because it is written at the same moment as the scrap total, reloading a checkpoint cannot keep upgrades while scrap reverts.
- New game and restart call `Controller.d60Reset()`.

## 3. Files changed (lab patches `stage7\patch\07_*.diff`, applied after Stages 3, 4, 6)
`Controller.as` (statics and functions `d60*`), `Level.as` (hooks), `Weapon.as` (base stats and `d60Refresh`), `Soldier.as` (Endurance factor), `Zombie.as` (spiked fence).

## 4. Static checks run (no game launched)
- Build: `harness\build.ps1` exit 0, FFDec reported no errors; manifest gate, hook-marker gate and tag gate (only the code tag differs from the original) passed.
- Decompiled export of the built SWF compared with the Stage 4 export: exactly 5 classes differ (Controller, Level, Soldier, Weapon, Zombie); the new functions are present and operator precedence is as written.
- Independent code review of the diffs (section 6); its fixes are in the final build (section 7).
- Messages shown in the in-game help text are kept under 30 characters (the longest original message is 29) because the text field width could not be measured without running the game.

## 5. Not verified (needs the owner's manual play)
Everything above at runtime: hotkeys and messages, upgrade effects, perks, variants and tints, spiked fence, event banners, dawn banner layout with the extra XP / supply text, Night 60 finale, real-save Continue.

## 6. Review findings (independent code review of build S7_FEATURES_1) and what was done
| # | Finding | Action |
|---|---|---|
| 1 | Scavenger ignored on the dawn bonus recomputed during Continue (`d60Load` ran after `loadCheckPoint`) | fixed: `d60Load` now runs before `loadCheckPoint` |
| 2 | `d60` key not tied to its `saveString` (another build overwriting `saveString` would leave stale upgrades) | fixed: `d60` now starts with night and scrap and is ignored unless both equal the first two `saveString` fields |
| 3 | Two-press confirm not bound to the weapon (switching gun between presses bought at a different price) | fixed: pending purchase remembers the weapon id |
| 4 | Swift zombies could be fast enough to skip the fence hit-test (unverified) | mitigated: swift speed gain capped at 3.0; still needs a play test on nights 50-60 |
| - | Nights 1-30 unchanged (factor 1.0 at level 0, all new code gated above night 30) | confirmed by review |

## 7. Final candidate
`D:\Projects\deadly30_lab\game\d60_rollout\stage7\builds\S7_FEATURES_2\deadly30.swf`, 19,290,845 bytes, SHA-256 `D880B8C8AFBA18CBDCD83F5A029FA78778A900A6999AB557B32616C4A14F2EF2`. Gates on this build: manifest, hook-marker and tag gates passed; exported scripts differ from Stage 4 in exactly Controller, Level, Soldier, Weapon, Zombie. Not run: the synthetic-save Ruffle checks, because the owner will play manually.
