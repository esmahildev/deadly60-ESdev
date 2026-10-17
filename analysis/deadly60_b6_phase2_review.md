# Deadly 60 — B6 Phase 2: lab skeleton, generator source and static review

Date: 2026-10-17. Branch: `stage-5/b6-phase2-review`, created from `stage-5/b6-phase1-design` at 7a0a39795713f95df7d71a68a9278ffee8b13271 (clean tree, remote head identical). Not merged into `main`. Documentation only in Git: the lab skeleton and the generator source stay in the lab and are not in this repository.

**Status: Deadly 60 is not runtime-verified.** Phase 2 prepared a lab skeleton and generator source and reviewed them statically. **The generator has not been executed. No synthetic save exists. No Ruffle, Java or original EXE was launched. No real save, backup, save metadata or `acceptance\savetest` was accessed. No game code or binary was modified.**

## 1. Decision record: Continue counter, Option A

The owner chose Option A: the original Deadly 30 Continue counter behavior is **preserved** in Deadly 60, and the optional victory-counter fix is **not implemented**. This is documented as a **preserved original-game quirk, not a new fix**.

What the quirk is (source-derived, see the Phase 1 design; not yet runtime-observed):
- A Continue loads the checkpoint through a path that skips the checkpoint write and the counter increment and sets `Tracker.nightsLasted` to `nightCount − 1`.
- The counter therefore stays one lower than in an uninterrupted run for the rest of that run. A **win after Continue can end with `nightsLasted` = 59 and the menu text "You Lasted 59 Nights"**, whereas an uninterrupted win reaches 60 and shows "Survivors have escaped!". The original shows "You Lasted 29 Nights" for a continued win and the same message at exactly 30 for an uninterrupted one. A death after a Continue likewise shows one night fewer than an uninterrupted run would.
- The Stage 3 menu thresholds (30 → 60) preserve exactly this behavior. Nothing was changed for this decision.

## 2. What was created (lab only, `D:\Projects\deadly30_lab\game\d60_rollout\SAVE\`)

