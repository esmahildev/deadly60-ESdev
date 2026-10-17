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

## 8. Approval requested now

1. Run the generator **dry run** for SV0 (writes nothing), and, if it matches section 5, the `--write` run that creates the two SV0 files.
2. Permission to write the launcher and preflight **source** and review it statically (no execution), as Phase 2c.

SV0 itself (any Ruffle launch) will be requested separately.
