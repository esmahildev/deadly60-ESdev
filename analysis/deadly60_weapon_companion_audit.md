# Deadly 60 — Weapon progression and companion system: audit and proposals

Date: 2026-10-18. Branch at time of writing: `stage-7/features-record`. **Audit and planning only.** No game code, patch, build, save or test was touched or run for this report; nothing is committed. **Deadly 60 is not runtime-verified**: every "implemented" below means "present in the lab patches/exports and statically checked", never "seen working in the game".

Evidence tags: **[D]** = read in a project document (path given). **[C-orig]** = read in the decompiled original source `analysis/as3_scripts/com/iggyzuk/deadly30/` (local only, git-ignored per `deadly60_stage0_design_record.md:191`). **[C-lab]** = read in the exported scripts of the newest lab build `FULL_UNLOCK_TEST_15` (`D:\Projects\deadly30_lab\game\d60_rollout\stage21\gate_results\FULL_UNLOCK_TEST_15\export\scripts\...`, lab only, not in Git). **[U]** = unverified.

---

## 1. Current-state findings

### 1.1 Which build contains what
- `S7_FEATURES_2` / `S8_FEATURES_2` / `S9_SAFE_2`: upgrades, perks, Mk II, defences, variants. No new weapons. [D] `deadly60_stage7_features_record.md`, `..._stage8_...`, `..._stage9_record.md`.
- `S9_SHOP_2` and the `FULL_UNLOCK_TEST_*` line: add the shop weapons (ids 31–34). The test line additionally has the Engineer (4th survivor), exclusive weapon loadouts, new art and SFX. [D] `deadly60_stage10_test_build.md`. That the Engineer is absent from S7–S9 is inferred from the stage order in these records, not re-diffed. [U]
- The test builds also switch several gates off (`d60Test`: day-30 and day-40 locks, low-night variants). [D] `deadly60_stage10_test_build.md` "Test state"; [C-lab] `Controller.as:1663`, Mk II check near `:1869`.

### 1.2 Weapons in the original
Twelve weapons in three ammo categories (type 1 rifle, 2 machine gun, 3 shotgun); ids 1–3 start owned, the rest are bought or found as blueprints. [C-orig] `Level.as:1508-1519` (`createShopItems`, `Controller.addWeapon(...)`). Shop items besides weapons: Big Turret, Mini Turret, Box, Fence upgrade, Armor upgrade, Boots upgrade, three ammo types (25 scrap each), Health Pack (400). [C-orig] `Level.as:1520-1529`. `Weapon` originally holds only rate, reloadTime, damage, bullets, magazine, type. [C-orig] `Weapon.as`.

### 1.3 Weapon upgrades (damage / reload / magazine, levels 1–3)
- Implemented in lab patches: hotkeys U / I / O at the base in daytime from day 30; damage +10 %/level, reload time −10 %/level (floor 12 frames), magazine +20 %/level; max 3; cost `400 × (level+1) × weapon art tier`; press twice to buy. [D] `deadly60_stage7_features_record.md` §1 row 3. [C-lab] `Controller.as:437-441` (`d60WeaponCost`), `:1663` (day-30 gate).
- Accuracy / spread upgrade and ammo-efficiency upgrade, both listed as options in `newplan.md:187-191`: **not implemented**; no document claims them.

### 1.4 Mk II
- Implemented: key L at the base in daytime; needs damage/reload/magazine all at 3 and `nightCount >= 40`; cost `5000 × artTier`; rifles +25 % damage, machine guns +20 % fire rate −15 % reload +5 % damage, shotguns +20 % damage +2 shells; the name is deliberately not changed (the Laser Gun and Golden AK behaviour compare names). [D] `deadly60_stage8_features_record.md` §2. [C-lab] `Controller.as` Mk II block (~`:1865-1895`) (`packed >> 6 & 1`, `nightCount < 40 && !d60Test`, `cost = 5000 * w.artTier`).
- **Ambiguity [U]:** "Day 40" is implemented as `nightCount >= 40`. The save field and banner use "DAWN n" for the counter, so whether that equals "after surviving night 40" or "during night 40" was never confirmed in play.

