# Stage 3b — Acceptance Check Report

Scope: confirm that the pinned test bed can run real gameplay and save/load, for both the original SWF and the 18-class no-change rebuild. No feature work, no gameplay source changes, nothing from the game committed. All experiments used disposable copies in a lab folder outside this repository.

## Verdict
**The test bed is usable for Stage 4, with conditions.** Gameplay, combat, death, pause, the night-to-dawn transition with a checkpoint write, and save persistence across a full restart all work in Ruffle for both SWFs, with 0 runtime errors in 15 of 16 runs, provided `--dummy-external-interface` is used (one run logged a burst of 6 transient errors that could not be reproduced). Two gaps remain: the dawn of *night 1* was not reached (the dawn transition was reached on a disposable copy of a mid-game save instead), and gameplay classes outside the exercised paths (for example the helicopter finale) are still untested.

## 1. Runtime and command lines
- Ruffle desktop **nightly 2026-10-16, 0.8.0-nightly.2026.10.16**, official GitHub release, asset digest verified, binary unsigned. `ruffle.exe` SHA-256 `A4C96B92...43E7` (unchanged at the end).
- Graphics: `dx12` on Intel HD Graphics 520.
- Base command (all runs): `ruffle.exe --graphics dx12 --storage <memory|disk> --load-behavior blocking --config <lab dir> --cache-directory <lab dir> --no-gui --width 750 --height 460 [--save-directory <lab dir>] [--dummy-external-interface] <swf>`
- Storage: `--storage memory` for Tests A/B; `--storage disk --save-directory <disposable dir>` only for Test C, and only inside the lab.
- Input: keyboard via window-targeted messages (works); the mouse needed the real cursor (Ruffle ignores posted mouse messages), guarded by a check that the window under the target point is the Ruffle window.
- Input/driver scripts, logs and screenshots are lab-side test artifacts and are not committed.

## 2. The `ExternalInterface` blocker
| Config | Runs | Runtime errors per run | Game over screen |
|---|---|---|---|
| no flag | 3 (1 original, 2 rebuilt) | 161-269 (1x `#2067 ExternalInterface not available`, 2x `enterFrame #1009`, the rest repeating frame-script `#1009` roughly every 0.7 s) | stays black |
| `--dummy-external-interface` | 16 (original and rebuilt) | **0 in 15 runs**; 6 transient `#1009` frame-script errors in 1 run (Settings screen, right after a hard kill of the previous process on the same disk storage; not reproduced in 3 reruns of the Settings screen: rebuilt x2, original x1; cause unknown) | shows the stats scroll, credits, then returns to the menu |

What the flag changes: Ruffle reports an `ExternalInterface` that exists but does nothing, so the Zinc-wrapper calls (`exe = true` path, for example the hide/show-mouse call) no longer throw. No SWF or code is modified. Without it, gameplay starts but is degraded and noisy; with it, the game runs cleanly. The original SWF showed the same error profile as the rebuild in both configs, so these errors come from the runtime environment, not from the rebuild.

## 3. Test A — gameplay (rebuilt SWF)
Reproducible steps: launch with the base command plus `--dummy-external-interface`; press Space to skip the intro; click NEW GAME; Space to skip the cutscene.

