# Stage 3b — Test Environment Report

Goal: a reliable, reproducible runtime in which the original and the rebuilt Deadly 30 SWFs reach the intro and main menu.
Result: **goal A reached.** Ruffle reaches the intro and menu for both SWFs using runtime flags only, with no game-code change. Gameplay itself was not tested.
Nothing from the game is committed. All experiments ran on disposable copies in a lab folder outside this repository.

## Reproducible recipe
```
ruffle.exe --graphics dx12 --storage memory --load-behavior blocking <file.swf>
```
Optional isolation of Ruffle's own data: `--config <dir> --cache-directory <dir>`. `--storage memory` means no shared object (save) is ever written.

## 1. Tests performed and exact runtimes
All builds come from the official `ruffle-rs/ruffle` releases; each download's SHA-256 was compared with the digest GitHub publishes for the asset (all matched). Binaries are unsigned. Ruffle only publishes dated nightlies; no newer build than 2026-10-16 existed.

Original SWF (copy) on dx12 (Intel HD Graphics 520), 24-28 s per run:

| Runtime | no flag | `--load-behavior blocking` | `--load-behavior delayed` |
|---|---|---|---|
| nightly 2026-10-16 (0.8.0-nightly) | stalls | **menu** | **menu** |
| nightly 2026-04-14 (0.2.0-nightly) | stalls | **menu** | **menu** |
| nightly 2025-10-14 (0.1.0) | stalls | stalls | stalls |

- "stalls" = game code starts (settings traces appear), window stays black, menu never reached.
- Graphics backends on the newest build: `dx12` and `gl` work; `vulkan` crashes inside the Intel graphics driver (Windows Error Reporting: `igvk64.dll` 31.0.101.2111, exception `0xC0000005`, faulting PID 0x46E8 = the first test launch). That is a driver crash, not a game issue.

## 2. Does the original SWF run?
Yes, on the two newer builds with `blocking` or `delayed`: the intro plays (credits screen seen), then the main menu appears (logo, greyed CONTINUE, NEW GAME), and the game's own menu code prints `Nights Lasted: 0`. CONTINUE is greyed (the game enables it only when a saved `nightsLasted` exists), which is consistent with the in-memory storage not exposing the real save.

## 3. Does the rebuilt SWF run?
Yes. The 18-class no-change rebuild from Stage 3 behaves identically: all 4 runs with `blocking` (and the 1 run with `delayed`) reach the menu. In the six repeat runs (3 original, 3 rebuilt, newest build, `blocking`) every run reached the menu 13.2-13.4 s after load, with 0 error lines. Limit: only intro and menu code ran, so the recompiled gameplay classes (`Level`, `Soldier`, ...) are still unexercised.

## 4. Test-only frame-jump workaround
A test copy (only `Deadly30_fla.MainTimeline` changed) selected behavior by a startup parameter:

| Variant | Result |
|---|---|
| original `gotoAndStop("Intro")` | ignored, stays on frame 2 |
| 1: `nextFrame()` | ignored, stays on frame 2 |
| 2: `gotoAndPlay("Intro")` | ignored at first; the timeline later ran on by itself through the "Game" frame and looped, producing unrelated errors, so this variant is invalid as a test |
| 3: wait until `framesLoaded == totalFrames`, then `gotoAndStop("Intro")` | **works**: frame 4 / label `Intro`, then Menu |
| 4: one-frame delay, no load check | ignored |

This is a diagnosis, not proof that the original game is correct. The shim is unnecessary because the runtime flag achieves the same effect without touching code.

## 5. Earliest confirmed failure point
`MainTimeline` frame 2 script (`stop(); GameController.setup(...); gotoAndStop("Intro")`). At that moment Ruffle reports `framesLoaded = 2` of `totalFrames = 8` and only 8.6-11.2 MB of the 19.25 MB SWF read. A jump to a frame that is not loaded yet is silently ignored, nothing is logged, and the timeline sits on frame 2. Evidence: the label table is correct (`3=Preload Objects, 4=Intro, 5=Menu, 6=Credits, 7=Settings, 8=Game`); variant 3 succeeds only once `framesLoaded` reaches 8; variant 4 (delay without waiting) fails; Ruffle's `blocking`/`delayed` load modes fix it with no code change. Classification: a Ruffle load-order compatibility issue (the SWF assumes a fully loaded movie, as when embedded in its EXE); not a defect of the rebuild.
Log excerpt (newest build, test copy):
```
DIAG f2 variant=0 frame=2 framesLoaded=2 totalFrames=8 bytesLoaded=8663779 bytesTotal=19252240
DIAG f2 after: frame=2 label=null
```