### 1.5 Experimental shop weapons
Shop slots are not blocked (the Stage 7/8 "blocked" verdict was corrected in `deadly60_stage9_record.md` §1: extra icons are additive PlaceObject tags in sprite 589). Stage 9 added four weapons (Service Carbine, Sawed-Off, Heavy Rifle, Rapid Repeater). Stage 11 replaced them with Wave Cannon (31), Gravity Gauntlet (32), Flamethrower (33), Tesla Rifle (34), each with its own effect, buffed ~3× in stage 14; ids 31/32 are Engineer-exclusive, 33 Russian-exclusive, 34 German-exclusive. [D] `deadly60_stage10_test_build.md` stage 11–19 notes. [C-lab] `Weapon` fields `d60Owner`, `d60Fx`, `artTier`, `fxRef` (names from the lab patches). Numbers are first-pass; none was played. Note: `deadly60_stage9_record.md` §3 table is **stale** (describes the superseded four).

### 1.6 Perks
Endurance (J, −6 % damage taken per level) and Scavenger (K, +10 % dawn scrap per level), max 3, cost `1500 × (level+1)`. [D] `deadly60_stage7_features_record.md` row 13. [C-lab] `Controller.as:443-446`. Saved in the additive `d60` SharedObject key; ignored unless night and scrap match `saveString`. [D] `deadly60_stage8_features_record.md` §3.

### 1.7 Crafting, parts, ammunition, explosives
- **Crafting / parts:** no document plans it and no code has it. The only currencies are scrap metal, health packs and three ammo counts. [C-orig] `Controller.scrapMetal`, `healthPacks`, `ammo[3]`. `newplan.md:83, 541, 569` explicitly warn against crafting screens.
- **Ammunition:** exists in the original (buy per category; `Level.as:1840`-region restore of `ammo`). No ammo-efficiency feature. Test builds give 999 ammo. [D] `deadly60_stage10_test_build.md`.
- **Special explosives (grenades, dynamite, etc.):** none in the original (a text search of all game classes found no grenade/molotov/dynamite/C4). Closest existing: **Mines** (key T, 250 scrap, up to 4, base room only, not saved) and the Bloater zombie's death explosion. [D] `deadly60_stage8_features_record.md` §2. A player-thrown explosive is **missing**.

### 1.8 Defensive equipment / shields
Original: fence upgrade (levels 1–5), armor upgrade, turrets, boxes. Added in lab: spiked fence (night 31+), Plating (G, 3 levels), Barbed Wire (B, 3 levels), Mines (T). [D] stage 7/8 records. Engineer-only **Energy Shield** (50 points, regenerates after 5 s without damage) exists only in the Engineer test-build lineage. [D] `deadly60_stage10_test_build.md` "stages 14–15". No general-purpose shield item exists.

### 1.9 Squad system already in the original (important for Section 2)
The original is **not strictly solo**:
- A new run starts with one soldier (American); up to two more survivors (Russian, German) are met as "lost soldiers" and join. [C-orig] `Level.as:2038-2043` (start), `:1912-1980` (meeting, `hud.addCrewIcon`, medal 14 "Union" at 3 soldiers).
- The player controls one soldier; the others are **AI followers** that walk to a random distance (75–275 px) behind the controlled soldier, jump-leap when far, shoot automatically, and mill about in the safe room when the player is inside. [C-orig] `Soldier.as:1152-1232`.
- Keys 1/2/3 switch the controlled soldier (`Level.as:2725-2735`, `switchSoldier` `:3430-3474`); `Input.NUMBER_4` exists but is unused in the original.
- F heals the **whole squad** with one health pack when anyone is ≤ 70 HP. [C-orig] `Soldier.as:1008-1035`.
- A dead soldier is removed from `soldierArray` for good; with 0 left (or 1 left plus an unmet lost soldier) game over starts. [C-orig] `Level.as:2289-2336`. Checkpoints write the survivor skin list and Continue re-adds them. [C-orig] `Level.as:1832-1842`; [D] `deadly60_b6_phase1_design.md:57`.
- XP/level is global (`Controller.exp`, `Controller.lvl`), not per soldier; ammo, scrap, health packs and per-category equipped weapons are shared. [C-orig]; [D] stage 10 notes (weapon equipped globally per type).
- The lab Engineer reuses exactly this system (4th survivor, key 4, joins at night 31, AI-follows like the others). [C-lab] `Controller.as:899-1066` (`d60EngineerExists/Alive/CheckEngineer/Swap`), `Level.as:314`.

