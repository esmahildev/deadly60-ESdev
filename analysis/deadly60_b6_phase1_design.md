# Deadly 60 — B6 synthetic saves: Phase 1 design (source inspection and planning only)

Date: 2026-10-17. Branch: `stage-5/b6-phase1-design`, created from `stage-5/followup` at 339b35588ee8116de111516041efcc6ed7ba74a3 (clean tree, remote head identical, hash verified). Not merged into `main`.

**Status: Deadly 60 is not runtime-verified.** Night 30 → 31, the Night 60 finale and save/Continue compatibility remain unobserved.

**What Phase 1 did and did not do.** It read game source and committed documents only. No real save, save metadata, backup, user save folder or save-related lab folder was opened, read, hashed, listed or copied. No `SAVE` folder, synthetic file, generator, launcher, test fixture or test-hook build exists or was created. No Ruffle, game or EXE wrapper was launched. No game code or SWF was touched. This report contains no binaries, logs or screenshots.

Labels: **[V]** read directly in the game source (static; not runtime-observed), **[D]** documented in an earlier committed report from a read made under earlier approval and not repeated now, **[A]** assumption, **[I]** inference, **[U]** unresolved.

---

## 1. Correction to an earlier inference (important)

Earlier in this project I inferred that after a Continue the first update would fire `nightOver` and re-run `addDay()`, which would restore `nightsLasted` to the uninterrupted value, so that a continued win would end at 60 (30 in the original). Reading the complete loader shows that this was **wrong**:

- The loader itself calls `addDay(false, true)` (Level.as:1882). With the second argument true, `addDay` skips the checkpoint write (2396) and skips `++Tracker.nightsLasted` (2432-2435) [V].
- `addDay(false, …)` calls `endDayTransition()` (2412-2417), which ends by setting `night = false` (Level.as:1782) [V].
- Later `addDay()` calls (the `nightOver` handler at the first update, or the end of a replayed cutscene) return at once because `night` is false (2392-2395) [V].

Consequences [V as code reading, not runtime-observed]:
1. After a load, **no checkpoint is rewritten and the counter is not incremented**; the day simply runs from `dayTimer = 0`. A loader trace is `LOADED STRING = …` then `DAY!`, with no `SAVE STRING`.
2. `Tracker.nightsLasted` is set to `nightCount − 1` at load (1828) and stays one lower than in an uninterrupted run for the rest of that run. **A win after Continue therefore ends with `nightsLasted` = final night − 1** (59 in the 60-night design, 29 in the original), and the menu then shows "You Lasted 59 Nights" (the original shows "You Lasted 29 Nights") instead of "Survivors have escaped!". Only an uninterrupted win reaches the exact threshold. This is the original's `nightsLasted` quirk, now confirmed from source, and it is preserved by the current thresholds (30 → 60).
3. The optional counter fix (the earlier "V1" idea, setting the counter at victory) is relevant again as a **user decision**: leaving it keeps parity with the original quirk; nothing was changed.
4. Cutscene nights: loading a night 5/10/15/20/25 checkpoint still replays that cutscene after 120 frames if it was not played earlier in the session (`nightOn` is set by `startGame`, the first update sets `nightOver` and starts the 120-frame timer). For nights 30+ the Stage 2 bound still prevents the invalid cutscene ids; the ordinary branch then calls `addDay()`, which returns at once. The Stage 2 and Stage 3 changes are therefore still correct and needed.
5. Earlier predictions that a Continue would show `SAVE STRING` after load (the committed prerequisites proposal, case SV0) are superseded by this section.

## 2. How the original stores and loads save data (source evidence)

**One shared object.** Name = `"so_" + gameName + 1` = `so_deadly301` (`GameController.getSharedObject`, line 271-275; `gameName = "deadly30"`, line 43) [V]. Ruffle with `--storage disk` keeps it at `<save dir>\localhost\<SWF path without drive>\so_deadly301.sol`, keyed by the SWF's own path [D: Stage 3b acceptance report, three disposable saves]. Ruffle writes it when the process exits normally; the game never calls `flush()` [D].

**Every key and its reader/writer [V].**

