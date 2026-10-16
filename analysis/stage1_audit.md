# Stage 1 — Audit (read-only)

Scope: Deadly 30 as decompiled from the original SWF (Flash/AS3). Everything here was read from code and never run. File:line references point to the local, uncommitted decompile (`analysis/as3_scripts`, which is not in this repository). No game code, data tables or assets are reproduced here.

## 1. Engine, languages, systems
- Flash/AS3, SWF v10, 750x460 at 30 fps, shipped as an EXE in a Multidmedia Zinc wrapper (`mdm`). Zinc is used only for exit, mouse hiding and resolution; setting `GameController.exe=false` bypasses it.
- Libraries: GreenSock TweenLite. The Newgrounds API is bundled but never called.
- About 50 game classes in `com/iggyzuk/deadly30`. `Level.as` (3,598 lines) is a god class: spawning, waves, loot, save/load, camera, bullets. Run state lives in statics (`Controller`, `Tracker`).
- Rendering: the display list is blitted into one bitmap every frame. No object pooling.
- Decompile quality is clean (no failure markers); local names are lost.
- The FLA and `assets.swf` are not available, so all art, sounds, frame labels and hit-boxes exist only inside the SWF. A full recompile is impossible; the chosen route is patching the SWF's ActionScript in place with JPEXS FFDec.

## 2. Entry points
`MainTimeline` (document class) -> menu frame labels -> `Controller.startGame()` -> `Controller.restart()` -> `new Level().create()`. One `Controller.loop` ENTER_FRAME drives `level.update()`, a fixed sequence of about 35 `updateXxx()` calls.

## 3. Original systems
- **Progression:** a single counter `Level.nightCount` is both night and day number. A day is 5,850 frames (~195 s). When the wave is cleared, `addDay` writes a checkpoint and runs the dawn transition. Every 5th night plays a cutscene (`nightCount/5`).
- **Waves:** data in `Nights.as` (30 rows, 30 concurrent caps, 29 night names). Enemy stats are linear in `nightCount` with no cap.
- **Night 30** is special-cased: a 5:00 countdown, runtime-generated endless waves, then a rescue helicopter, then cutscene 6 and the menu.
- **Weapons:** 12 weapons in one table (`Level.as:1508-1519`) in 3 types sharing 3 ammo pools. Spread, pellets, pierce and the two projectile weapons are hard-coded in `Soldier.as`. No per-weapon upgrades. Unlocks come from blueprints at player level-ups (cap 13).
- **Upgrades:** fence (5 levels), armor (3), boots (3), bought with scrap.
- **Resources:** scrap, three ammo types, medkits (no drops). None are capped.
- **Enemies:** 11 types (ids 0-10) created by one if-chain in `Level.addEnemy`.
- **Base:** fences (hold E to repair, already exists), generator, boxes, two turrets. No repair for boxes or turrets and no cap on how many are placed.
- **Saves:** Flash SharedObject `so_deadly301`. The checkpoint is one unversioned, pipe-delimited string written at dawn inside the base (`Level.as:1594-1705`, read at `1785-1904`). Soldier HP, EXP and box/turret HP are not saved.

## 4. Hard-coded 30-night limits
- `Level.as:340, 984, 2695, 2781` and `HUD.as:122`: `== 30` / `!= 30` branches (night 30 is the finale).
- `Level.as:1172`: `30 - nightCount` goes negative past 30.
- `Nights.as` arrays (30 rows, 29 names) and `Level.as:313` (name lookup `nightCount - 2`).
- `Level.as:2571/2575` and `Controller.as:245-278`: the cutscene id is `nightCount/5`; only 0-6 exist. Id 6 is the win cutscene, which exits to the menu (`Controller.as:179`), so night 30 must not use the normal 5th-night path. Ids 7+ (nights 35, 40, ...) have no cutscene.
- `Level.as:1888-1899`: the checkpoint loader indexes the wave table by `nightCount`, which throws for night >= 31.
- Win flow and text: `Helicopter.as`, `MainTimeline.as:163-170`, `Medals.as`.
- `Tracker.nightsLasted` is set to `nightCount - 1` on load (off by one).

## 5. Compatibility and regression risks
- Old saves can continue into a longer campaign: the loader reads fixed indices and ignores trailing sections. New data must be appended with safe defaults; a missing section currently throws.
- Adding a weapon touches the weapon table, the shop (index-aligned to 12 weapons + 2 turrets), firing branches in `Soldier.as`, blueprint pools, and needs new clips and sounds inside the SWF.
- Adding an enemy needs a new library symbol plus edits to `addEnemy`, `Nights.as` and the unlock lists.
- Existing bugs: wrong array index for super-rare spawns (`Level.as:1143`); duplicate `==9000` branch (`2886/2890`); `Dog.fallBack` is never set; leaving and re-entering the base rebuilds boxes and turrets at full HP (repair exploit).
- Stats are uncapped, so nights 31-60 will outpace the original curve unless the formulas change.

## 6. Unverified
Build and test commands (none exist), the SharedObject location on disk, and whether FFDec can recompile these classes. These are checked in Stage 3.
