# Deadly 30: Extended Survival Edition — Stage 4 Implementation Plan

Status: **PLAN ONLY. Nothing here is implemented or approved for implementation.** This document contains no game code, assets or saves.
Written for: the project owner, as the gate before any gameplay change.

Evidence tags used below:
- **R** = re-read in the decompiled source during this planning pass (file:line given).
- **A** = taken from the Stage 1 audit and not re-read now; spot-check the line before editing it.
- **T** = established by the Stage 3/3b tests (runtime evidence).
- **ASSUMPTION** = unverified; a stage below contains the task that verifies it.

Source paths are relative to `com/iggyzuk/deadly30/` unless they start with `GameController`, `MainTimeline` (`Deadly30_fla/`) or are written in full.

---

## 0. Decisions that shape the plan (recommended defaults; tell me if you disagree)

| # | Decision | Default used in this plan |
|---|---|---|
| D1 | Separation of Classic and Extended data | **Everything is separate** (checkpoint, best nights, XP level, unlocked guns, medals) except display settings (quality, resolution). Classic data is never written by Extended code. |
| D2 | Existing Classic saves in Extended | **Not converted, not read implicitly.** An optional, explicit "import a copy of my Classic checkpoint into Extended" action may come later (copy only, never move). This supersedes the Stage 2 default ("old saves load as Extended"), because you now require that old saves are never silently converted. |
| D3 | Mode-selection UI before the art workflow exists | An **interim programmatic text toggle** on the menu (no new sprites), replaced by proper art once the asset workflow is proven. |
| D4 | Bug-fix policy | Classic keeps the original behavior, bugs included (logged, not fixed), except crash-class hardening that cannot change a valid save's behavior. Fixes for known bugs apply to Extended only. |
| D5 | What the end user runs | Not decided here. Testing uses Ruffle with the verified flags; the original executable is never replaced. A deployment decision is needed before release, not before Stage 4 work. |

---

## 1. Current verified baseline and known limitations

### 1.1 Verified (T)
- Pinned runtime: Ruffle desktop nightly **2026-10-16** (`0.8.0-nightly.2026.10.16`), `dx12`. Verified flags: `--graphics dx12 --storage memory|disk --load-behavior blocking --dummy-external-interface` (plus `--no-gui --width 750 --height 460` and isolated `--config/--cache-directory`).
- The original SWF and the 18-class no-change rebuild behave the same in: menu, new game, movement, firing/reload, enemy spawning, damage, player death, game over, pause, Continue from a real-save copy, sleep -> night -> dawn with checkpoint write (night 12 -> 13), 15 of 16 runs with 0 errors.
- Rebuild route: FFDec 26.3.0 `-replace` of AS3 scripts changes only the `DoABC2` tag; all other tags are byte-identical. Re-decompiling the rebuilt SWF differs from the original only in compiler-inserted wrappers (15 residual lines in 18 classes, none behavioral on inspection).
- Save facts: one SharedObject per slot, named `so_deadly30` + slot number; slot 1 is used everywhere today. Ruffle persists it **only on a graceful exit** (the game never calls `flush()`), keys the file by **the SWF's path**, and a hard kill loses unsaved data.
- Real-save checkpoint format confirmed by loading one: `night,scrap,medkits,armor,fence,boots,ammo0,ammo1,ammo2,kills | soldier skins | soldier gun ids | bought weapon ids | boxes (x,y:...) | turrets (x,y,scaleX,type:...)`.

### 1.2 Not verified (must not be assumed)
Gameplay of nights 1-30 beyond the paths above; **night 1's dawn** (not reached in 4 attempts; a test-coverage gap, not evidence of a game bug); nights after 13; the Classic finale (night 30 countdown, helicopter, win cutscene, return to menu); the shop UI; weapon switching between soldiers; multi-night sessions; large-wave performance; any behavior of a patched class beyond the paths exercised.

### 1.3 Known limitations
- No tests exist in the game; Ruffle runs at an effective ~19-24 fps here (inferred), so timing assertions must use frames/trace order, not wall time.
- `Math.random` is not seedable: runtime tests must be structural (traces, counts, ranges), not exact-value.
- Mouse input needs the real cursor; a hard exit loses saves; storage is path-keyed.
- No FLA/`assets.swf`: art, sounds and clip structure live only inside the SWF; the XML round-trip re-encodes 60 art tags and is **not** an acceptable patch route.

---

## 2. Architecture and dependency map

### 2.1 Design principles
1. **Classic is the control group.** Every change is gated so that, with the Extended flag off, behavior is identical to the original; a source-diff gate and runtime oracle prove it (section 6).
2. **Data-driven, one home.** Tunables live in static members of the existing `Nights` class (proven edit path), not scattered literals. No new classes until the workflow spike proves FFDec can add them (ASSUMPTION).
3. **Mode flag follows precedent.** `GameController` already holds mode flags `survival` and `gameContinue` (GameController.as:55-57, R). Add `extended` and `saveSlot` beside them.
4. **No new art or sound** until the asset workflow (Stage 0b) is proven; every art-dependent item is parked behind that gate.
5. **Test hooks never enter a production SWF.** Hooks exist only as a separate patch applied to disposable test builds; the build script refuses a production build if any hook marker is present.

