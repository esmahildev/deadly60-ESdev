# Deadly 60 - Stage 8 feature batch (record)

Status: **built and statically checked; not runtime-verified. Deadly 60 is not runtime-verified.**
Lab patches only (nothing generated is committed). Builds on Stage 7 (`deadly60_stage7_features_record.md`); the Stage 7 candidate `S7_FEATURES_2` is untouched (SHA-256 `D880B8C8...2EF2`).

## 1. What was investigated first
- **New weapons with new shop slots - blocked, with evidence.** The 12 shop icons (`rifle_1` ... `shotgun_4`) are anonymous timeline instances inside the `Shop` symbol: `Shop.as` declares them as plain `MovieClip` fields and there is no exported class for any of them (no per-weapon class exists in the decompiled source), so code cannot instantiate a 13th icon. Adding one means editing the SWF's Shop timeline itself (FFDec `-swf2xml` / `-xml2swf` round trip of a 19 MB SWF), which is memory-heavy on this laptop and cannot be validated without launching the game. A wrong edit would break the shop screen for the whole game. It is deferred, not impossible; the safe substitute below is implemented instead.
- **Ranged zombie by reusing the puke projectile - not reused** (the fatty's puke is a screen-space displacement effect, not a projectile); a small purpose-built projectile was written instead.

## 2. Implemented in this batch (all in existing classes; no new art)
| Planned item | Implementation | Counterplay |
|---|---|---|
| New weapons (late-game, progression-unlocked) | **Mk II evolution**: key L at the base in daytime, needs all three upgrades at level 3 on that weapon and day >= 40, cost `5000 x (tier+1)`. Rifles +25 % damage; machine guns +20 % fire rate, -15 % reload, +5 % damage; shotguns +20 % damage, +2 shells. Weapon name is NOT changed (the game compares names for the Laser Gun and Golden AK beams); Mk II is shown on help page 1. Persisted in the same `d60` key (bit 6) | costly, one per weapon |
| Explosive / disruptive zombie | **Bloater** (green tint, from night 38; event night 37): explodes on death; soldiers within 130 px take 25 damage, other enemies within 130 px take 60 | kill it at range; it also hurts other zombies |
| Ranged / special attacker | **Spitter** (cyan tint, from night 44; event night 49): fires a slow green projectile every 5 s at a soldier 180-420 px away, with a growing green dot as wind-up; 10 damage | duck, jump, or kill it first; hits only a standing soldier on the ground |
| Boss / elite | **Brute** (dark red tint): 4x HP, +60 % damage, -30 % speed, 6x XP; only on event night 45 (8 % of spawns) | kite it, mines |
| Fast zombie, armored zombie, elite | Stage 7 variants swift / armored / elite (unchanged apart from the shared variant table) | |
| Reinforced barricade | **Plating** (key G, 3 levels, `2000 x level`): zombie damage to the fence -15 % per level | |
| Slowing / area-denial device | **Barbed wire** (key B, 3 levels, `1500 x level`): zombies within 260 px outside each fence are slowed (about 17 / 28 / 38 %); drawn as wire along the ground | |
| Defensive trap | **Mines** (key T, `250` each, up to 4 at fixed spots outside the fences, base room only): trigger when an enemy is within 40 px; 150 damage within 150 px; not saved (the placement message says so) (cleared by Continue/restart, like scrap spent after the dawn checkpoint) | |
| Special events | night 37 Bloater, 42 Elite Hunt, 45 Brute, 47 Swift, 49 Spitter, 52 Armored; a tag on the night banner; themed variants 30 % (Brute 8 %); nights 56 and 58 are excluded from all variants (approved B6/C8 rows untouched) | |
| Resource rewards, XP, perks, upgrades, supply drops | unchanged from Stage 6/7 | |

Hotkeys at the base in daytime from day 30: U damage, I reload, O magazine (selected weapon); J Endurance, K Scavenger; G plating, B wire, T mine, L Mk II; H cycles three info pages. Every purchase asks for the key twice (first press shows the price). All help messages are under 30 characters.

## 3. Save compatibility
`saveString` and checkpoint structure unchanged. The additive `d60` key now stores `night#scrap#<13 weapon values 0..127>|endurance,scavenger,plating,wire`; it is ignored unless its night and scrap equal the first two `saveString` fields. Older-format values load with defaults. Mines and spit projectiles are not saved.

## 4. Static checks (no game launched)
Build `S8_FEATURES_2` (15 patches, 18 classes) compiled without FFDec errors; manifest, hook-marker and tag gates passed; exported scripts differ from the Stage 7 candidate in exactly Controller, Enemy, Weapon, Zombie. Independent code review: see section 5.

## 5. Review
Independent review of build S8_FEATURES_1 found one blocking and two should-fix defects, all fixed in S8_FEATURES_2:
1. Mk II renamed the weapon, which broke the Alien Laser Gun beam and Golden AK-47 bullet (Soldier compares names) - fixed: no rename.
2. Mines were drawn and triggered in every room - fixed: only in the base room.
3. Bloater damage used a roof flag instead of a vertical check - fixed: soldiers within 80 px vertically are hit.
Accepted low items: mine/wire anchoring uses the fence bounds at purchase/draw time (may shift if the fence frame changes), mines are not saved.
Suggested in-game checks from the review: Mk II on the Laser Gun and Golden AK, a mine after travelling to another room, a Bloater on the roof, Continue after buying wire/plate/mines.

## 5b. Final candidate
`D:\Projects\deadly30_lab\game\d60_rollout\stage8uilds\S8_FEATURES_2\deadly30.swf`, 19,295,362 bytes, SHA-256 `88BCB2C816DF037CDA6A05E3B6222B39B37A4453FC9017A5E3E639607EA72F6E`. Manifest, hook-marker and tag gates passed; exported scripts differ from the Stage 7 candidate in exactly Controller, Enemy, Weapon, Zombie.

## 6. Not done / not verified
- New weapon models with their own shop slots (section 1).
- New zombie sprites, new structure sprites, wind-up animations (art).
- Everything above at runtime, including layout of help text and banner lines, projectile/mine/wire visuals, balance of the event nights, Night 60 finale, real-save Continue. The owner will play.