| Key | Written by | Read by | Needed for Continue |
|---|---|---|---|
| `saveString` | `Level.addCheckPoint` (1700-1702) | `Level.loadCheckPoint` (1792-1797) | yes (the checkpoint) |
| `nightsLasted` | `Controller.addGameOver` (520-531) and the victory path (195-207), as `max(old, Tracker)` | menu frame script (`MainTimeline.as:154-199`) | yes: Continue is enabled only if it is ≥ 1 (176); text: 1–29 "You Lasted N Nights", exactly 30 "Survivors have escaped!" (becomes 60 after Stage 3), anything else no text |
| `lvl` | HUD level-up (HUD.as:358) | `Controller.loadSavedVariables` (237-240), guarded | no |
| `achievedGuns` | `Level` (1497-1498) | `loadSavedVariables` (233-236), guarded | no |
| `achievedMedals` | `MedalManager` (45-46) | `loadSavedVariables` (229-232), guarded | no |
| `quality` | Settings (86-87) | `GameController.setup` (300-303), guarded | no |
| `resolution` | `GameController` (140-141) | `GameController.setup` (296-299), guarded | no |

All keys except `saveString` and `nightsLasted` are read with null/falsy guards, so a shared object holding only those two keys is acceptable to the game's own code [V]. Whether the runtime accepts a hand-built file is a separate question ([U], section 4).

**`saveString` layout [V]** (written 1602-1699, read 1805-1881). Six `|`-separated sections:

| # | Content | Separator | Loader behavior and failure modes |
|---|---|---|---|
| 0 | night, scrap, medkits, armor, fence, boots, ammo0, ammo1, ammo2, kills (10 comma fields, integers) | `,` | `nightCount = int(field0)`; the loop at 1888-1899 reads `waveArray[0 … nightCount−1]` and calls `.length`, so a night above the number of rows throws (a night above 30 in the original build; above 60 in the candidate) |
| 1 | one soldier skin id per soldier | `,` | one `addSoldier` per entry; skin 0 = American, 1 = Russian, 2 and above other soldier types |
| 2 | one equipped weapon id per soldier | `,` | one `equiptWeapon(soldierArray[i], id)` per entry; **an id not in the weapon table, or more entries than soldiers, dereferences null** (2918-2945) |
| 3 | bought weapon ids | `,` | ids matched against the weapon table; unknown ids are ignored |
| 4 | boxes `x,y` | `:` between boxes | empty string = none (1864) |
| 5 | turrets `x,y,scaleX,type` | `:` between turrets | empty string = none (1873) |

Weapon ids 1–12 exist (Level.as:1508-1519; ids 1–3 start owned). The loader indexes sections 0–5 unconditionally, with no try/catch; a missing section throws [V]. Sections beyond 5 are never read [V].

**Defaults used by a new game (a safe baseline for plausible values)** [V, Level.as:2021-2030]: one soldier (skin 0), scrap 5,075, 1 medkit, ammo 15/200/18, armor/fence/boots upgrade level 1.

## 3. Verified facts, assumptions and unresolved questions

| Item | Status |
|---|---|
| Shared-object name, key list, readers/writers, `saveString` layout, loader failure modes, Continue gating | **Verified** in source (static) |
| Loader side effects (no checkpoint rewrite, counter not incremented, `DAY!` trace) | **Verified** in source (static); not runtime-observed |
| Ruffle disk layout and write-at-exit | **Documented** earlier ([D]); a hand-built file placed there is unverified |
| A `.sol` is AMF0 with a 4-byte length at offset 2 (= file size − 6) and 2-byte string lengths | **Documented** earlier ([D], derived from a byte dump of a copy of the real save under earlier approval; marked an assumption there); not re-verified |
| Exact header bytes, property terminators, number/string markers and padding of the container | **Unresolved**: no non-personal reference file is known to exist in the lab |
| Whether Ruffle (and the original build under Ruffle) accepts a from-scratch file | **Unresolved**; this is the question SV0 answers |
| Contents of the lab folder `acceptance\savetest` | **Unresolved and deliberately not inspected**: it may hold a copy derived from the real save |
| The CONTINUE button position | **Verified** at about (140, 188) in a menu screenshot from this session (stage coordinates, disabled state); to be re-confirmed before clicking |
| Compatibility of the user's real saves with the candidate | **Cannot be established from synthetic files** (section 6) |

