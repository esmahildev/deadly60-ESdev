# Stage 3 — Technical Feasibility Report

Scope: feasibility only. No feature work, no change to the original SWF, no save touched. All experiments ran on copies in a lab folder outside this repository; nothing from the game (SWF, decompiled source, generated files) is committed.

Result summary: **export and rebuild are feasible; running the result is NOT yet verified.** The unmodified SWF and the rebuilt SWF behave identically in the only runtime tested (Ruffle), but neither reaches the menu in it.

## 1. Backup of the original (PASSED)
- Original SWF copied to a read-only backup; SHA-256 of source and copy match (19,250,808 bytes, `FWS` v10). The memory-dump copy of the same SWF has the identical hash.
- Whole `analysis/` folder (390 files, 41,808,497 bytes) copied; per-file hash comparison: 0 mismatches; manifest of 391 hashes written and re-verified at the end (0 mismatches).
- The backup was never passed to any tool; tools only received a writable copy (`input.swf`, hash-identical to the original).
- The real save file was also hashed and given a read-only backup copy (see 5).

## 2. FFDec export (PASSED)
- FFDec 26.3.0 (stable release), SHA-256 matches the digest GitHub publishes for the asset. Runs on the installed Java 8.
- `-export script`: 381 of 381 scripts in 34 s; output is byte-identical to the earlier decompile (0 files differ).

## 3. No-change rebuild (PASSED with caveats)

| Route | Result |
|---|---|
| SWF -> XML -> SWF (`-swf2xml`/`-xml2swf`) | Works, but not byte-identical: 60 tag lines (vector shapes, fonts, sprites) are re-encoded. **Not recommended for patching.** |
| `-replace` of AS3 scripts from their own exported source | **Recommended route.** 16 classes each recompiled alone (6-15 s each); 18 classes (incl. `Level`, `Soldier`, `Controller`, `HUD`, `Shop`, `SafeRoom`, `Helicopter`, `GameController`, `Deadly30_fla.MainTimeline`) in one build took 40 s. |

- Tag-level diff of the 18-class rebuild vs the original: exactly **one** tag differs, `DoABC2` (all game code, 305,230 -> 327,872 bytes). The other 24,031 tag lines (art, sound, shapes, sprites, symbol bindings) are identical. A patch therefore touches only the code tag.
- Re-decompiling the rebuilt SWF: 6 of the 18 classes are textually identical; 12 differ. After removing compiler-inserted type wrappers (`Boolean(...)`, `Number(NaN)`), 15 lines remain: `x -= 1` becoming `x = x - 1`, renumbered temporaries in one `Level` filter builder, and a metadata attribute line. None looked behavioral, but **this is unproven at runtime**.
- Edit cycle proven: a diagnostic copy with added `trace()` lines compiled and the traces appeared at runtime.
- FFDec flags `-replace` as EXPERIMENTAL. Script name for the document class is `Deadly30_fla.MainTimeline` (not `MainTimeline`).

## 4. Runtime (FAILED to reach the menu; not caused by the rebuild)
- Runtime: Ruffle desktop nightly 2026-10-16 (0.8.0-nightly), SHA-256 matches GitHub's digest; binary is unsigned. Run with `--storage memory` so no shared object was written. Backends: `dx12` and `gl` start; `vulkan` exited at launch with no error.
- Baseline (unmodified copy) and the 18-class rebuild behave identically: SWF v10, 750x460 @ 30 fps loads, `GameController.setup` runs (its `trace` output appears), then the window stays black. No AVM2 error is logged.
- Evidence (diagnostic build, test copy only): on main-timeline frame 2, `totalFrames=8`, Ruffle's label table is correct (`3=Preload Objects, 4=Intro, 5=Menu, 6=Credits, 7=Settings, 8=Game`), but after `gotoAndStop("Intro")` it is still on frame 2 with `currentLabel = null`; a numeric `gotoAndStop(4)` behaves the same. Frame 4/5 scripts and the `Intro` constructor never run. So the game never leaves its first frames in Ruffle.
- Not tested: gameplay, saves, any in-game system. Whether the original runs in Adobe's player is not tested here (the original EXE bundles its own Flash runtime and was not run).
- Side effects of running Ruffle: it created its own data folder under `%LOCALAPPDATA%\ruffle` (`recents.toml`, a log, and an automatically downloaded Cisco OpenH264 DLL, Authenticode signature valid) and one stray `ruffle` process remains that could not be stopped (access denied).

## 5. Save location (found, not modified)
- `%APPDATA%\Macromedia\Flash Player\#SharedObjects\<random-id>\localhost\so_deadly301.sol` (single file, 311 bytes, last written 2026-10-14). Matches the game's `so_deadly30` + slot 1 naming (`GameController.getSharedObject`).
- Not opened, not modified: hash unchanged after all runs; a read-only backup copy was made. Ruffle with `--storage disk` would use its own folder, so Ruffle testing cannot touch this file.

## 6. Limitations and risks
- No working runtime yet, so rebuilt-code equivalence and every later feature test is blocked until one exists.
- FFDec's AS3 compiler is experimental; each patched class must be re-verified by re-decompiling and by running.
- Running a patched SWF inside the original EXE wrapper is untested (the SWF is embedded in the third-party EXE; replacing it was out of scope).
- Ruffle's AVM2 and timeline behavior may keep diverging from Flash Player even after this issue is worked around.

## 7. Recommended next step (needs your decision; Stage 4 is NOT started)
Bounded "Stage 3b" to obtain a runnable test environment, then stop and re-report:
1. Test-only shim in a diagnostic copy (for example `nextFrame()` instead of `gotoAndStop("Intro")`) and one older and one newer Ruffle build, to see whether this is a Ruffle regression or a quirk of this SWF.
2. If Ruffle cannot run the menu, run patched SWFs in a Flash runtime you are licensed to use (for example by replacing the SWF in a copy of your own installation), with your approval.
3. Only after a runtime plays the game: re-run the Classic-mode regression oracle from the Stage 2 plan, then begin Stage 4.

## Results by category
- **Passed:** original backup + verification; FFDec export 381/381; single-tag (`DoABC2`) rebuild of 18 classes; edit-compile-trace cycle; real save located and left unchanged.
- **Failed:** the game does not reach its menu in Ruffle (baseline and rebuild alike); Vulkan backend start.
- **Skipped:** gameplay, save migration, running the original EXE, replacing the SWF inside the EXE.
- **Unverified:** runtime equivalence of recompiled classes; behavior in Adobe's Flash runtime; whether the stray `ruffle` process is from these tests.