### 2.2 Modules and dependencies
```
Stage 0a  build+gates harness (no game code) ---------------------------+
Stage 0b  asset-workflow + FFDec capability spike (disposable) ---+     |
Stage 0c  baseline coverage on ORIGINAL (finale, shop, n>13) -----|-----+--> prerequisites for everything
                                                                  |
Stage 1   mode flag + save-slot separation  <---------------------+
   |
Stage 2   finalNight plumbing (Classic identical)
   |
Stage 3   Extended skeleton 31-60 + night-30 milestone + finale at 60 + loader hardening
   |-----------------------------+------------------+
Stage 4  difficulty bands     Stage 6  repair+persist   Stage 8  weapon upgrades
   |                             |                          |
Stage 5  special events       Stage 7  reinforced base  Stage 9  XP/rewards/unlocks/char upgrades
   +-----------------+-----------+--------------+---------+
                     |
Stage 10 zombie behaviors / tint variants  -->  Stage 11 (GATED on 0b) new art: weapons, enemies, traps, mode-select art
                     |
Stage 12 balance, performance, full regression, final verification
```

### 2.3 Lab layout (outside git; nothing proprietary is ever committed)
```
D:\Projects\deadly30_lab\game\
  src_baseline\   exported scripts of the original (read-only)
  patches\NN_<stage>\   one unified-diff set per stage (private, never pushed)
  builds\NN_<stage>_{prod|test}.swf   + manifest (SHA-256 of sources, patches, SWF)
  run\deadly30.swf    FIXED path used for every save-related test
  savetest\<run-id>\  fresh disposable save dir per run
```
The repository receives only plans and reports. Rollback = rebuild from the previous stage's patch set; the baseline and all prior builds stay on disk with hashes.

---

## 3. Stages in implementation order

Each stage lists: purpose, exact code touched (with evidence), design, acceptance criteria, rollback. Sizes: S (<1 day), M, L.