| Item | Result |
|---|---|
| Gameplay begins | **Yes.** HUD, tutorial hints, crosshair, MP-40 30/30, 200 rifle rounds, 5075 scrap, 1 medkit, NIGHT 1, LVL 1 (matches the starting values read from the code). |
| Movement / jump | Yes (world scrolls, jump visible). |
| Weapon fire / reload | Yes. Mag 30 -> 25 -> 18 during a burst; reload restores 30 and the reserve drops by exactly the rounds used. Knife used when the magazine is empty. |
| Enemy spawning | Yes. Zombies arrive from both sides in groups of 2-3, from about 70 s of game time. |
| Damage | Yes. Enemy kills with gibs; the player flashes red when hit, the health bar drops; one medkit used (count 1 -> 0, bar refilled). Achievement pop-ups ("Kill a naked zombie") appear. |
| Player death | Yes: 10 game-over events across all runs (7 with the dummy flag; about 5-7 kills in the runs where I fought, none when idle): grayscale fade, then the Game Over scroll (kills, survivors, nights, scrap), credits, return to the menu. |
| Pause | Yes (P toggles; pause panel shows controls, quality and resolution). Pause state is read from the game's own `PAUSE`/`NOT PAUSE` traces. |
| **Dawn of night 1** | **Not reached.** 4 attempts with 3 tactics (blind bursts, aimed bursts with pause, retreating through the base); the 36-HP starting soldier dies before all 11 zombies are cleared. Injecting input blindly is too imprecise to rely on. |
| **Dawn transition (other route)** | **Reached**, on a disposable copy of a mid-game save: sleep until night (E at the bed), night 13 played out in about 73 s with the squad and turrets, then `SAVE STRING = 13,...` and `DAY!` fired (checkpoint written at dawn). 0 errors. |

Differences from the original: none found beyond run-to-run randomness (see Test B). Effective speed: the first-zombie lead-in takes 60-78 s of wall-clock; if it is the 1,450-frame tutorial from the code, that implies roughly 19-24 fps against the nominal 30 (inferred, not measured). Timing-sensitive balance checks need that caveat.

## 4. Test B — original vs rebuilt
| Check | Original | Rebuilt |
|---|---|---|
| Menu reached, 13 s intro | yes | yes |
| No-flag error profile | same classes (`#2067` x1, `enterFrame #1009` x2, repeating `#1009`) | same |
| Dummy-flag errors | 0 | 0 |
| New game: starting HUD and values | identical | identical |
| Firing/reload bookkeeping | mag used = reserve used (7 and 7) | mag used = reserve used (12 and 12); counts differ only with how long the key was held |
| Unattended death, same inputs (no flag) | game over 67 s after level creation (1 run) | 64-66.5 s (2 runs) |
| Continue on a real-save copy | loads night 12 checkpoint | identical load string |
| Sleep -> night 13 -> dawn | `NIGHT!`, `SAVE STRING = 13,...`, `DAY!` in 77 s, 0 errors | same in 73 s, 0 errors |
| Dawn bonus scrap | identical value | identical value (deterministic) |
Only ammo remaining and kill count differ (random per run). Matching the original's menu was not treated as proof; the comparison covers gameplay, error profile, death, load and dawn.

## 5. Test C — save persistence (disposable storage only)
Isolation: a dedicated empty directory inside the lab for each test (`S1`, `S2`, `S3`); config and cache also inside the lab; before and after the tests no `.sol` file outside the lab was created or modified (checked across the user profile data folders); the real save hash and timestamp are unchanged.

| Test | Result |
|---|---|
| Change quality setting, same process | game reads the new value back |
| Hard kill, then full restart | **setting lost, no file ever written** |
| Graceful close, then full restart | **setting persisted** (file written on exit) |
| Storage layout | `<save dir>/localhost/<SWF path>/so_deadly301.sol`: **keyed by the SWF's own path**, so the original and rebuilt SWFs (different paths) do not share saves, and renaming or moving a SWF loses its saves |
| Disposable copy of a real save (format check) | menu shows "You Lasted N Nights", CONTINUE enabled, checkpoint loads (night, squad, weapons, upgrades, turrets all restored) |
| Dawn checkpoint write, graceful close, full restart, CONTINUE | **persisted**: restart loads the night-13 checkpoint; the file's hash changed only in the disposable copy |
| Same flow on the original SWF with its own copy | identical results |
Cause of the hard-kill loss: the game calls only `SharedObject.close()` (reported as a stub by Ruffle) and never `flush()`; Ruffle writes to disk when the process exits normally.