---

## 2. Companion and team audit

| Proposed element | Status | Evidence / note |
|---|---|---|
| Warrior companion (melee/tank) | **Missing** | no document or code; zombie targeting rules for a taunting unit not examined [U] |
| Medic companion | **Missing** | healing is a global health-pack action (F), `Soldier.as:1008-1035`; no healer unit |
| Engineer companion | **Partial / different** | an Engineer exists as a *playable 4th survivor* with shield, +20 % scrap, +25 % sci-fi weapon damage; it does **not** repair, build or operate turrets. [D] stage 10 notes; [C-lab] `Controller.as:1071` (+25 % for `skin == 3`) |
| Long-range marksman companion | **Missing** | closest: Heavy Rifle-style weapon ideas, but no unit role |
| Follow command | **Exists implicitly** | AI soldiers always follow the controlled soldier, `Soldier.as:1152-1219`; not player-selectable |
| Defend-base / hold-position / attack commands | **Missing** | the safe-room idle behaviour (`:1220-1232`) happens only when the player is inside the base and cannot be commanded |
| Companion EXP / levels / skill trees | **Missing** | only global player level; per-soldier state does not exist. A skill tree conflicts with `newplan.md:209` and `:525` |
| Limit on active companions | **Implicit** | original max 3 (medal at 3, HUD crew icons, keys 1–3); lab Engineer makes 4. Behaviour of HUD/keys/`speach` pairs beyond 3 skins is **[U]**; meeting speeches exist only for skins 0–2 (`Level.as:1926-1949`) |
| Survival, recovery, availability between nights | **Original rule: permanent death**, restored only from the checkpoint list | `Level.as:2289-2336`, `:1832-1842`. Whether HP is refilled at dawn for soldiers was not confirmed [U] |

### Conflicts with the original gameplay style
1. **Not a true solo loop**, so "companions" are an extension of a mechanic the game already has; but those soldiers are story rescues (found, not bought), limited to two, and are the player's own avatars (switchable). Pure non-controllable support units are a new concept.
2. **Balance:** nights 1–60 were tuned (and nights 56/58 rows approved) for at most 3 shooters; the lab already makes it 4. Each extra AI shooter adds damage without input skill. [D] `deadly60_b6_phase1_design.md`, stage 7–8 records.
3. **Controls:** original keys used: A D W S E F R X Z P M Shift, arrows, 1/2/3, 6, Esc, Enter [C-orig]; lab adds U I O J K H G B T L Q 4 [D]. Command keys would compete for free letters (candidates C, V, N, Y; **[U]** — requires a full key scan including the Zinc wrapper and menu code).
4. **`newplan.md` rules:** Feature 14 "avoid excessive UI and menu complexity / crafting / inventory" (`:527-549`), Feature 13 "do not turn the game into a complicated RPG" (`:525`), "no complicated skill tree" (`:209`).
5. **Content location:** `single_mode_roadmap.md` ND-2 asks where new content may appear; the owner requires the original curve and balance to stay. Companions in nights 1–29 would change that curve. The owner's answer to ND-2 is not recorded in the files I read [U].
6. **Technical:** the soldier rig has skin frames 0–3 (skin 3 added for the Engineer through additive SWF tag patches); more skins need more art patches, HUD face frames and save-list handling; saves holding unknown ids crash earlier loaders (stated for weapon ids in `deadly60_stage9_record.md` §3, expected to apply to survivor ids too [U]).

---

## 3. Feature-by-feature status

