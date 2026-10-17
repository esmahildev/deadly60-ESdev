# Deadly 60 - FULL_UNLOCK_TEST (temporary test build, record)

Status: **compile/static checks only; not played. A test tool, not a release candidate.** Lab patches (`stage10\patch_t`) and generated files are not committed.

## What it is
`S9_SHOP_2` (experimental shop weapons) + a small test patch + a separate launcher and save folder. Built as `FULL_UNLOCK_TEST_RAW_1` (26 patches, code) + the Shop tag patch = `FULL_UNLOCK_TEST_1`.

## Test state (all through existing save fields; written once on the first launch)
| Item | How |
|---|---|
| Start at night 10 | `saveString` field 0 = 10 (the game's own counter; "DAWN 10", then night 11). `-Night N` starts elsewhere (2..59) after `-Reset` |
| Continue | `nightsLasted` = N-1 is present, which is what the menu checks. Also in the code (all later checkpoints): a checkpoint now writes `nightsLasted`, so closing the window mid-game no longer disables Continue |
| People | the game has three survivors (skins 0 American, 1 Russian, 2 German; the "Union" medal is the full crew); all three are in the save's survivor list (they are on the player's side, as for any recruited survivor). Their weapons: Golden AK-47, Alien Laser Gun, Striker |
| Weapons | all 12 originals + the four Stage 9 weapons (ids 31-34) in the bought list; every shop entry is marked found by the existing survival-mode path; the new weapons' night locks are 0 |
| Scrap | exactly 999999 in the save; no code resets it |
| Upgrades | every weapon at damage/reload/magazine 3 and Mk II, Endurance 3, Scavenger 3, Plating 3, Wire 3 (the optional `d60` key, matching night and scrap); armor 3, fence 5, boots 3; 9 medkits; ammo 999 per type; player level 20 |
| Day-30 / night-40 locks | hotkeys (U I O J K H G B T L) work at any night; Mines (T) are bought with scrap as usual |
| Variants, thorns, rewards at low nights | with the test flag they are active at any night (variants use the night-44 pool, so swift/armored/bloater/elite/spitter can spawn); dawn survival bonus/XP are clamped at 0 below night 31 |

## Limitations
- Event-night tags/themes (nights 37, 42, 45, 47, 49, 52) and the Brute depend on the real night number: start with `-Reset -Night 36`, `41`, `44`, `46`, `48`, `51` to reach them without grinding.
- Night 56/58 balance rows and the Night 60 finale are the real ones; start at 55 or 59.
- Mines are not saved (design).
- Saves with ids 31-34 cannot be loaded by earlier candidates.

## Isolation
Progress folder `DEMO\saves\FULL_UNLOCK_TEST` only (Ruffle `--storage disk`). `SHOP_CHEAT`, other builds, the real save and `acceptance\savetest` are untouched. `-Reset` moves progress to a backup folder; nothing is deleted.

## Update (stage 11): new-weapon cards
The four experimental shop weapons were replaced by Wave Cannon, Gravity Gauntlet, Flamethrower and Tesla Rifle (shop cards drawn from the owner's concept art, a NEW GUNS page button, distinct firing effects), plus six-digit scrap font fixes. Test build FULL_UNLOCK_TEST_3 (stage11\builds), 19,455,330 bytes, SHA-256 948F46912B10378B399C1A0603C187A3E9B42E16E88AD62EF9E3D07186CC43AB. Static checks only; not played.

## Update (stages 12-13): in-hand art, effects, sounds
The four new weapons show concept-art guns in the soldier's hands (cut-out sprites added to the arms animation), coloured HUD weapon names, bigger firing effects, and five synthesised sound effects (new DefineSound tags + trigger sprites in the HUD; 22.05 kHz PCM; flamethrower uses no-overlap playback). The sounds are procedural and have only been checked numerically (no clipping, durations, spectra), not heard. Test build FULL_UNLOCK_TEST_5 (stage13\builds), 19,612,500 bytes, SHA-256 D89AB72796728EA36786F96E545AB7586EE5CE4F1D42C5400641F3475832BA09.