## 6. Hashes (SHA-256)
| File | Before | After |
|---|---|---|
| Original SWF (backup) | `3B9ABEE6...FC9C` | `3B9ABEE6...FC9C` |
| Original SWF (analysis copy) | `3B9ABEE6...FC9C` | `3B9ABEE6...FC9C` |
| Working copy of the original | `3B9ABEE6...FC9C` | `3B9ABEE6...FC9C` |
| 18-class rebuild | `0B24B469...B699` | `0B24B469...B699` |
| Backup manifest (391 entries) | 0 mismatches | 0 mismatches |
| Real save | `AFAF490B...` | `AFAF490B...` (size and last-write time unchanged) |
The original SWF, its backups, the original executable and the real save were not modified; the executable was not run.

## 7. Files changed and test artifacts
- Repository: this report and a README row only (branch `stage-3b-acceptance`).
- Lab (outside the repository): 7 driver scripts; 60 log files; 275 screenshots (42 MB); 3 disposable save files (45, 311, 311 bytes); before/after hash files; isolated Ruffle config/cache folder; plus earlier Stage 3/3b artifacts. No game code was changed in any new build; the only SWFs used were the working copy of the original and the unmodified 18-class rebuild.
- Outside the lab, from Ruffle itself: its data folder under the user profile (recents list, log, the Cisco OpenH264 DLL) and one terminated-process remnant (see below). Windows crash dumps from the earlier driver crash. None were deleted.

## 8. Remaining risks
1. Hard exits lose all saves; any automated test must close Ruffle gracefully.
2. Storage is keyed by SWF path: pin one SWF file name/location, or saves will not carry over between builds.
3. Mouse input needs the real cursor (the player window must be visible and topmost); automated tests cannot run in the background.
4. Reaching the first dawn of a new game by manual input is impractical; Stage 4 verification should rely on the loaded-save route above, or on test-only hooks (night jump, ammo) in disposable builds, as the Stage 2 plan already allows.
5. Not exercised: the final night, helicopter and ending, nights beyond 13, the shop UI, multi-soldier switching, and weapons other than the MP-40/Golden AK in the loaded save.
6. Ruffle nightly pinned; a different build may behave differently (the oldest build tested fails to start the game).
7. Performance: effective frame rate looks below 30 fps on this machine; large-wave performance tests need a proper measurement.
8. The Vulkan backend crashes in the Intel driver; use `dx12`.

## 9. Cleanup suggestions (not performed)
- Lab: the 42 MB of screenshots and the logs can be deleted when no longer needed.
- Ruffle data in the user profile: recents list, log and the OpenH264 DLL (only useful for video, which this game does not use).
- The terminated-process remnant (0 threads, 0 handles) should clear on sign-out or reboot.
- Windows crash-dump files from the earlier Vulkan crash.

## 10. Recommendation
Approve Stage 4 planning on top of this test bed, keeping these rules: graceful close only; fixed SWF path; `--dummy-external-interface` and `--load-behavior blocking` always on; use the loaded-save route for dawn and checkpoint tests; add test-only hooks to disposable builds only if night-1 coverage is needed. Stage 4 itself has not been started.

## Results by category
- **Passed:** gameplay start; movement; firing and reload; enemy spawning; damage and kills; player death and game over; pause; original-vs-rebuilt comparison; dawn transition and checkpoint write (loaded save, both SWFs); save persistence across graceful restart; real-save format loads; originals, backups and the real save unchanged.
- **Failed:** nothing in the functional checks; the no-flag configuration is degraded (161-269 errors per run) and is not recommended. One unexplained burst of 6 transient errors (see section 2) occurred in 1 of 16 dummy-flag runs.
- **Skipped:** the first dawn of night 1 (unreliable manual control, 4 attempts); running the original executable; final night and ending.
- **Unverified:** the cause of the one transient error burst; behavior beyond night 13; large-wave performance; other Ruffle versions with `--dummy-external-interface`; multi-day sessions.