| Feature | Status | Evidence |
|---|---|---|
| Existing weapons / categories | **Planned+implemented (original)** | §1.2 |
| New shop weapons (ids 31–34) | **Implemented in lab (test line), not played** | §1.5; stale rows in stage 7/9 docs |
| Damage / reload / magazine upgrades, levels 1–3 | **Implemented, not played** | §1.3 |
| Accuracy upgrade | **Missing** | `newplan.md:187` |
| Fire-rate upgrade | **Missing** (only via Mk II for machine guns) | `deadly60_stage8_features_record.md` |
| Ammo-efficiency upgrade | **Missing** | `newplan.md:191` |
| Mk II, day-40 condition | **Implemented, not played; "day" meaning ambiguous** | §1.4 |
| Experimental shop weapons | **Implemented in lab only** | §1.5 |
| Endurance, Scavenger perks | **Implemented, not played** | §1.6 |
| Weapon crafting / parts | **Missing; conflicts with newplan Feature 14** | §1.7 |
| Ammo purchase | **Original** | §1.7 |
| Special explosives (thrown) | **Missing**; Mines + Bloater exist | §1.7 |
| Shields | **Partial**: Engineer-only shield (test line), armor upgrade original | §1.8 |
| Warrior / Medic / Marksman companions | **Missing** | §2 |
| Engineer companion | **Partial / different design** | §2 |
| Follow command | **Implicit only** | §2 |
| Hold / defend / attack commands | **Missing** | §2 |
| Companion EXP / levels / trees | **Missing; trees conflict** | §2 |
| Active-companion limit | **Implicit (3, lab 4)** | §2 |
| Companion survival / recovery | **Original permanent death** | §2 |

---

## 4. Missing requirements (need an owner decision before any design is final)
1. What does "Engineer companion" mean: keep the current playable Engineer, or an AI support role (repair, turret help)?
2. Is "Day 40" the counter value at dawn 40 or the start of night 40?
3. Should weapon upgrades unlock earlier than day 30?
4. Accuracy, fire-rate and ammo-efficiency upgrades: wanted or not (listed in `newplan.md` as options only)?
5. Special explosives: thrown item, or keep Mines only?
6. Are companions allowed before night 31?
7. Per-companion weapon loadouts vs. the global per-type loadout.
8. Recovery rule for fallen companions.
9. Whether the Engineer and exclusive weapons remain in the final game or only in the test line.

## 5. Conflicts with existing decisions
- `newplan.md:83, 541, 569` vs crafting/parts.
- `newplan.md:209, 525` vs skill trees / companion RPG levelling.
- `newplan.md:527-549` and ND-2 vs anything that makes nights 1–29 easier.
- Stage 7 table row 2 ("Additional weapons: Blocked") vs Stage 9 correction; Stage 9 §3 weapon list vs Stage 11 replacement. These docs conflict with each other and should be read with the dates.
- Stage 8 decision: Mk II must not rename weapons; a "Mk II"/crafted-variant design must keep that constraint.
- Original loop: survivors are rescued by story events; a purchasable or recruitable companion is a rule change.
- Weapon exclusivity (stage 18) vs original "equip per ammo type for the whole squad".

---

## 6. Proposal A — Minimal extension (preserve the original game)

**What it is.** Keep the three-category arsenal and existing squad. Finish and tune the upgrades that already exist; add at most one optional lever if wanted.

**Player-facing benefits.** Clear reason to spend scrap in nights 31–60 (levels 1–3, Mk II at day 40, two perks), no new menus, no new units, same controls.

**Contents.**
1. Treat §1.3–1.6 as the baseline; no new systems.
2. Optional A1: one "Ammo Efficiency" level (e.g. every Nth shot free) *or* an "Accuracy" level, not both; only if a play test shows the current upgrades are too flat.
3. Explosives: do not add a thrown item; rely on Mines and the Bloater.
4. Crafting/parts: none; scrap stays the single currency.
5. Shields: keep armor upgrade; decide whether the Engineer's shield ships.

**Risks.** Design: upgrades stacking with Mk II and perks could trivialise nights 56–60 (rows tuned before the upgrade system). Technical: all changes live in `Controller`/`Weapon`/`Level` patches (additive `d60` save key); any new level needs another bit in the packed weapon value (currently 6 bits + Mk II bit 6, values 0..127). [D] stage 8 §3.
**Dependencies.** Shop hotkey flow, `d60` save key, `Weapon.artTier`.
**Balance problems.** Damage ×1.33 (3 levels) × Mk II ×1.25 × Engineer ×1.25 on new weapons; Endurance −18 %; Plating −45 % fence damage. Combined effect is calculable but was never play-tested. [U]
**Phases.** (0) Owner answers §4 items 2–5. (1) Owner plays the current candidate and reports balance. (2) Tune numbers only. (3) Optional A1 as one small patch with its own build and review.
**Approval questions.** §4 items 2, 3, 4, 5, 9.