| Path | State |
|---|---|
| `inputs\` | empty (no synthetic save exists) |
| `run\` | empty |
| `sessions\` | empty |
| `tools\gen_synthetic_save.py` | generator source, 12,949 bytes, SHA-256 `DA71455AC047AABB011CD19014DD68AAA2B84CEFA5102B8A8D288AB9A334F1E0`; **not executed** |
| `ISOLATION.txt` | plain-text boundary statement |

No launcher, preflight script or test fixture exists yet.

## 3. Generator design (Route B; the container format is an unverified assumption)

- **No input files.** The only file operation in the source is one exclusive-create `open(path, "xb")`; nothing is read, listed, appended, moved or deleted.
- **Fixed interface.** Only `--case <id>` (a fixed table: SV0–SV6, NIGHT_05/10/25/34/35/55/59, KEY_30/59/60) and `--write`. Without `--write` it is a dry run that builds, self-checks and prints, and writes nothing. No path, directory or environment value can be supplied.
- **Computed output paths.** `<SAVE>\inputs\<CASE>.sol` and `<CASE>.manifest.txt`. It refuses if the path is not directly inside `inputs`, contains a forbidden fragment (`appdata`, `macromedia`, `sharedobjects`, `backup_saves`, `acceptance`, `savetest`, `\users\`, `\windows\`), has a non-directory or a reparse point (junction or symlink) anywhere from the drive root down, or already exists.
- **In-memory proof before any write.** It builds the shared-object bytes, re-parses them with a strict parser (length field = size − 6, tag, padding, property terminators, no trailing bytes) and compares the parsed values with the intended ones. The `saveString` is validated against the loader's expectations: six sections (seven only for the extra-section case), ten integer fields in section 0, equal numbers of soldier skins and equipped ids, empty box and turret sections, weapon ids 1–12 for valid cases, and exactly one declared violation for the deliberately invalid cases (night 31, unknown equipped id, unknown bought id, extra section).
- **Layout assumed [A]:** `00 BF` | 4-byte big-endian length | `TCSO` | `00 04 00 00 00 00` | 2-byte name length | `so_deadly301` | `00 00 00 00` | per property: 2-byte name length, name, AMF0 value (number `00` + double, or string `02` + 2-byte length + bytes), `00`. Two properties are written: `nightsLasted` (number) and `saveString` (string); every other key the game reads is guarded.
- **No cleanup.** If the second file fails after the first was created, it reports and stops; it never deletes or replaces anything.

## 4. Static review (no execution)

| Check | Result |
|---|---|
| Syntax, with warnings treated as errors | pass (a docstring escape warning found in the first pass was fixed with a raw docstring) |
| Imports | `argparse, hashlib, os, stat, struct, sys` only |
| `open()` calls | exactly one, line 228, mode `xb` (a later edit added a try/except around the writes; the check was repeated on the final file) |
| `os` attributes | only `path.*` (normcase, normpath, join, dirname, splitdrive, lexists), `lstat`, `fsync` |
| Forbidden calls (environment, listing, walking, remove/rename/replace/mkdir, subprocess, socket, urllib, pathlib, glob, eval/exec/compile) | none |
| Read-like method calls | none |
| Executing top-level statements | none (definitions and constants only; run guard present) |
| Command-line arguments | exactly `--case` and `--write` |
| Files in the skeleton after the review | no cache files; `inputs` is empty |

Manual review findings (all addressed or accepted): (a) a failure while writing the second file would otherwise surface as a raw traceback, so a handled error path was added (exit code 3, nothing cleaned up); (b) the dry run still checks the lab directories' metadata and the targets' existence (in-lab only), which is intended, so a dry run already refuses an existing target; (c) the check-then-create gap is harmless because the create is exclusive and fails if anything appeared; (d) a lab that sits behind a junction would be refused (fail closed); (e) all content is ASCII.

Limits of this review: it is source reading and syntax-tree inspection. It proves what the source can and cannot do, not that the bytes are what the game's runtime accepts.

## 5. Independent reference for the first output (computed in memory; nothing written)

A second implementation, written separately from the generator and fed with the case data read from the generator's syntax tree, gives for **SV0** (night 12, `nightsLasted` 11, saveString `12,5075,1,1,1,1,15,200,18,0|0|2|1,2,3||`):

- 113 bytes; the length field is 0x0000006B (107 = 113 − 6);
- SHA-256 `0D15E321EB5CDD2A99437F7E6F86F862873BB81F82729DE97E665BCF50D6FC2D`;
- head bytes `00 bf 00 00 00 6b 54 43 53 4f 00 04 00 00 00 00 00 0c 73 6f 5f 64 65 61 64 6c 79 33 30 31 00 00 00 00 …`.

Other reference values: SV1 `53318EA5…7F6B85`, SV2 `AC43FAEA…0F70F4`, SV3 `215F925C…B0D0D6` (all 113 bytes), SV4 `A1208B82…519D` (114), SV5 `DAB85D16…B48048` (116), SV6 `3D84E08D…FE1B9E` (115). When the generator is later run, its output for each case must match these exactly; any difference stops the work. Both implementations come from the same understanding of the format, so a match shows consistent coding, not that the format is correct.

## 6. Proposed first run (needs the owner's separate approval; nothing below has been done)

1. **Dry run:** `python -I D:\Projects\deadly30_lab\game\d60_rollout\SAVE\tools\gen_synthetic_save.py --case SV0`. Expected: it prints `case=SV0 intent=valid bytes=113 sha256=0D15E321…FC2D`, the saveString above, `self-check=PASS`, the two target paths, and `DRY RUN: nothing was written`. Writes nothing. Any mismatch with section 5, a refusal or an error: stop and report.
2. **First write (only after the dry run matches):** the same command with `--write`. Expected writes: exactly two new files, `inputs\SV0.sol` (113 bytes) and `inputs\SV0.manifest.txt`. Expected output ends with `WROTE: …`. Nothing else is created, read or changed.
3. **Isolation:** the generator has no input-file path and no network or environment use; the outputs are inside the lab skeleton; the file is not used by any game until a later, separately approved session.
4. **Cleanup:** none is performed by the tool. The two files stay as evidence; deleting them needs separate approval.

## 7. What still comes before SV0 itself (Phase 2c and 3)

- A launcher and preflight script (source only, statically reviewed, a preflight-only run with no Ruffle) as designed in the Phase 1 report: SWF copy in `run\` checked by hash, a new `sessions\<Tag>\` folder, `--storage disk` with `--save-directory` inside it, the synthetic file copied into the Ruffle layout, free memory ≥ 2.00 GiB, one window, graceful close.
- Only then the SV0 run on the original SWF, with the expected load trace (`LOADED STRING`, then `DAY!`, no `SAVE STRING`) and the menu text. Route A (a Ruffle-authored fresh-game file) remains the fallback if the format check fails.
- Synthetic results will never be presented as compatibility with the owner's real saves, and the Night 60 finale still requires direct human play.

## 8. Phase 2b result: SV0 dry run (added after the owner authorized only the dry run)

Executed once, with no `--write`, with Ruffle, Java, the launcher and the original EXE not involved:

- **Command:** `python -I -B "D:\Projects\deadly30_lab\game\d60_rollout\SAVE\tools\gen_synthetic_save.py" --case SV0`; **exit code 0**.
- **Before running:** the generator hash equalled the reviewed value (`DA71455A…F1E0`); its path is inside `SAVE\tools\`; the source shows the only `open` call inside `write_new`, reached only after the `if not args.write` branch returns (line 268-270); `inputs\` was empty.
- **Output:** `case=SV0 intent=valid bytes=113 sha256=0D15E321EB5CDD2A99437F7E6F86F862873BB81F82729DE97E665BCF50D6FC2D`, `saveString=12,5075,1,1,1,1,15,200,18,0|0|2|1,2,3||`, `self-check=PASS`, the two computed target paths, and `DRY RUN: nothing was written`. No `WROTE` line.
- **Comparison with the independent reference (section 5):** size 113 = 113 and SHA-256 equal; the saveString equals the reference. **Match.**
- **After running:** `inputs\` still empty (0 items); `SV0.sol` and `SV0.manifest.txt` do not exist; no cache directory was created under `SAVE`; the generator file is unchanged; no Ruffle or java process.

What this shows: the generator's builder and the independent second implementation agree byte for byte for SV0, and the dry run writes nothing. What it does not show: that the container layout is what Ruffle or the game accepts (still an unverified assumption, decided by SV0 itself). **Deadly 60 is not runtime-verified.**

## 9. Phase 2b result: SV0 written (owner authorized writing the SV0 files only)

- **Preconditions checked first:** branch `stage-5/b6-phase2b-dryrun` at 7590dc4bd6f9dda3d3e0f30b3fffc05b64680032 with the same remote head; generator SHA-256 `DA71455A…F1E0` unchanged; `inputs\` empty; neither target existed; the source still has the exclusive-create `open` (the only code-level `open` call), the path check with the reparse-point test, the existing-target refusal and the dry-run guard; no Ruffle or java process.
- **Command:** `python -I -B "D:\Projects\deadly30_lab\game\d60_rollout\SAVE\tools\gen_synthetic_save.py" --case SV0 --write`; **exit code 0**. Output: `case=SV0 intent=valid bytes=113 sha256=0D15E321…FC2D`, the saveString `12,5075,1,1,1,1,15,200,18,0|0|2|1,2,3||`, `self-check=PASS`, the two target paths and `WROTE: … SV0.sol and … SV0.manifest.txt`.
- **Created exactly two files** under `SAVE\inputs\`: `SV0.sol` (113 bytes, SHA-256 `0D15E321EB5CDD2A99437F7E6F86F862873BB81F82729DE97E665BCF50D6FC2D`, equal to the reference; ordinary file, not a reparse point) and `SV0.manifest.txt` (296 bytes). `inputs\` holds 2 items; no cache directory; no other file appeared under `SAVE`; the generator and `ISOLATION.txt` are unchanged.
- **Manifest:** names case SV0, intent valid, build original, shared object `so_deadly301`, `nightsLasted=11`, the saveString above, `bytes=113`, the same SHA-256, and the line that the container format is assumed and unverified until SV0 runs.
- **Independent parse of the written file** (our own synthetic file, in memory): header length = file size − 6, tag `TCSO`, name `so_deadly301`, properties `nightsLasted` = 11.0 and `saveString` as above, every byte consumed.
- **Deviations:** none. (A first attempt to run the precondition check failed on a quoting error in my check script before anything ran; it touched nothing and was re-run.)
- **Not done:** no Ruffle, Java, launcher or original EXE; no SV0 runtime test; no other case written; no real save, backup, metadata or `acceptance\savetest` access; no change to game code, binaries or the generator.
- **Not claimed:** that Ruffle or the game accepts this container layout. That remains unverified until the SV0 run. The generated files are lab artifacts and are not in Git. **Deadly 60 is not runtime-verified.**

## 10. Phase 2c: launcher and preflight source, static review (not executed)

Lab file `SAVE\tools\launch_sv.ps1` (18,152 bytes, SHA-256 `5AE909AECC4053F78614429B3982D0889BCF0C0169C0341EB451ECD0C71FE2A4`). **Never run.** `sessions\` is empty.

- **Interface:** `-Build original|candidate`, `-Case <id from the generator table>`, `-Tag <new name>`, exactly one of `-PreflightOnly` / `-Launch`. No free path parameter exists; every path comes from constants.
- **Preflight (all collected before anything is created):** tag format; every planned path inside the SAVE root and free of forbidden fragments; the session folder does not exist; no reparse point in the sessions and run paths; source SWF and its run copy have the pinned hash (original `3B9ABEE6…FC9C`, candidate `155A5167…304D`); the synthetic `.sol` and its manifest exist, the manifest hash equals the file and its case equals the request; Ruffle, the input helper and the video library match their pinned hashes; the Ruffle arguments contain `--storage disk` and the explicit `--save-directory` inside the session; no live Ruffle process or window and no java process; free memory ≥ 2,097,152 KB (3 rechecks, 20 s apart, then stop). `-PreflightOnly` then creates only the new session folder and `preflight.json`.
- **Launch (all create-new):** copy the SWF into `run\` (verified by hash), copy the synthetic file into the Ruffle layout `<session>\savedir\localhost\<SWF path without drive>\so_deadly301.sol`, start one Ruffle window with the verified flags, wait for the menu to appear **on its own** (no key is sent before it, to avoid the original game's known late-Space `#1009` loop), screenshot, click CONTINUE through the guarded input helper, wait for the loader trace and `DAY!`, screenshot, then close gracefully (3 requests of 20 s; never killed). It records traces, errors, hashes before and after (the session copy of the `.sol`), memory, session file list and a classification.
- **Pass criteria (source-derived expectations, to be observed):** the menu trace shows `NIGHTS LASTED: 11` (proves the synthetic container was read), `LOADED STRING = …` equals the input, `DAY!` appears, no `SAVE STRING`, no AVM2 errors, empty stderr, graceful close. A menu showing 0 is classified as the container not being read.
- **Static review (parse only, then a syntax-tree audit):** 0 parse errors; ASCII only; no `Remove-Item`, `Move-Item`, `Rename-Item`, `Stop-Process`, `Invoke-Expression`, web calls, `Set-Content`/`Add-Content`/`Out-File`, `Add-Type` or jobs; no `.Kill(`, no `taskkill`; `CloseMainWindow` present; exactly one `Start-Process` (Ruffle); three `Copy-Item` calls (SWF source to `run\`, the synthetic file into the session, screenshots into the session); read-type commands only on the manifest, the run's own log, the session listing and file hashes; the only environment variable referenced is `RUST_LOG`; no line names a real-save location outside the forbidden-fragment list; the only external command is the hash-pinned input helper. Noted: `-Force` appears once, only to read directory attributes in the reparse-point check.
- **Limits:** the review shows what the script can and cannot do; its logic has never been executed, so defects can still show up in the preflight-only run, which is why that run comes first. Expected side effects outside the session: new PNGs in the harness `runtime_tmp\shots` folder (named with the tag) and Ruffle's own config folder updates, as in the earlier runs.

## 11. SV0 runtime result (original build): PASS, the synthetic file format is accepted

Run under the owner's explicit approval of the two-step sequence. Verified before running: launcher `5AE909AE…FE2A4`, generator, `SV0.sol` (113 bytes, `0D15E321…FC2D`) and manifest unchanged; free memory 2.78 GiB.

**Step 1, preflight only** (`launch_sv.ps1 -Build original -Case SV0 -Tag V1_SV0_ORIG_PF -PreflightOnly`): **PASS**, exit 0, nothing launched; only `sessions\V1_SV0_ORIG_PF\preflight.json` was created; no problems listed; memory gate 2.88 GiB. (An earlier attempt exited with code 5 at the memory gate, with free memory at 0.47 GiB, and created nothing.)

**Step 2, runtime** (`launch_sv.ps1 -Build original -Case SV0 -Tag V2_SV0_ORIG_RUN -Launch`): exit code 0, classification `LOAD_OK`. Observed in Ruffle on the original SWF (pinned hash, copy in `SAVE\run\`), with in-lab disk storage only:

| Criterion | Observed |
|---|---|
| Menu shows the synthetic key | trace `NIGHTS LASTED: 11`; the menu screenshot reads "You Lasted 11 Nights" with CONTINUE enabled |
| Loaded string equals the input | `LOADED STRING = 12,5075,1,1,1,1,15,200,18,0\|0\|2\|1,2,3\|\|` (identical) |
| No `SAVE STRING` | none in the trace; the load trace is `LOADED STRING`, then `DAY!` |
| No AVM2 errors | 0 error lines, stderr empty |
| Graceful close | one close request sufficed; no Ruffle, window or java process afterwards |
| Restored state | the screenshot after Continue shows the HUD "DAWN 12", scrap 5,075, 200 rifle-class ammo, 1 medkit, weapon MP-40 30/30 and the safe room ("Sleep Till Night"); the loader's enemy-unlock traces `NEW ENEMY:1,2,3,4,6,5` appeared |

What this establishes: the hand-built container (Route B) **is read by Ruffle and by the original game**, Continue loads a night-12 checkpoint from disk storage, and the source-derived load trace (no checkpoint rewrite) matches what happened. This is the first runtime observation of the Continue path. It is the original build only; it says nothing yet about the candidate, nights above 12 or the counter after a death, and nothing about the owner's real saves.

Side observations: Ruffle rewrote the session's own copy of the file at exit (124 bytes against the 113-byte input, a Ruffle-authored serialization of the same data); the input in `inputs\` is unchanged; the lowest free memory during the run was 1.98 GiB (the gate applies at launch). No key was sent before the menu appeared on its own.

Evidence (lab only, not in Git): `SAVE\sessions\V2_SV0_ORIG_RUN\` with `result.json` (`97914CC9…`), `traces.txt` (`E0D1C75A…`), `ruffle.out.txt` (`C925536…`), `V2_SV0_ORIG_RUN_menu.png` (`BFCB9D9D…`), `V2_SV0_ORIG_RUN_loaded.png` (`DCB06AB7…`). Launcher, generator, `SV0.sol` and manifest remained unchanged. No real save, backup, metadata or `acceptance\savetest` was accessed; no original EXE or second runtime instance was launched. **Deadly 60 is not runtime-verified.**

## 12. Approval requested now

1. (Done: dry run in section 8, write in section 9.) The two SV0 files exist.
2. (Done in section 10.) The launcher and preflight source exist and passed the static review.

3. (Done in section 11.) SV0 passed on the original build.

**One approval needed for the next step (the rest of the matrix with the same reviewed tools):** (a) write the remaining synthetic cases with the generator (`--write`, one case at a time, create-new, in `SAVE\inputs\`); (b) original-build cases SV3, SV4, SV5, SV6, SV1, SV2 (predicted failures and boundaries); (c) candidate-build cases NIGHT_05, NIGHT_25, SV1/SV2, NIGHT_34, NIGHT_35, NIGHT_55, NIGHT_59 and KEY_30/59/60, each as preflight-only then launch with a new tag, one Ruffle window at a time under the memory gate. The Night 60 finale itself will still need a person to play it.