## 4. Proposed B6 design (all of it a plan; nothing exists)

### 4.1 Principles
- Build everything under one new lab root `D:\Projects\deadly30_lab\game\d60_rollout\SAVE\`, never inside the repository, never under the user profile.
- No file read or copied from a real save, its folder, its backup or `acceptance\savetest`. The generator has **no input-file path at all** (its content is built in), so it cannot read user data by construction.
- Create-new only; nothing is ever overwritten, moved or deleted by the tools.
- One Ruffle window at a time, free memory ≥ 2.00 GiB before every launch, graceful close only, stop at the first anomaly.

### 4.2 Source of the container format (decision for Phase 2)
- **Route A (preferred): a Ruffle-authored, non-personal reference file.** Run the original SWF (a copy at a fixed isolated path) with `--storage disk --save-directory <new session dir>` through a fresh game, with the user's early manual nights (section 7, variant B) or the unattended flow, and let the game and Ruffle write the file themselves. A fresh-game file contains no personal data and shows the exact container Ruffle reads and writes. The generator then only substitutes values (night, strings) into a template laid out exactly like that file.
- **Route B: build from the documented AMF0 layout** with a built-in constant (header, name `so_deadly301`, properties `nightsLasted` as number and `saveString` as string) and a self-check parse. It needs no run first but rests on the assumptions above.
- A unified design is possible: try Route B first; if the original build refuses the file in SV0, switch to Route A. Neither route reads real data.

### 4.3 Generator (to be created only in Phase 2)
- Input: a case id from a fixed table (below), nothing else. Output path is computed, never taken from the caller: `SAVE\inputs\<CaseId>.sol`.
- Refuses if the path is not strictly inside `SAVE\inputs\`, contains `..`, is UNC or drive-relative, has a reparse point (junction/symlink) in any ancestor, already exists, or resolves under `%APPDATA%`, the user profile data folders, `backup_saves` or `acceptance`.
- Builds the shared-object bytes, then **re-parses its own output** and checks: header length = file size − 6, string length field = actual length, terminators intact, the `saveString` has exactly six sections with the expected field counts, ids are in the allowed sets for the case. Records SHA-256, size and an audit log of every path it touched (it should touch exactly one: the output file).
- Fails closed: any check failing deletes nothing, writes nothing further, and reports the first error.

### 4.4 Launcher and preflight (to be created only in Phase 2)
Preflight runs first, with no Ruffle and no write except its own new log folder, and must pass all of:
1. The SWF under test is a copy at `SAVE\run\<name>.swf` and its SHA-256 equals the recorded value (original: `3B9ABEE6…FC9C`; candidate: `155A5167…304D`).
2. The session folder `SAVE\sessions\<Tag>\` does not exist yet (create-new), is canonically inside `SAVE\sessions\`, and has no reparse point in its ancestry.
3. The planned Ruffle command line contains `--storage disk` **and** `--save-directory` equal to `<session>\savedir`, and `--config`/`--cache-directory` inside the lab; a missing or different value refuses the launch.
4. The input `.sol` named by the manifest exists in `SAVE\inputs\`, its hash equals the manifest, and it is copied (not moved) to `<session>\savedir\localhost\<SWF path without drive>\so_deadly301.sol`.
5. No live Ruffle or java process or window; raw free memory ≥ 2,097,152 KB (3 rechecks, 20 s apart, then stop).
Launch uses the existing hash-pinned input helper and the same verified flags as the earlier runs; graceful close only. Afterwards the launcher lists only the session folder, compares the `.sol` hash before and after, saves stdout/stderr and the trace, and records everything create-new.

### 4.5 Isolation: what it proves and what it cannot
- By construction: Ruffle does not use the Flash Player shared-object folder where real saves live; the launcher forbids runs without an explicit `--save-directory`; the generator has no read path; every path is allowlisted.
- Not provable by observation: nothing outside the lab may be listed or hashed, so the absence of a read of the real save cannot be demonstrated by comparison (this is the rule that protects the save). Optional defense in depth, each needing its own approval: running Ruffle with a restricted environment, or in Windows Sandbox (changes a system feature; GPU support uncertain).
- The claim would therefore be "isolated by construction; nothing observed contradicts it", never "proven".

### 4.6 SV0: the first, original-build compatibility check
- Build: the **original** SWF copy; input: a minimal valid synthetic checkpoint at night 12 (one soldier skin 0; equipped id 2; bought ids 1,2,3; empty box and turret sections; defaults from section 2) and `nightsLasted` = 11. Not run in Phase 1.
- Steps: preflight → launch → menu shown → (screenshot to confirm the Continue position) → click CONTINUE → observe → graceful close.
- Expected, from the source (predictions only): menu text "You Lasted 11 Nights" and Continue enabled; traces `LOADED STRING = …` equal to the input, then `DAY!`; HUD "Day 12"; no `SAVE STRING`; no AVM2 errors. If the soldier is allowed to die before the next dawn, the game-over trace should read `Nights Lasted: 11`, which would be the first runtime confirmation of the counter quirk in section 1.
- Acceptance: all of the above observed. If the container is rejected (menu shows no text and a disabled Continue, or a parse error is logged), stop, record, and decide between Route A and a format fix.

### 4.7 Case table (original build first, then the candidate)

| Case | Content | Build | Prediction [I/V code] |
|---|---|---|---|
| SV0 | night 12, `nightsLasted` 11 | original | loads, as in 4.6 |
| SV1 / SV2 | night 29 / 30 | original | loads |
| SV3 | night 31 | original | throws at `waveArray[30]` during Continue |
| SV4 | unknown equipped id (for example 99) | original | null dereference during load |
| SV5 | unknown bought id | original | ignored |
| SV6 | a seventh section appended | original | ignored |
| UV0 | the SV0 file | candidate | loads identically |
| UV1 | night 29 | candidate | loads; the following night is ordinary night 30 |
| UV2 | night 30 | candidate | loads; night 31 follows |
| UV3 | nights 34, 35, 55, 59 | candidate | loads; for 35 and 55 no cutscene and no freeze |
| UV4 | night 5 (and 10, 25) | candidate | story cutscene replays after about 4 s if not yet played this session |
| UV5 | `nightsLasted` 30, 59 and 60 | candidate | menu text per the new thresholds |

Minimum first set: SV0, SV3, SV4. Cases that reach the target nights (5, 25, 30, 31, 35, 55, 59) need the later phases.

## 5. Stop rules

Stop immediately, record the original output and ask, on: any generator refusal or self-check failure; any parse or load error that is not the case's predicted result; any path outside `SAVE\`; a failed isolation check; an unexpected process or window; a window that ignores the close request (never kill); free memory below the gate; a changed hash of the input `.sol` or the SWF under test.

## 6. What synthetic tests cannot show

- The user's real save was written by Adobe's Flash Player inside the EXE wrapper, and the candidate is run by Ruffle. A hand-built or Ruffle-authored file proves that the game's **loader** accepts that data in Ruffle; it says nothing about how a Flash-Player-written file (possibly with different array encodings for `achievedMedals`/`achievedGuns`, several soldiers, many turrets and boxes) behaves.
- Compatibility with real saves would need a separately approved test on a disposable copy of a user save (or the owner's own Continue test in the real environment). It cannot be claimed from synthetic cases.
- A synthetic dawn-59 save only reaches night 60. The 5:00 countdown, the helicopter, boarding, cutscene 6, the menu text and the `nightsLasted` value of the finale require **direct human gameplay observation**.

## 7. Optional early-night manual checklist (Nights 1–5, candidate, user-run)

No automation, no saves. Run the candidate with in-memory storage so nothing is written (single line):

`& "D:\Projects\deadly30_lab\runtime\ruffle\ruffle.exe" --graphics dx12 --storage memory --load-behavior blocking --dummy-external-interface --config "D:\Projects\deadly30_lab\game\harness\ruffle_cfg" --cache-directory "D:\Projects\deadly30_lab\game\harness\ruffle_cfg" --width 750 --height 460 "D:\Projects\deadly30_lab\game\d60_rollout\stage4\builds\S4_TEXT_1\deadly30.swf"`

Before you start: free at least 2 GiB of memory; close extra Ruffle windows; press Space only **while the intro is playing** (pressing Space/Enter/Esc at the menu after the intro ended by itself triggers the original game's harmless repeating `#1009` error described in the follow-up report). Note what you see and nothing more; these observations say nothing about nights above 5.