## 7. Proposal B — Optional companion expansion (later, off by default)

**What it is.** A separate, later expansion that does not change the default experience: reuse the existing survivor/follow system and add behaviours, not new genres.

**Design principles.** Default off or unlocked only by a late-game event (nights ≥ 31); total squad capped at 4 including the current Engineer; no skill trees; no per-companion currencies; death stays permanent unless the owner chooses otherwise.

**Possible contents (each independently approvable).**
1. **Commands** — the lowest-risk, art-free step: one key toggles follow ↔ hold-position for AI soldiers, using the existing `followTarget` logic (`Soldier.as:1152-1232`). Defend-base reuses the safe-room idle behaviour. "Attack" needs target logic the original lacks (risk).
2. **Roles on existing/new survivors** — Warrior (high HP, short range), Medic (slow heal-over-time that **consumes** health-pack stock or has a long cooldown so it does not replace F), Marksman (long range, low fire rate). Engineer: keep the current design or redefine (§4.1).
3. **Progression** — one linear 1–3 level per companion paid in scrap at the base (like perks), stored additively in the `d60` key; no tree.
4. **Recovery** — default: permanent death (original); option: paid revival at dawn (easier, needs explicit approval).

**Player-facing benefits.** Tactical choices between nights; more reasons to protect specific survivors.
**Risks.** Technical: each new skin needs rig frames, HUD face frames, meet/rescue dialogues and save handling (the Engineer took ~12 stages of patches and is still unplayed); HUD/keys assume ≤3 survivors (§2); AI shooters add DPS the wave tables do not account for. Design: AI Medic can silently remove the health-pack economy; a Warrior may break zombie targeting [U].
**Dependencies.** `Level.soldierArray`, `switchSoldier`, `d60` save key, Soldier rig, `Controller.d60EngineerSwap` style code.
**Balance problems.** More shooters = lower difficulty by default → a global enemy-scaling counter-adjustment or a hard squad cap is required, and the Night 56/58 approved rows must be re-validated.
**Phases.** (0) Approvals, no build. (1) Commands only, behind a flag, no art; owner play-test. (2) One role at a time (Marksman first: fewest rule changes), separate build and review each. (3) Optional progression and revival rules. Each phase is a separate branch/build; nothing merges without approval.
**Approval questions.** §4 items 1, 6, 7, 8, 9, plus: should companions be a toggle in a menu, a late-game unlock, or never shipped?

## 8. Recommended order of work
1. Owner decisions on §4 (no build).
2. **Runtime verification of what already exists** (owner plays: upgrades, Mk II, perks, new weapons, Engineer, banners/help text). This is the real blocker; every later design depends on it.
3. Fix document staleness (Stage 7 row 2, Stage 9 §3) as a doc-only change.
4. Proposal A tuning (+ optional A1).
5. Proposal B phase 1 (commands only) *only if* approved.
6. Proposal B roles, one at a time.

## 9. Explicitly unverified assumptions
- Nothing in Deadly 60 has been run in the game by the owner or by me; all "implemented" rows rest on code export, static gates and FFDec renders. [D] all stage records.
- The meaning of "Day 40" relative to `nightCount` (§1.4).
- Whether the Engineer exists in S7–S9 candidates (inferred from the stage order).
- Soldier HP behaviour at dawn; zombie targeting of non-controlled soldiers; HUD and key handling with a 4th/5th survivor; free key codes (§2).
- The balance effect of stacked upgrades/perks/Mk II/Engineer bonuses.
- Owner's answer to ND-2 (content location) is not in the files read.
- The decompiled originals and lab exports are local; this report cites them by path and line but they are not in Git.
- No line number in `Level.as`/`Soldier.as` was re-checked against the SWF itself; they come from the FFDec export used throughout the project.
