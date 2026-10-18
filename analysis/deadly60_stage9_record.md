# Deadly 60 - Stage 9 record (shop investigation, extra weapons, presentation fixes)

Status: **built and statically checked; not runtime-verified. Deadly 60 is not runtime-verified.**
Fallback candidates `S7_FEATURES_2` and `S8_FEATURES_2` are untouched (hashes in section 7). Lab patches only; generated builds are not committed.

**Cleanup note (2026-10-18): this record describes the Stage 9 candidates `S9_SAFE_2` and `S9_SHOP_2` as they were. Parts of it are superseded by later stages:** the four weapons in section 3 were replaced in Stage 11 (current set: section 3b); the small reused shop icons in section 1 limit (1) were replaced by drawn weapon cards and a NEW GUNS page button (`deadly60_stage10_test_build.md`, stage 11). The Stage 9 candidates themselves are unchanged, and no code or build was changed by this cleanup.

## 1. Shop investigation (the earlier "blocked" verdict was too strong - corrected here)
Method: a read-only SWF tag scanner (Python, run on lab build copies) plus FFDec's own renderer for the Shop sprite.
- The Shop is **one sprite (character 589, class `com.iggyzuk.deadly30.Shop`, 1 frame)** that places 12 weapon icons, 10 shop items and text fields with ordinary PlaceObject2/3 tags (instance names `rifle_1` ... `machinegun_4`, `box`, `bigTurret`, `fence`, ...). The icons are existing library characters (for example 453 = Winchester icon) with on/off frames; the click handlers are AS3 listeners set in `Shop.setButtons` and resolved **by instance name** (`shopRef`) in `Shop.getShopItem`.
- So an extra icon is an additive placement of an existing character under a new instance name, plus a declared `MovieClip` field in `Shop.as`; no new art asset is needed. This was implemented by a small tag-level patch tool (appends 4 PlaceObject2 tags to sprite 589, rewrites that sprite's length and the SWF length; every other tag byte-identical - verified by a tag-by-tag comparison).
- **Real limits found:** (1) the Shop layout is full: the only free area is a ~115 x 40 px patch at the bottom right, so the extra icons are small (scale 0.2) - they reuse existing icons, they are not new art; (2) weapon identity is coupled to the clip name in `Soldier` firing code (pellet patterns, pierce counts, sounds are chosen by `shopRef`) and to `(id - type)/3 + 1` for the in-hand art frame - both decoupled with new `Weapon.fxRef` and `Weapon.artTier` fields (original weapons keep identical values); (3) blueprint selection would pick unfound ids as blueprints with out-of-range art frames - excluded for ids >= 31.
- No game-visible visual check is possible without running the game; the FFDec static render of the Shop sprite shows the 4 icons in the free area (they are hidden by alpha until unlocked).

## 2. Two candidates
| Candidate | Contains | Why separate |
|---|---|---|
| `S9_SAFE_2` | all Stage 7/8 features + presentation fixes (below) | no SWF tag edit, no shop changes beyond hover text and glow |
| `S9_SHOP_2` (experimental) | `S9_SAFE_2` + 4 new shop weapons + the Shop tag patch | touches shop code and the Shop sprite; if the shop misbehaves, use `S9_SAFE_2` |

## 3. New weapons in `S9_SHOP_2` (historical - replaced in Stage 11, see section 3b)
Unlocked by night (survived), bought in the shop like any gun, equipped per ammo type, upgradable with U/I/O and Mk II like the originals, saved in the same bought/equipped id lists (ids 31-34).
| Weapon | Type / fire pattern | Stats (mag / dmg / rate / reload) | Cost | Unlock | Role |
|---|---|---|---|---|---|
| Service Carbine | rifle, Winchester behaviour | 8 / 18 / 24 / 45 | 900 | day 31 | reliable, cheap |
| Sawed-Off | shotgun, Super Double Barrel pattern | 2 / 60 / 25 / 28 | 3200 | day 36 | close range burst |
| Heavy Rifle | rifle, Bolt Action pierce | 3 / 65 / 60 / 90 | 6500 | day 42 | powerful, slow, precise |
| Rapid Repeater | machine gun, AK sound | 60 / 22 / 1 / 80 | 7500 | day 48 | rapid fire, high ammo use |
Numbers are first-pass and unplaytested. The Rapid Repeater uses fire delay 2 (30 damage) and Mk II never slows it. The new weapons stay locked in survival mode; Continue does not re-announce old unlocks. Saves containing these ids cannot be loaded by earlier candidates (unknown weapon ids crash the original loader).

## 3b. Current weapon set (ids 31-34; `FULL_UNLOCK_TEST_*` line, from Stage 11)
Same ids, shop slots, save lists and unlock mechanism as section 3; names, stats and fire effects replaced. Values below are read from the lab export of `FULL_UNLOCK_TEST_15` (`Controller.d60AddWeapons`), which includes the Stage 14 damage buff; they are first-pass and unplayed. Non-test builds use the "Unlock" night; the test builds set it to 0.
| Id | Weapon | Ammo type | Mag / dmg / rate / reload | Cost | Unlock (survived night) | Effect | Exclusive to |
|---|---|---|---|---|---|---|---|
| 31 | Wave Cannon | rifle (1) | 8 / 130 / 26 / 60 | 4500 | 31 | beam | Engineer (skin 3) |
| 32 | Gravity Gauntlet | shotgun (3) | 5 / 110 / 36 / 70 | 5500 | 36 | gravity wells | Engineer (skin 3) |
| 33 | Flamethrower | machine gun (2) | 100 / 16 / 1 / 60 | 6500 | 42 | flame | second survivor, Russian (skin 1) |
| 34 | Tesla Rifle | rifle (1) | 14 / 95 / 16 / 50 | 8000 | 48 | chain lightning | third survivor, German (skin 2) |
Exclusivity (Stage 18): the shop restricts buying and equipping each of these to its owner while the owner is in the squad; the three original families (ids 1-12) are unrestricted. Custom art, effects and sounds for ids 31-34 are described in `deadly60_stage10_test_build.md`. Not runtime-verified. The `S9_SAFE_2` candidate has none of these weapons.

## 4. Presentation (measured from the SWF's text-field records, not guessed)
- Help line (`helpTxt`): 750 px wide, 24 px font, centred, no word wrap -> about 55-60 characters fit. Stage 7/8 messages (<30 chars) fit.
- Night/day banner description (`descText`): 436 px wide, 58 px high, word-wrapped, 25 px font -> **2 lines**. The Stage 6-8 third banner line ("Supply Drop ...") would have been clipped; it now goes to the help line and the banner keeps two lines ("Bonus Scrap:N  XP +M" fits). Night names plus an event tag wrap to two lines.
- Shop hover name field: 422 px wide, 30 px font (about 28 characters) -> the label is `name [DRM]` (upgrade levels) and ` II` for Mk II.
- Still needs visual confirmation in the game: all of the above, the glow filters, the small extra icons.

## 5. Other changes
- Shop hover text shows upgrade levels and Mk II; Mk II weapons get a gold glow in the shop.
- Only the brute gets a glow filter (performance); the other kinds keep their tint; mines blink.
- Mk II no longer renames the weapon (Stage 8 fix kept).

## 6. Checks (static only; no game launched)
- Builds `S9_SAFE_2` (18 patches) and `S9_SHOP_RAW_2` (23 patches) compiled without FFDec errors; manifest, hook-marker and tag gates passed for both; exports differ from `S8_FEATURES_2` in exactly Controller, Level, Shop (safe) and additionally Soldier, Weapon (shop weapons).
- `S9_SHOP_2` = `S9_SHOP_RAW_2` + Shop tag patch: tag-by-tag comparison shows only sprite 589 differs (1031 -> 1123 bytes), declared file length correct, FFDec exports all scripts and renders the Shop sprite.
- Model test of the `d60` save-key logic: 11 of 11 cases passed (matching night+scrap, missing key, night/scrap mismatch, garbled, older formats, out-of-range values). This is a Python port of the algorithm, not execution of the compiled code.
- Independent code review of the diffs and the SWF tool: no compile-level errors or Nights 1-30 regressions. Findings fixed: Mk II slowed the new machine gun; survival mode exposed the new weapons; Continue re-announced old unlocks; clip lookup by index instead of name; glow filters on every variant zombie (performance). Noted, not changed: a save that holds ids 31-34 cannot be loaded by earlier candidates.
- Not verified: everything visual or interactive (shop icon size and clicking, glow, banner/help layout, balance, any gameplay).

## 7. Candidates
- `S9_SAFE_2`: `stage9\builds\S9_SAFE_2\deadly30.swf`, 19,295,949 bytes, SHA-256 `543FEC4823BDBF314E5468116FD0B649CF41460EEFAE27F3F665379F38E04D0A`.
- `S9_SHOP_2` (experimental): `stage9\builds\S9_SHOP_2\deadly30.swf`, 19,297,868 bytes, SHA-256 `6C68E6A8C884637C281FD8CDF558FFAA1029359B61E450016F81D617004335F3`.
(both under `D:\Projects\deadly30_lab\game\d60_rollout\`)

## 8. Fallbacks
`S7_FEATURES_2` SHA-256 `D880B8C8AFBA18CBDCD83F5A029FA78778A900A6999AB557B32616C4A14F2EF2`; `S8_FEATURES_2` SHA-256 `88BCB2C816DF037CDA6A05E3B6222B39B37A4453FC9017A5E3E639607EA72F6E`.