1. Menu: title art (still "Deadly 30"), CONTINUE greyed with no progress.
2. New game → intro cutscene → HUD "NIGHT 1".
3. Night 1: enemies come in waves; kills raise EXP; the night ends when the wave is cleared. Write down whether anything stalls or repeats.
4. Dawn: a "Day 1" banner with "Infected Killed" and a bonus-scrap line; the HUD clock changes.
5. Nights 2–4: the intro banner shows the original names ("What are these green creatures!?" for night 2 and so on). Note the HUD "NIGHT n" and any odd text.
6. End of night 5: the first story cutscene should play after the night is cleared; note that it plays and returns to the game at dawn.
7. Continue test inside the same session (storage is in memory): if you die, return to the menu and click CONTINUE; note the "Day n" banner, the squad, weapons and scrap restored, and the night number. After closing Ruffle nothing remains.
8. Optional: record the Ruffle window title and Windows time of each observation.

Report as: "observed" for what you saw, "not observed" for the rest. Variant B (to capture a non-personal reference save) needs a new disposable folder and is part of the Phase 2 approval, not this checklist.

## 8. Phases and the approvals they need

| Phase | Content | Approval needed |
|---|---|---|
| 1 (this report) | design and source inspection | none further |
| 2a | create `d60_rollout\SAVE\` with `inputs\`, `run\`, `sessions\`, `tools\` and an isolation statement file (no scripts yet) | explicit approval to create the folder |
| 2b | choose Route A or Route B; approve the generator, a static audit of it (findings fixed), then a generator **dry run that writes one file** in `SAVE\inputs\` | explicit approval of the generator and of writing its first file |
| 2c | launcher and preflight scripts, static audit, **preflight-only run** (no Ruffle) | explicit approval of the launcher |
| 3 | SV0 on the original build, one window | explicit "run SV0" approval with the memory gate met |
| 4 | SV1–SV6 | approval after the SV0 result is reviewed |
| 5 | the candidate cases UV0–UV5 | approval after Phase 4 |
| 6 | human finale observation, real-save compatibility (if wanted) | separate approvals; the latter needs an explicit decision about any copy of user data |

**Approvals requested now (for Phase 2 only, each separately revocable):** (1) create the `SAVE` folder skeleton under `d60_rollout`; (2) Route B first with Route A as the fallback (or tell me to start with Route A); (3) create the generator script, to be statically audited before any run; (4) later, the launcher with a preflight-only first run. Phase 3 and everything after will be requested separately.

## 9. Risks and safeguards

| Risk | Safeguard |
|---|---|
| A tool touches a real save or its folder | no input-file path in the generator; allowlisted canonical paths; explicit `--save-directory`; no listing of any user folder; stop rules |
| The hand-built container is rejected or mis-parsed | self-check parse; SV0 first; Route A fallback; stop on the first anomaly |
| A wrong prediction (like the earlier one in section 1) misleads | predictions are labelled; SV0 observes the actual load trace and the counter before any later case is trusted |
| A modified binary is tested instead of the candidate | only the unmodified original and the Stage 4 candidate are used; no test hooks |
| Memory pressure invalidates a run | gate at launch; runs stay one window at a time |
| Over-claiming | every report keeps observed / not observed separate; "Deadly 60 is not runtime-verified" stays until the target nights are actually observed |

## 10. Status summary

Phase 1 is complete as a design. The save format used by the game is documented from source; the file container format is not independently verified and is the main open question; the earlier assumption about the post-Continue counter was wrong and is corrected above (a continued win ends at 59 in the 60-night design, as it ends at 29 in the original). Nothing was created or run. **Deadly 60 is not runtime-verified.**