### Stage 0a — Reproducible build and gate harness (S, no game-code change)
**Purpose:** make every later step verifiable before any gameplay edit.
**Work:** scripts (lab only) that (1) take `src_baseline`, apply an ordered patch set, (2) replace the touched classes with FFDec `-replace` (class names as FFDec expects: e.g. `Deadly30_fla.MainTimeline`), (3) write a manifest, (4) run the **tag gate** (only `DoABC2` differs from the original SWF), (5) re-export scripts from the output and run the **source-diff gate** (differences vs the baseline export must lie only inside hunks listed by the stage, after normalizing the compiler's `Boolean(...)`/`Number(NaN)`/`x = x - 1` wrappers), (6) refuse to emit a production SWF if a test-hook marker is present, (7) copy the SWF to the fixed run path, (8) run the smoke scenario (Ruffle verified flags; menu -> new game -> scripted death; compare the trace sequence with the baseline).
**Acceptance:** with an **empty** patch set the harness reproduces the Stage 3 results: tag gate = 1 tag; source-diff gate = only known wrappers; smoke trace sequence = original's; hashes recorded.
**Rollback:** n/a (no game change).

### Stage 0b — Asset-workflow and FFDec capability spike (M, disposable copies only)
**Purpose:** decide, with evidence, what can be added without new proprietary art: this gates Stage 11 and several interim-UI choices.
**Questions to answer (each ASSUMPTION today):** can FFDec add a new AS3 class (not just replace)? can it replace a single shape/sprite/sound by character id while leaving all other tags byte-identical (verify with the tag gate)? can a library symbol be duplicated and recolored? can a programmatic `TextField` use an embedded game font (the SWF contains `Lobster`-family fonts) so interim UI looks consistent? does `gotoAndStop(n)` beyond a clip's last frame clamp in the player and in Ruffle (needed for fence levels > 5, see Stage 7)?
**Acceptance:** a short written workflow doc, plus one proof per answered question on a disposable copy, each with a tag-level diff showing only the intended tag(s) changed. Unanswered items are recorded as "not possible", and the dependent features move to "cut or interim".
**Rollback:** n/a.

### Stage 0c — Baseline coverage on the ORIGINAL SWF (M)
**Purpose:** convert today's "unverified" into verified baselines before comparing a patched build against them.
**Tasks (original build, fixed path, disposable storage, graceful close):**
- Classic finale: byte-patch a disposable copy of the real-save checkpoint to night 29 (see 3.0.1; prove the technique first, else use the 3.0.2 hook), sleep to night 30, observe the 5:00 countdown (`addDeadly30Timer`, Level.as:340-343 R), the helicopter (`Level.addCopter`, Helicopter.as:100-103, 178-184 R), cutscene 6, `removeCutscene(6)` (Controller.as:179-212 R) and the return to the menu; record the trace sequence and best-nights value.
- Nights 14-29 with the loaded squad: record traces of `NIGHT!`, `SAVE STRING`, `DAY!` per night.
- Shop UI: open the shop at the workbench, buy/equip one weapon and one upgrade, close; record traces and HUD.
- Night 1 dawn: use a **test-flavor** hook (3.0.2) to reach it deterministically; keep as a baseline trace.
**Acceptance:** each item has a recorded trace/screenshot set and a pass/fail/unverified entry. Failures found in the original are logged as baseline bugs, not fixed.

#### 3.0.1 Byte-patched save recipe (test technique, **ASSUMPTION: untested**)
The disposable save copy is an AMF `.sol` (311 bytes in the real save's case); the night number is the first two characters of the checkpoint string. In theory, replacing `12` with another two-digit number keeps every length field valid (same byte count), so the harness could generate nights 29, 30 and 59 starting points from a copy without any game hook; for Extended slots the same trick would apply to a copy placed in the slot-2 file. **This has not been tried**: Stage 0c must first prove it on a disposable copy (the game loads it and logs the expected `LOADED STRING`). If it fails, the fallback is the test-flavor night-jump hook (3.0.2), which is also disposable-only. Stages 3-5 acceptance depends on one of the two working.

#### 3.0.2 Test flavor
A separate patch (marker comment `// TESTHOOK`), applied only to disposable test builds: (a) print per-enemy stats at spawn, (b) print the wave table digest at each `addNight`, (c) optional key to set the night counter and skip a night. The same hook patch is applied to the baseline and to the candidate so differences isolate to the feature patch. Never shipped.

---

### Stage 1 — Mode flag and save-slot separation (M) — foundation
**Purpose:** guarantee Classic data can never be touched by Extended code, before any Extended feature exists.
**Exact code (R unless noted):**
- `GameController.as:55-57` add `extended:Boolean=false`, `saveSlot:int=1` next to `survival`/`gameContinue`.
- `GameController.as:271-275` `getSharedObject(param1:int=1)` builds the name `"so_" + gameName + param1`; **all 14 call sites use the default** (Settings.as:86; MedalManager.as:45; Level.as:1497, 1700, 1792; MainTimeline.as:156; HUD.as:357; Controller.as:195, 226, 520; GameController.as:140, 295; ClearSaveDataButton.as:19, 72). Plan: change the default so an omitted argument resolves to `saveSlot`; the display-settings sites (GameController.as:140, 295; Settings.as:86) pass slot 1 explicitly so quality/resolution stay shared; `ClearSaveDataButton` keeps slot 1 (its current behavior).
- `MainTimeline.as:75-88` (`playGame`, `continueGame`) and `154-199` (menu frame: reads `nightsLasted`, builds the "You Lasted N Nights" text, enables Continue only when `nightsLasted >= 1`): set `GameController.extended`/`saveSlot` from the selected mode; re-read the selected slot when the mode changes; Continue enabling and listeners must be re-evaluated (currently added once, lines 176-186).
- Interim UI (D3): a programmatic text toggle ("Classic" / "Extended"), default Classic every launch (the selection is kept in memory only, so Extended code never writes to slot 1).
**Design notes:** Extended slot 2 file is `so_deadly302`. Classic's keys are unchanged. Settings stay shared. No automatic migration (D2).
**Acceptance:**
- Classic flows write only `so_deadly301`; after an Extended session the Classic file is byte-identical (hash before = after) on disposable disk storage.
- Extended writes only `so_deadly302` (separate file in the same save dir).
- Classic Continue from the real-save copy loads exactly the original string/flow (compare with Stage 0c trace).
- Source-diff gate: only the hunks above. 0 runtime errors with `--dummy-external-interface`.
**Rollback:** remove patch `01_slots`; slot 1 data was never modified.

### Stage 2 — `finalNight` plumbing, Classic bit-identical (M)
**Purpose:** replace every hard-coded 30 with one accessor, with **no behavior change** in Classic.
**Add:** `Nights.finalNight()` returning 30 when not extended, 60 when extended (static function in `Nights.as`, R: class holds only `addNights`, Nights.as:11-73).
**Every hard-coded night-30 site (R):**

| Location | Current | Change | Classic-equivalence argument |
|---|---|---|---|
| Level.as:340 | `nightCount == 30` -> create finale timer in `addNight` | `== Nights.finalNight()` | same value in Classic |
| Level.as:984 | `nightCount != 30` guards signpost key | `!= finalNight` | same |
| Level.as:1172 | `Math.random()*(30 - nightCount) == 0` (alarmed spawn) | `(finalNight - nightCount)` | same for Classic; for Extended n>30 the expression stays positive instead of negative (otherwise "alarmed" would never fire) |
| Level.as:2695 | `if(nightCount != 30)` wraps the whole night/day cycle | `!= finalNight` | same |
| Level.as:2781 | `updateNight30` early-returns unless 30 | `!= finalNight` | same |
| HUD.as:122 | clock icon alpha 0 only at night 30 | `== finalNight` | same |
| MainTimeline.as:163-170 | `<30` text, `==30` "Survivors have escaped!" | use `finalNight` (and slot-specific `nightsLasted`) | same |
| Medals.as:39 | medal 18 text "Survive all 30 days and nights" | mode-aware text | text only; Classic unchanged |
| Level.as:1888-1899 | loader loop over `waveArray[0..nightCount-1]` | bound by `waveArray.length` | no effect for valid Classic saves (nightCount <= 29) |
Left untouched on purpose: Level.as:2711 (`nightCount % 5 == 0` -> cutscene), 2571/2575 (`nightCount/5`), Controller.as:245-278 (cutscene ids 0-6), Enemy.as:66 (hard music every 5th night) — handled in Stage 3.
**Acceptance:** source-diff gate = exactly the table above; Classic oracle (section 6) passes: identical wave tables, names, caps and enemy stats for nights 1-30; runtime: Classic new game, Continue, night 12 -> 13 dawn, and the Stage 0c finale produce the original's trace sequences.
**Rollback:** remove patch `02_finalnight`.

### Stage 3 — Extended skeleton: nights 31-60, night-30 milestone, finale at 60, loader hardening (L)
**Purpose:** a playable, safe 60-night campaign with placeholder difficulty, built only from existing enemy ids 0-10 and existing screens.
**Exact code and design (R unless noted):**
- `Nights.as`: add `buildExtended(level)` producing rows 31-60 for `waveArray`/`waveAmount` from band templates (Stage 4 defines the content; Stage 3 uses a simple placeholder derived from nights 21-29 patterns) and extend `nightNameList` (59 entries needed; today 29 names for nights 2-30, Nights.as:13). `Level.create` already calls `Nights.addNights(this)` (Level.as:2009); call the extension from there only when extended.
- `Level.as:313` guard the name lookup (`nightNameList[nightCount-2]` is `undefined` past the list).
- **Cutscene trap:** `Level.as:2711-2715` sets a cutscene when `nightCount % 5 == 0`; `2571-2575` plays `addCutscene(nightCount/5)`; `Controller.addCutscene` knows ids 0-6 only (Controller.as:245-278) and id 6 is the **win cutscene** whose `removeCutscene(6)` destroys the level and returns to the menu (Controller.as:179-212). In Extended: cutscenes only for nights 5-25; at night 30 and later 5th nights go straight to `addDay()`; id >= 7 must never reach `addCutscene`; the win cutscene is reached only through the helicopter at `finalNight`.
- Night 30 milestone: a normal-cadence night with an event (Stage 5) and the usual checkpoint at dawn; **no countdown/helicopter** because Level.as:340/2695/2781 now compare with 60.
- Finale at 60: reuse the existing finale machinery unchanged (`addDeadly30Timer`, `updateNight30`, `addCopter`, `Helicopter.checkSoldiers`). `updateNight30` indexes `waveArray[wave]`/`waveAmount[wave]` with `wave=59`, so row 59 must exist (it does after `buildExtended`). The duplicated `==9000` branch (Level.as:2886-2893, R) is left as is (Classic fidelity); the finale table is reused as is.
- **Loader hardening (crash-class only):** Level.as:1805-1811 indexes sections 0-5 of `split("|")` without checks (the `if(saveString)` guard at 1793 does not prevent a crash when the string is empty); give each section a safe default; clamp `nightCount` to `[1, finalNight-1]`; if a loaded night equals `finalNight`, treat it as the previous dawn (otherwise `deadly30Timer` is null and Level.as:2802 throws every 31 frames).
- Off-by-one `Tracker.nightsLasted = wave` (Level.as:1828): **Extended only** (D4).
**Acceptance (all with disposable slot-2 saves made by the byte-patch recipe):**
- Classic oracle still passes; Classic finale still ends at 30 exactly as in Stage 0c.
- Extended: night 29 -> 30 -> 31 transition (no win cutscene at 30; checkpoint at dawn 30 and 31; save/load at 29, 30, 31, 59); night 60 finale reaches the helicopter and win flow; best-nights value is written to slot 2 only; menu shows Extended text.
- A hostile-save test: empty, truncated and over-range strings in the slot-2 file load without an exception (clamped), 0 errors.
**Rollback:** remove patch `03_extended`.

### Stage 4 — Difficulty progression 31-60 (L)
**Purpose:** a gradual, data-driven curve; difficulty from combinations, caps and events, not from stat inflation.
**Exact code:**
- Stat scaling sites (R): the per-type `setProperties` formulas reference `Controller.level.nightCount` directly in GreenZombie.as:13-17, BlueZombie.as:13-17, RedZombie.as:13-17, WhiteZombie.as:13-17, Fatty.as:226-241 (two types), Dog.as:79-91 (two types), Bird.as:185-188 (35 occurrences); Tank/RedMan use fixed values (A). Add `Level.statNight()` returning `nightCount` for `nightCount <= 30` and a compressed value beyond (proposal: `30 + 0.5*(n-30)`); replace the factor in those lines. In Classic `statNight()==nightCount` always.
- Proposed effect (linear, uncapped today vs proposal at night 60; speed in px/frame; tunable): Green HP 450 -> 375, Blue speed 5.6 -> 4.6, Red HP 520 -> 460, White HP 565 -> 505 and damage 32 -> 29. In addition cap each type's speed at 1.25x its night-30 value.
- Roaming/day spawns: Level.as:1128 adds `int(nightCount/4)` to the spawn count without a cap (at night 60: +15); use `statNight()` there and clamp; Room.as:108 (scrap count `+ nightCount`) and Room.as:343 (corpse decoration `random*(nightCount*10)`, up to ~600 at night 60, baked into a ground bitmap, so a performance and memory risk) likewise.
- Wave rows 31-60 from band templates in `Nights.as`, one new combination per band, concurrent cap kept within the original's observed range (original caps 3-20 on nights 1-29, 2-26 in the finale table):

| Band | Intent | Cap range | Composition idea (ids from the original list) |
|---|---|---|---|
| 31-35 | consolidation | 12-16 | nights 21-26 mixes with +15% queue length; first elite on 35 |
| 36-40 | pressure on fences | 14-18 | Fatty/RedFatty (4, 10) with heavy zombies (2, 3); elite on 40 |
| 41-45 | speed + flank | 16-20 | Dogs/BigDogs (6, 9) with Birds (7); elite on 45 |
| 46-50 | giants | 16-22 | Tank (5) groups with zombie escorts; elite on 50 |
| 51-55 | mixed arms | 18-24 | all families, short windows; elite on 55 |
| 56-59 | final approach | 20-26 | highest variety; no new type |
These are **starting values to tune**, not final balance.
- Resource balance: the dawn bonus is `kills*7` (Level.as:1770); add a bounded late-game stipend so ammo/repair never become impossible (no softlock), without new resource types.
**Acceptance:** Classic oracle unchanged; for n = 31..60 each stat is within its cap table; a spawn-load test at the highest cap records frame behavior (with the Ruffle caveat); a table check that no wave row contains an id outside 0-10 until Stage 10/11.
**Rollback:** remove patch `04_difficulty`.

### Stage 5 — Special-night events (M)
**Purpose:** variety on selected nights without new art.
**Design (deterministic, derived from the night number only):** `Nights.eventFor(n)` table; announced through the existing night intro text (`hud.playIntroText`, Level.as:313 R). Initial set, all built from existing mechanisms:
- **Horde:** scale `waveAmount[wave]` (cap) for the night; clamp to 26.
- **Elite:** insert Tank (5) / RedMan (8) into the spawn queue at fixed offsets (queue is consumed from the head by `updateNightEnemies`, Level.as:586-630 R).
- **Supply drop:** at dawn, drop extra scrap/ammo pickups with the existing `addScrapMetal`/`addAmmo`.
- **Blackout:** reuse the existing light system (`lightsOn`, `Light`, generator death path in SafeRoom.as:188-200 R); requires a spike to confirm a night can run with lights forced off without breaking the lighting bitmap (ASSUMPTION).
Rare by design (about 1 night in 3, never two in a row); event state needs no saving because it is a pure function of the night.
**Acceptance:** each event triggers only on its listed nights, once, never in Classic; save at dawn before an event night and reload shows the same event; no event on night 60's finale logic.
**Rollback:** remove patch `05_events`.

### Stage 6 — Box/turret repair and persistence (L)
**Purpose:** repair, visible health, and HP that survives room changes and saves.
**Evidence (R):** Box has a public `HP` (1000) and no health bar (Box.as:18, 67-79); Turret has `startHP`/`HP` and no repair (Turret.as:65, 84-104, 114-136); the only repair is the fence's hold-E logic (SafeRoom.as:505-612: +10 HP x fence level per tick, 3 ticks/frame by day and 1 at night, 1 scrap per 10 ticks, status icon text "Repair Fence"). Leaving the base and returning rebuilds objects through `Room.addSavedObjects` (Room.as:134-156), which calls `addBox`/`addTurret` with only x, y, scale, type, so **HP and ammo are lost on every room change** (this also makes re-entry a free full repair). The checkpoint saves only box x,y and turret x,y,scale,type (Level.as:1655-1699) and the loader recreates them at full HP (Level.as:1864-1881); **fence HP is also not saved**: `loadCheckPoint` calls `upgradeFences()` (Level.as:1836; SafeRoom.as:157-186), which resets both fences to full.
**Design (Extended only):** (1) carry HP through `Room.addSavedObjects` and add an HP parameter to `addBox`/`addTurret`; (2) new save section for per-object HP and per-fence HP (Stage 1/3 section parser); (3) hold-E repair next to a box/turret mirroring the fence logic, cost proportional to missing HP, day-heavy and night-limited, disabled while an enemy is within a set distance; (4) a programmatic health bar (graphics drawn in code; no new art); (5) fix of the re-entry full-heal for Extended.
**Acceptance:** damage a box and a turret, leave and re-enter the base: HP unchanged; save at dawn and reload: HP restored; repair never exceeds max, costs the expected scrap, and fails with feedback when scrap is 0; Classic unchanged (re-entry still heals: baseline quirk preserved).
**Rollback:** remove patch `06_repair`.

### Stage 7 — Reinforced base and limited character upgrades (M)
**Evidence (R):** fence upgrade levels 1-5 with HP table in `SafeRoom.upgradeFences` (SafeRoom.as:157-186) and costs hard-coded in two places (Shop.as:349-383 when the shop opens; 573-645 on purchase); fence visuals are driven by `leftFence.gfx.gotoAndStop(Controller.fenceUpgrade)` (SafeRoom.as:231-232) over what looks like a 5-frame clip; armor divides damage by 1/2/4 (Soldier.as:254-262); boots speeds (A: Soldier.as setMoveSpeed ~1469-1483).
**Design (Extended only):** raise the maximum fence level (e.g. to 8) with new HP rows and costs in the `Nights` config; reuse frame 5 for levels above 5 (clamp, verified in Stage 0b); armor level 4 (divisor 6) as the one "limited character upgrade"; reinforced box = box HP scaled by fence level (no new shop item, so no new clip); shop gating (`mouseEnabled=false` at max, Shop.as:361-365, 575-580) switched to a max-level variable.
**Acceptance:** purchase paths charge the table costs; levels persist through save/load; armor 4 reduces damage as specified; max level disables the button; Classic's maxima remain 5/3/3.
**Rollback:** remove patch `07_base`.

### Stage 8 — Weapon upgrades (L)
**Evidence (R/A):** all stats live on `Weapon` objects (magazine, damage, rate, reloadTime) created for every level in `createShopItems` (Level.as:1508-1519 R); firing reads them (Soldier updateShooting, A); damage for the two projectile weapons also comes from `gunSelected.damage` (Level.as:1314, 1331, A); pierce counts, pellet patterns and range are hard-coded by weapon name (Soldier.as:582-678, A) and are left alone, so every weapon keeps its identity. Shop buttons are named clips matched by name (`getShopItem(btn.name)`, Shop.as:489; `mOverItem` shows name and cost in two text fields, Shop.as:138-152 R), so **no new shop button can be added without art**.
**Design (Extended only):** per-weapon upgrade level (max 3) in a new static; effects applied to the live `Weapon` object after `createShopItems` and after `loadCheckPoint`; UI without new clips: clicking an already-equipped owned weapon (currently a no-op re-equip, Shop.as:512-535 R) attempts an upgrade, and `mOverItem` text shows "level a/b, upgrade cost" in the existing fields; caps keep total DPS gain within about +35% and never touch rate for machine guns; costs from a table (no free upgrades for cost-0 starters).
**Acceptance:** each level changes exactly the specified fields; persists across save/load and room changes; the magazine/reserve bookkeeping stays one-to-one; Classic weapons unchanged.
**Rollback:** remove patch `08_upgrades`.

### Stage 9 — XP, rewards, unlocks (M)
**Evidence (R):** level-ups come from `HUD.updateExp` (HUD.as:334-361): `expList` (12 thresholds, A: HUD.as:68), cap at level 13 (HUD.as:341), a blueprint reward (`addBluePrint`, Level.as:1465-1500) drawn from a pool that **runs out** (`getNotFoundItem` returns 0 when everything is found, Level.as:1182-1248), and level saved in slot key `lvl` (HUD.as:357-358). `Controller.exp` is not saved (reset at new game, Level.as:2026). Level feeds the start scrap `(lvl-1)*100` (Level.as:2030) and the laser sight at level > 5 (Level.as:2045).
**Design (Extended only, slot 2):** extend thresholds and the cap (e.g. to level 20) in the config; levels above 13 reward scrap/medkits (no blueprints); persist `exp`; unlock the existing 12 weapons by night/level rather than only by random blueprints; no new weapons until Stage 11. Reuse the existing 100-frame EXP bar.
**Acceptance:** thresholds and rewards match the table; `lvl` and `exp` persist in slot 2 only; Classic's `lvl` untouched (hash check); farming cannot out-earn surviving (reward per night bounded in the config).
**Rollback:** remove patch `09_xp`.

### Stage 10 — Additional zombie behaviors and variants (L)
**Design:** behaviors as parameters on existing classes: fence-pressure multiplier (existing fence hits: Zombie.as:373-407 R; Fatty/Dog/Tank multipliers A), a visible wind-up reusing the existing Tank attack window, retreat/approach for dogs. Variants as tint plus stats on existing symbols: a variant field set in `Level.addEnemy` (Level.as:2489-2544 R) after `create`, with new ids >= 11 added to `addEnemy`, `Nights`, and the unlock lists (Level.as:1778-1780, 1900-1902, 3560-3588 R). The base-class route is preferred because it needs no new class (if Stage 0b proves new classes are possible, subclassing is an option).
**Acceptance:** each behavior has a visible tell and a counter-strategy documented; no change to ids 0-10 in Classic; performance at the highest cap not worse than Stage 4 beyond a recorded threshold; interactions between types tested pairwise.
**Rollback:** remove patch `10_behaviors`.

### Stage 11 — Art/sound-dependent content (GATED on Stage 0b)
New weapons (needs a shop clip, blueprint frame, arm/gun frames, sounds; also `Shop.as:416`'s index alignment, A), new enemy sprites, traps, mode-select art. **Not started unless Stage 0b proves a safe workflow.** If not possible, these items are cut or replaced by tint variants.

### Stage 12 — Balance, performance, full regression, final verification (L)
Mirror of `newplan.md` section 21 Stage 5: Classic oracle on the final build, full manual matrix, performance at the heaviest nights, final-night and ending in both modes, restart/replay state check (statics `Tracker`, `Controller.cutscenesPlayed`, `American.meet*` are never reset, A), release-build gate (no hooks, hashes, manifest).

---

## 4. Save compatibility and migration strategy

| Topic | Strategy |
|---|---|
| Classic slot (`so_deadly301`) | Untouched by Extended code. Format unchanged. Only crash-class loader hardening that cannot change a valid save's behavior. |
| Extended slot (`so_deadly302`) | Same sections 0-5 so the existing loader path is reused, plus appended extension sections (version/mode, HP of boxes/turrets/fences, weapon upgrade levels, XP/exp). Sections are appended, never reordered; a missing section means a safe default. |
| Legacy saves | Never converted automatically. Optional later: an explicit, confirmed import that **copies** the Classic checkpoint into slot 2 after the Classic file has been backed up by hash outside the game. The game itself never writes a backup into slot 1. |
| Display settings | Shared (slot 1, read/write exactly as today). |
| "Clear Save Data" | Unchanged: slot 1 only (no accidental Extended wipe). Extended clearing is a separate, later item. |
| Persistence in Ruffle | Writes only on graceful exit; tests must close the window normally; hard kill = data loss is an existing limitation to document, not fix. |
| Path-keyed storage | All save tests use the fixed run path `game\run\deadly30.swf` and a fresh disposable save dir per run; to compare original and patched builds under the same save, copy each build to that same path in turn. |
| Forward compatibility | The version section lets a later build ignore unknown sections; unknown trailing sections are already ignored by the current loader. |

Test saves are to be generated from a disposable copy of the real-save backup by the byte-patch recipe (3.0.1, untested until Stage 0c) or, failing that, by the test-flavor hook (3.0.2); the real save and its backup are never opened for writing.

---

## 5. Hard-coded night-30 and related items (consolidated list, R)
Level.as:313, 340, 984, 1172, 1888-1899, 2571/2575, 2695, 2711, 2781 (and `deadly30*` state, 2778-2906); HUD.as:122; MainTimeline.as:163-170; Medals.as:39 (text); Helicopter.as:102 (win cutscene), 183; Controller.as:170-179 and 245-278 (cutscene ids); Nights.as:11-73 (30 rows, 29 names); Level.as:1828 (`nightsLasted` off-by-one on load); Enemy.as:66 (music every 5th night); Room.as:277, 281 (squad recruit gates at nights 3 and 6, harmless); existing bugs to log, not fix in Classic: Level.as:1143 (wrong array length for super-rare spawns, R), Level.as:2886/2890 (duplicate `==9000`, R).

---

## 6. Testing and regression matrix

### 6.1 Classic regression plan (original vs rebuilt, nights 1-30)
1. **Static oracle:** from the original export and from each candidate export, extract every wave row, cap, night name and enemy stat formula; evaluate stats for n = 1..30 and every type; require equality. Run after every stage.
2. **Source-diff gate:** only the hunks listed for the stage.
3. **Tag gate:** only `DoABC2` differs from the original SWF.
4. **Runtime oracle (test flavor, 3.0.2):** same hook patch on baseline and candidate; compare per-night wave digests, spawn stats and trace order for a set of nights spanning every band (1, 2, 5, 10, 15, 20, 25, 29) and the finale.
5. **Behavioral smoke on the fixed path:** new game -> scripted death; Continue from the real-save copy; night 12 -> 13 dawn with checkpoint write; Classic finale (Stage 0c).
6. **Error budget:** 0 runtime errors with `--dummy-external-interface` (the one unexplained 6-error burst is tracked; any recurrence is investigated, not ignored).

### 6.2 Per-stage matrix (R = required, - = not applicable)

| Test | 0a | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Tag gate | R | R | R | R | R | R | R | R | R | R | R | R |
| Source-diff gate | R | R | R | R | R | R | R | R | R | R | R | R |
| Classic static oracle | R | R | R | R | R | R | R | R | R | R | R | R |
| Classic runtime oracle | - | R | R | R | R | R | R | R | R | R | R | R |
| Classic finale baseline | - | R | R | R | - | - | - | - | - | - | - | R |
| Slot isolation (hash of slot-1 file) | - | R | R | R | - | - | R | R | R | R | - | R |
| Extended 29->30->31 and 59->60 | - | - | - | R | R | R | - | - | - | - | R | R |
| Hostile-save loads | - | - | - | R | - | - | R | R | R | R | - | R |
| Save/load round trip (graceful close, restart) | - | R | - | R | - | R | R | R | R | R | - | R |
| Shop/purchase paths | - | - | - | - | - | - | - | R | R | R | - | R |
| Performance at highest cap | - | - | - | - | R | R | - | - | - | - | R | R |
| Restart/replay state check | - | R | - | R | - | - | - | - | - | - | - | R |

### 6.3 Run conditions (verified configuration)
`ruffle.exe --graphics dx12 --storage memory --load-behavior blocking --dummy-external-interface` for non-save tests; `--storage disk --save-directory <fresh disposable dir>` only for save tests, with a **graceful close** and the **fixed SWF path**; isolated `--config/--cache-directory`; keyboard by window messages, mouse by the guarded real cursor; assertions on traces, counts and ranges (no exact random values, no wall-clock thresholds).

### 6.4 Result reporting
Every stage report lists Passed / Failed / Skipped / Unverified, the exact command lines, build hashes, and the before/after hashes of the original SWF, its backup and the real save.

---

## 7. Risks, mitigations and rollback

| Risk | Impact | Mitigation | Rollback |
|---|---|---|---|
| FFDec's experimental compiler mis-compiles a patched class | Silent behavior change | Source-diff gate on every stage; runtime oracle; small patches | Rebuild from previous patch set |
| `Level.as` (3,597 lines) edited via recompiles | Collateral breakage | One concern per stage; gate each hunk; keep edits minimal | Drop the stage patch |
| Classic accidentally altered | Violates the core rule | Classic oracle on every stage; Extended gating via one flag; slot hash checks | Drop the stage patch |
| Cutscene 6 fires at night 30 in Extended | Campaign ends at the midpoint | Explicit guard (Stage 3); test 25/30/35 | Drop stage 3 patch |
| Loader crashes on bad/foreign data | Lost session | Hardening and hostile-save tests (Stage 3) | Drop patch |
| Stats unbounded past night 30 | Impossible late game | `statNight()` compression and caps (Stage 4) | Drop stage 4 patch |
| Effective fps below 30 under Ruffle | Misleading timing results | Trace-order assertions; separate performance measurement | n/a |
| Random events break determinism | Flaky tests | Structural assertions; events derived from night number only | n/a |
| New art/sounds impossible | Features cut | Stage 0b decides; interim programmatic UI; tint variants | n/a |
| Hard-kill data loss, path-keyed saves | False "lost save" reports | Graceful-close rule; fixed path; document for release | n/a |
| Proprietary code leaking via git | Legal/ethical | Patches and sources stay in the lab, never committed; repo holds plans/reports only | n/a |
| Unseeded randomness and Ruffle quirks hide bugs | Escapes to release | Final verification on more than one Ruffle build, and by you in a licensed Flash runtime before release (decision D5) | n/a |

**Rollback strategy:** every stage ships as `patches\NN_<stage>` + `builds\NN_<stage>_*.swf` + manifest. Going back one stage is "rebuild from the previous patch set and re-run its gates". The baseline export, original SWF, backups and real-save backup are read-only and hash-verified before and after each session.

---

## 8. Recommended first implementation task (small and independently verifiable)

**Stage 0a: the reproducible build and gate harness with an empty patch set.** No game source is changed and no game behavior can change. It is verifiable on its own: the harness must reproduce the Stage 3/3b facts (one differing tag; only the known compiler wrappers in the source diff; the smoke trace sequence equal to the original's; recorded hashes) and must fail loudly when given a deliberately wrong input (a tampered hunk, an extra differing tag, a hook marker in a production build). Everything else in this plan depends on those gates.

The first **game-code** change after that, and the smallest meaningful one, is Stage 1's first slice: add `extended`/`saveSlot` to `GameController` and route `getSharedObject` through the slot while every call site still resolves to slot 1. Its acceptance is that Classic behavior and the slot-1 file are identical to the baseline.

---

## 9. What remains open (nothing here is assumed to work)
Stage 0b's capability questions; the Classic finale baseline (Stage 0c); the shop UI; nights after 13; the night-60 ending; whether a night can run with lights forced off; whether fence levels above 5 render acceptably with a clamped frame; end-user deployment (D5).

**Approval requested:** approve the plan (or the decisions in section 0) before any implementation. No stage will start without your explicit go-ahead.