## 6. Changes made to disposable copies
Only the test copies listed in a lab-side record: the Stage 3 trace builds (`Intro`, `Sfx`, `MainTimeline` with added `trace()` lines) and the Stage 3b variant build (`Deadly30_fla.MainTimeline`: a parameter switch, the variants above, trace lines, one extra field and one extra handler). The flag-only runs used `input.swf` (hash-identical to the original) and the unmodified 18-class rebuild. None of these files may be used as deliverables.

## 7. Original, backups and real save
- Original SWF, its backup copy, and the analysis copy: hash-identical to the baseline, re-verified at the end. 391-entry backup manifest: 0 mismatches.
- Real save: hash and timestamp unchanged since before Stage 3; read-only backup copy still matches.
- No shared-object (`.sol`) file was written anywhere by any Ruffle run (all used `--storage memory`); 0 found under the lab or Ruffle data folders.
- Remaining uncertainty about saves: in-memory storage cannot test save persistence. Ruffle reports `SharedObject.close()` as a stub, and the game relies on `close()` instead of an explicit flush, so whether checkpoint saves persist under `--storage disk` is **untested**. Ruffle also uses its own folder layout, so a real save cannot be used directly; any later save test must use a disposable copy and a disposable save directory.

## 8. Stray process and downloaded files
- Stray `ruffle` process (PID 14592): parent PID 18152 (my first test launch, the Vulkan run), created in the same second, 0 threads, 0 handles, ~24 KB; it could not be queried (path, owner and command line unavailable) or stopped (access denied) and was not force-terminated. The Windows crash record for PID 18152 (Vulkan driver crash) explains the origin. It looks like a terminated process whose kernel object is still held open by something I could not identify (the holder is unknown). It is expected to disappear on sign-out or reboot.
- Files Ruffle created outside the lab: `%LOCALAPPDATA%\ruffle\recents.toml`, `log\ruffle.log`, and `video\openh264-2.4.1-win64.dll` (974,944 bytes, Authenticode valid, signer Cisco WebEx LLC, file version 2.4.1.2401). Purpose: Ruffle's optional H.264 video decoder, downloaded automatically from Cisco's server on first run. This game uses no video. Nothing was deleted. For the extra test runs I used isolated config folders in the lab, pre-seeded with a copy of the same DLL, and saw no further downloads.
- Windows Error Reporting also wrote crash-dump files for the Vulkan crash under `C:\ProgramData\Microsoft\Windows\WER`; untouched.

## Step 4 (licensed runtime): not needed
Skipped by design: the condition was "if Ruffle still cannot reach the intro or menu", and it can. The original EXE was not run and not modified.

## Remaining risks
- Gameplay is untested. A forced run of the game frame showed `Error #2067: ExternalInterface is not available` (the Zinc wrapper path, `GameController.exe = true`). Ruffle has `--dummy-external-interface`, untested here, and `exe` can be set false in a test copy if needed.
- Ruffle's AVM2 and rendering behavior may differ from Flash Player in gameplay (hit-testing, filters, sound); the oldest build tested fails entirely, so the Ruffle version must be pinned.
- Save persistence under Ruffle is unverified (see 7).
- Rebuilt classes beyond the menu are unexercised.

## Recommended next step (needs your approval; Stage 4 not started)
A short acceptance check of the test bed, still without feature work: start a New Game in the pinned runtime (newest build, `dx12`, `--load-behavior blocking`, `--dummy-external-interface` if required), play to the first dawn, and check for runtime errors; then test checkpoint save persistence with `--storage disk` against a disposable save directory and a disposable copy of a save. If that passes, Stage 4 can begin with the 60-night foundation from the Stage 2 plan, with every change checked by a Classic-mode regression comparison.

## Results by category
- **Passed:** original reaches intro and menu (2 of 3 builds, flags only); rebuilt reaches intro and menu (6/6 repeat runs); root cause of the stall identified; originals, backups and real save unchanged.
- **Failed:** the oldest build (2025-10-14) in every load mode; Vulkan backend (driver crash).
- **Skipped:** licensed-runtime and original-EXE tests (not needed); gameplay; save persistence; the stray process cannot be stopped.
- **Unverified:** gameplay under Ruffle; the recompiled gameplay classes at runtime; `--dummy-external-interface`; Ruffle `--storage disk` persistence.
