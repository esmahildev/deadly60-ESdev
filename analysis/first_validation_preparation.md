# Deadly 30 — First Safe Validation Stage: Preparation (Planning Only)

Status: **PLANNING ONLY. Nothing in this document has been executed.** No FFDec, Ruffle, Java, build or experiment was run for it, and no source, SWF, asset or save file was changed.
Written for: the project owner, as the gate before the first validation runs.
Date: 2026-10-17. Contains no game code, no assets and no save contents.

Evidence tags: **V** verified in a file or by a read-only command during this review · **P** stated in an earlier report, not re-checked · **A** assumption or inference (not tested) · **REC** recommendation.
Every command below is marked **NOT EXECUTED**.

Approved direction this plan follows: one game mode, no mode selector, no second save slot; original behavior preserved by default; nights 1–29 unchanged and new content gated to night 31+; the real save untouched and all tests on isolated copies; the smallest justified recompile list per patch; code-drawn visuals only after a reviewed proof of concept; `Nights.finalNight()` not implemented yet.

---

## 0. Summary

Two validation activities are prepared:

- **A. Controlled C5 re-test.** C5 is the "Nights only" recompile that logged 36 `#1009` errors in a confounded batch. Raw evidence shows those earlier runs were also hit by **GPU out-of-memory panics** and stale windows (section A1). The new test is designed so that environmental interference makes a run *invalid* instead of being mistaken for a recompile problem.
- **B. Save-safety tests on the ORIGINAL build with synthetic saves.** They settle, without any modified build, what the original game does with a night-31 checkpoint, an unknown equipped-gun id, an unknown bought-weapon id and an appended section. These answers decide how a future modified build may safely write saves.

Both need your approval before any run. **Free memory at the time of this review was 0.51 GB (and 1.16 GB earlier in the session)**, far below the 2.0 GB launch threshold, so neither could start today without you closing other applications first.

---

# PART A — Controlled C5 re-test preparation

## A1. What the documented C5 failure actually is

| Fact | Evidence |
|---|---|
| C5 = `E1\controls\C5_nights_only.swf`, SHA-256 `F495D256FDCD6A7D77845EA20878D0022A0963BFF20E29AEE439852CDF6A19A8`, 19,252,727 bytes (original 19,250,808 + 1,919) | V: `E1\cmp1009_build_hashes.txt`, `controls\probe_C5_nights_only\result.json` |
| Run at about 12:16 local, on the second control batch, 3 minutes after C4 | V: `probe_C5_nights_only\command.txt` timestamp; `probe_C4_…` at 12:13 |
| Result: 36 ERROR lines, all `#1009` "Error running AVM2 frame script", at roughly 0.6–1.4 s intervals; only 4 trace lines (menu reached); `started=false`, `exitedGracefully=false` | V: `result.json`, `errors.txt` (36 lines) |
| **`ruffle.err.txt` (260 bytes) contains a panic: `wgpu error: Out of Memory`** | V |
| Same panic in the **original (C0)**, C2, C3, C4, C6; C1 shows `OutOfMemory` in `wgpu-core … queue.rs`. **7 of the 8 runs in `controls\` have an OOM panic on stderr, including the original, which had 0 `#1009` errors.** C6b (repeat of the 18-class build) had empty stderr and still 23 errors. | V |
| The GPU is an integrated Intel HD Graphics 520 (shares system RAM) | V: Ruffle log line "Using graphics API dx12 on Intel(R) HD Graphics 520 (type: IntegratedGpu)" |
| Stale Ruffle windows were open during that period (two stale screenshots of other window PIDs at 12:30) and free memory was 1.1–1.4 GB | V: `E1\stale_*.png`; P: Stage 0b report |
| A later set, `controls2\`, had **0 errors** with empty stderr for the original (r1) and for the **18-class harness build (r1, menu reached, graceful exit)**; the recompiled C1 run (r1) never reached the menu (2 trace lines, so incomplete) | V: `controls2\*\result.json` |
| The Stage 0b report's historical record does **not** list the OOM panics or the `controls2` runs | V (I did not edit that report) |
| The C5 build recipe is **not recorded as a script anywhere**; only the name, hash and size exist | V: searched the spike folders |

Interpretation (A): the earlier `#1009` counts cannot be attributed to the recompile. OOM presence alone does not explain them either (the original had a panic and 0 errors; C6b had errors and no panic), so the confounding is plural and the earlier data can neither convict nor clear C5. Only a clean controlled run can.

## A2. What the controlled O1/C1 pair established (V)

- O1 = `E1\input.swf` (`3B9ABEE6…FC9C`); C1 = `E1\controls\C1_both_unmodified.swf` (`B2CBCC68…7BF8`, `Nights` + `Deadly30_fla.MainTimeline` recompiled from unmodified sources).
- Both: `COMPLETED + closed + exit verified`, 0 ERROR lines, empty stderr, all six flow steps reached, 35 trace lines. Free memory before 2.36 / 2.39 GB; peak working set 945 / 911 MB; about 82 s from Level Created to game over.
- The two 35-line trace sequences are identical. Digest (SHA-256 of the lines joined with LF, UTF-8): **`2E8A7B07275647BC53CE55DA570329B98E8B9DA8ABBEB7D58CA0D70354055ED1`** for both (V, computed read-only from the two `result.json` files).
- One sample per build; says nothing about C5 specifically. (V)

## A3. Exact inputs for a Nights-only re-test

| Item | Path | Expected identity |
|---|---|---|
| Control (original), same file as O1 | `D:\Projects\deadly30_lab\game\spike_0b\E1\input.swf` | SHA-256 `3B9ABEE6E4E760522D43CC2BD4CB5D3C5F54E44CDC22C397B0F5313C0AC3FC9C`, 19,250,808 bytes |
| Candidate C5 (existing build) | `D:\Projects\deadly30_lab\game\spike_0b\E1\controls\C5_nights_only.swf` | SHA-256 `F495D256FDCD6A7D77845EA20878D0022A0963BFF20E29AEE439852CDF6A19A8`, 19,252,727 bytes |
| C5 source (the only recompiled class) | `D:\Projects\deadly30_lab\game\src_baseline\com\iggyzuk\deadly30\Nights.as` | SHA-256 `88EB7C280FF4D97E8BEFF2DEC8A81B56B9175A27F665259D355B3D1309D414C3`, 10,474 bytes (V) |
| Recompile class list | exactly one class: `com.iggyzuk.deadly30.Nights` | (A: that this is what C5 was; see A4) |
| Ruffle | `D:\Projects\deadly30_lab\runtime\ruffle\ruffle.exe` | SHA-256 `A4C96B927B3826F6A0E2CBC871B71B7D7561B9EB760B020AFAAEA638E15243E7`, version `0.8.0-nightly.2026.10.16` (V) |
| FFDec / Java (only for the recipe check) | `D:\Projects\deadly30_lab\tools\ffdec\ffdec.jar` | SHA-256 `1587E60B2EC2C0E4F5721FD5F2AAAF2379F6D416569DE99054B5C5D1E9BA841C`, v26.3.0; Java 1.8.0_503 (V: S0a_A manifest) |
| Ruffle config/cache (shared with O1/C1, for comparability) | `D:\Projects\deadly30_lab\game\harness\ruffle_cfg` | file list recorded before/after |
| Flags (identical to O1/C1) | `--graphics dx12 --storage memory --load-behavior blocking --dummy-external-interface --no-gui --width 750 --height 460 --config <ruffle_cfg> --cache-directory <ruffle_cfg>`, `RUST_LOG=info` | V: `cmp_pair\O1\command.txt` |

## A4. Step A-0: confirm the C5 recipe (needs FFDec = a Java run; **NOT EXECUTED**)

Goal: prove that the existing C5 file is exactly "unmodified baseline `Nights.as` recompiled alone", rather than trusting its name. C1's recipe was reproduced byte-identically earlier, which suggests FFDec output is deterministic (V), but C5 has not been checked.

```
# NOT EXECUTED. Writes only to a NEW folder; never to controls\.
java -Xmx3g -jar D:\Projects\deadly30_lab\tools\ffdec\ffdec.jar -replace ^
  D:\Projects\deadly30_lab\game\spike_0b\E1\input.swf ^
  D:\Projects\deadly30_lab\game\spike_0b\E1\cmp_c5\verify\C5_rebuild_check.swf ^
  com.iggyzuk.deadly30.Nights ^
  D:\Projects\deadly30_lab\game\src_baseline\com\iggyzuk\deadly30\Nights.as
```
(The argument layout matches `harness\build.ps1` lines 72-75, V.)
Expected: output SHA-256 equals `F495D256…19A8`. Then run the existing full-tag hashing tool (`spike_0b\taghash.ps1`, PowerShell, no Java) between `input.swf` and the output: expected exactly 1 differing tag (`DoABC2`).
- If equal: use the existing C5 file as the candidate.
- If different: the old C5's recipe is unknown. Use the rebuilt file as "C5′" (a new build with a documented recipe) and treat the old C5 file as history. The test then answers the question for the rebuilt file.
Run this step **separately and first**, with no Ruffle window open and at least 2.5 GB free (JVM heap is capped at 3 GB; A).

## A5. Test design (Ruffle; **NOT EXECUTED**)

**Session root (must not exist yet):** `D:\Projects\deadly30_lab\game\spike_0b\E1\cmp_c5\`. Each run gets its own new folder `cmp_c5\<RunTag>\`; the session folder also gets `hashes_BEFORE.txt`, `hashes_AFTER.txt`, `session_summary.txt`.

**Do not reuse the existing scripts unchanged (V, hazards):**
- `E1\cmp_pair\run_one.ps1` builds its output folder as `cmp_pair\<Tag>`; running it again with tag `O1` or `C1` would **overwrite the O1/C1 evidence** (`New-Item -Force`, `Set-Content`).
- It also runs `Remove-Item "$H\runtime_tmp\shots\*"` (line 18), which deletes earlier screenshots.
- `cmp_pair.ps1` is hard-wired to O1 and C1.
So a **new copy** (for example `cmp_c5\run_one_v2.ps1` and `cmp_c5\driver.ps1`) is needed, with a new output root, a refusal if a run folder already exists or is non-empty, no `Remove-Item`, and the extra instrumentation below. Creating them needs your approval; they are not created now.

**Run order (decision needed):**
- **Design 1 (minimum, same rules as O1/C1):** `O` then `C5`. Two windows, one sample each.
- **Design 2 (REC):** `O`, `C5`, `C5`, `O` (an ABBA-type order). Four windows, one at a time, about 2 minutes each plus settling, about 10–12 minutes. It gives two C5 samples (to see intermittency), two original runs (a negative control that shows the session's own noise) and balances order or time drift.
- Tags for Design 2: `R1_O`, `R2_C5`, `R3_C5`, `R4_O`. Never `O1` or `C1`.
- No automatic repeats. At the first INVALID or unexplained error the driver stops and reports.

**Per-run protocol (identical to `run_one.ps1`, V lines 24-47):** wait for window; `front`; Space to skip the intro; wait for `NIGHTS LASTED`; 10 s menu dwell; real-cursor click NEW GAME at stage (140, 245); wait `ADD CUTSCENE`; 3 s; Space; wait `Level Created`; unattended until `ADD GAME OVER SCREEN NOW!`; wait for the second `NIGHTS LASTED`; 2 s; `CloseMainWindow()`, wait up to 60 s; no force-kill.

**Extra instrumentation (read-only; REC, needs approval):**
1. Before each launch: `Get-CimInstance Win32_Process` for `ruffle.exe`/`java.exe` (PID, ThreadCount); list of the 10 largest processes by working set; free physical memory; GPU counters `\GPU Adapter Memory(*)\Dedicated Usage`, `Shared Usage`, `Total Committed` (V: the counter set exists on this machine; values not yet sampled).
2. During the run, every 5 s: free memory and the Ruffle working set (minimum free value is recorded).
3. After exit: same process list, free memory, and size/time of `ruffle_cfg\*` files.
4. The launch timestamp (UTC) is recorded so log freshness can be checked.
5. `RUST_LOG=info` stays (comparability). A more verbose log is only for a follow-up after a genuine failure.

**Preconditions (all must hold before the first launch):**
- You have closed heavy applications; free memory is at least 2.0 GB (the O1/C1 runs had 2.36–2.39 GB). **Currently 0.51 GB: not met.**
- No live `ruffle.exe` (ThreadCount > 0), no window titled `Ruffle*`, no `java.exe`. The 11 existing zero-thread `ruffle.exe` entries are dead process entries; they are recorded, not touched.
- `cmp_c5\` does not exist; hashes of the control, candidate, Ruffle and `ruffle_cfg` file list are recorded; the 13-entry protected-hash list, backup manifest (391), `src_baseline` (381) and the real-save **metadata** are recorded (section B4).
- You have been told before each batch that a window will open and the real cursor will move (the NEW GAME click uses the real cursor).

## A6. How a genuine recompile problem is distinguished from interference

A run is first classified **VALID** or **INVALID**; only valid runs count.

| # | Validity check | If it fails |
|---|---|---|
| V1 | SWF SHA-256 equals the expected value before launch and again after exit | INVALID (input changed) |
| V2 | No live Ruffle/Java and no `Ruffle*` window before launch; none appear that this run did not start | INVALID |
| V3 | Free memory at least 2.0 GB at launch | Nothing is launched (exit "memory") |
| V4 | `ruffle.err.txt` is **0 bytes** (no panic, no OOM) | INVALID; record the text |
| V5 | Run folder was new and empty; first log timestamp is later than the recorded launch time; log shows the dx12/Intel line; `command.txt` names the expected SWF | INVALID (stale or foreign output) |
| V6 | All six flow steps reached (started, menu, cutscene, levelCreated, gameOver, menuReturn) | INVALID (incomplete flow) |
| V7 | Graceful close and exit verified within 60 s; no live process or window afterwards | INVALID; **stop, do not force-kill, ask** |
| V8 | Minimum sampled free memory during the run at least 0.8 GB | "PRESSURE-FLAGGED": a clean result still counts; an error result is **inconclusive** |

Only the redirected per-run `ruffle.out.txt` is analyzed. The shared `ruffle_cfg\log\ruffle.log` is never used as evidence, because Ruffle appends to it across runs (V: it was modified by the previous runs).

**Interpretation of VALID sessions:**

| Observation | Conclusion allowed |
|---|---|
| All runs 0 errors, flow complete, key-marker trace sequence equal to the original's | "In this session a Nights-only recompile showed no `#1009` errors" (two samples under Design 2). Does **not** prove the recompile safe in general. |
| An original (O) run shows errors or a panic | The environment or baseline is unstable: the whole session is void, no conclusion about C5, investigate memory/GPU first |
| C5 errors in a valid, unpressured run while both O runs are clean and identical | A **genuine candidate problem**. Stop. No automatic repeat. Follow-up (needs approval): compare C5 against the known-clean C1 in the same session, diff the `DoABC2` contents, and capture a more verbose log to find the failing frame script |
| Errors in one C5 run but not the other | Intermittent: report the rate and conditions, conclude nothing |
| 0 errors but a different trace | Compare the two O runs first. Raw repeated lines such as `SHOW MOUSE` may vary with timing (A), so equality is judged on the ordered key markers: `NIGHTS LASTED`, `ADD CUTSCENE`, `REMOVE CUTSCENE`, ` --- RESTART GAME --- `, `Level Created!`, `ADD GAME OVER SCREEN NOW!`, second `NIGHTS LASTED` |

## A7. Expected outputs and acceptance criteria

Expected for a clean valid run (from O1/C1, V):
- `result.json`: `outcome = "COMPLETED + closed + exit verified"`, `totalErrors = 0`, `errorsByPhase` all 0, `flowComplete = true`, all six steps true, `closeRequested = true`, `exitedWithin60s = true`, `liveRuffleAfter = 0`, `ruffleWindowsAfter = 0`.
- `ruffle.err.txt` 0 bytes; `errors.txt` empty; 35 trace lines; peak working set about 0.9 GB; Level Created to game over about 80 s (information only).
- Reference trace for comparison: `E1\cmp_pair\O1\result.json` (`traces`), digest `2E8A7B07…55ED1`.

Acceptance of the **test** (not of C5): every run VALID; session hashes before = after; the real-save metadata unchanged; no unexpected process left. Acceptance of **C5**: 0 errors in every valid C5 run plus a clean O control. Anything else is reported as Failed or Unverified, never as Passed.

## A8. Log and evidence locations (future)

Per run, in `…\E1\cmp_c5\<RunTag>\`: `command.txt`, `ruffle.out.txt`, `ruffle.err.txt`, `result.json`, `errors.txt`, `preflight.json` (processes, top-10 memory, GPU counters, free memory, UTC launch time), `memory_samples.csv`, `postflight.json`. Per session: `hashes_BEFORE.txt`, `hashes_AFTER.txt`, `session_summary.txt`, `driver.out.txt`. Nothing is overwritten and nothing is deleted.

## A9. Stop conditions and cleanup

Stop immediately (no further launches, no force-kill) on: V3/V4/V5/V6/V7 failure; any protected-hash or backup mismatch; an unexpected Ruffle/Java process or window; a window that ignores close (record, ask); free memory below 0.8 GB during a run; your interruption.
Cleanup: **none is performed.** All run folders stay as evidence. Dead zero-thread `ruffle.exe` entries are not touched. Expected, non-protected changes: `ruffle_cfg\recents.toml` and `ruffle_cfg\log\ruffle.log`.

---

# PART B — Save-safety test preparation

## B1. Verified save facts

| Fact | Evidence |
|---|---|
| The real save is `%APPDATA%\Macromedia\Flash Player\#SharedObjects\RQ626TTB\localhost\so_deadly301.sol`, 311 bytes, last write 2026-10-14 16:30:08; it is the only file in that folder | V: metadata only (content **not** opened in this review) |
| A backup copy exists: `D:\Projects\deadly30_lab\backup_saves\so_deadly301.sol.2026-10-17.bak`, 311 bytes, SHA-256 `AFAF490BA5D33E72209F2E0C12E64C194A13F3C291469F2746060BB6D8B756E1`, which equals the real save's recorded hash | V (backup hashed; real-save hash from the earlier recorded value) |
| SharedObject keys in the file: `nightsLasted`, `lvl`, `quality`, `achievedMedals`, `achievedGuns`, `resolution`, `saveString` | V: read from the backup copy |
| Layout of `saveString`: sections separated by `\|`: 0 = night, scrap, medkits, armor, fence, boots, ammo×3, kills; 1 = soldier skins; 2 = equipped gun ids; 3 = bought weapon ids; 4 = boxes (`x,y:…`); 5 = turrets (`x,y,scaleX,type:…`). The backup's checkpoint is night 12. | V: backup copy; P: Stage 3b |
| The `.sol` is AMF0: the string value has a 2-byte big-endian length, and the file header has a 4-byte length at offset 2 (file size minus 6) | A: inferred from the byte dump (string length byte `0x7D` = 125 matches the string); to be verified by the generator's self-check |
| Ruffle disk layout: `<save-dir>\localhost\<SWF path without drive>\so_deadly301.sol`, keyed by the SWF's own path (example: `S3\localhost\Projects\deadly30_lab\work\input.swf\so_deadly301.sol`) | V: three existing disposable saves; P: Stage 3b |
| Ruffle writes only on a graceful exit; the game never calls `flush()` | P: Stage 3b |
| Loader indexes sections 0–5 unconditionally; sections beyond 5 are never read | V: Level.as:1805-1811 |
| Section 3 ids are matched against `Controller.weaponArray`; an unmatched id is ignored | V: Level.as:1851-1862 |
| Section 2: `equiptWeapon(soldier, id)` searches `weaponArray` for the id; if none matches, the target stays `null` and the next loop reads `_loc3_.type` | V: Level.as:2918-2945 |
| Last loop reads `waveArray[0 … nightCount-1]` and uses `.length` on each row; `waveArray` has 30 rows | V: Level.as:1888-1899; Nights.as (30 pushes, P count) |
| Weapon ids encode type and upgrade: `gunUpgrade = (id - type) / 3 + 1` | V: Soldier.as:876-893 |
| Menu: `nightsLasted` 1–29 and 30 have text; above 30 no text is set; Continue is enabled for any value ≥ 1 | V: MainTimeline.as:154-199 |
| "Clear Save Data" calls `clear()` on the whole object | V: ClearSaveDataButton.as:72-77 |
| The menu screenshot (750×460, 1:1 with stage coordinates) shows CONTINUE at about (140, 190) and NEW GAME at (140, 245) | V: `acceptance\shots\C2_menu.png`; the CONTINUE coordinate is an estimate (A) to be confirmed with a screenshot before clicking |

## B2. The three compatibility risks explained

**1. `waveArray[30]` (rollback hazard).** If a future build allows nights beyond 30 and writes a checkpoint with night ≥ 31, the *original* loader reaches `waveArray[30]`, which does not exist, and then calls `.length` on `undefined`. Expected: a TypeError (likely `#1010`, A) during Continue. The menu would still show an enabled Continue button (V). So a rolled-back original game would offer Continue and crash on it. Night 30 itself loops over indices 0–29 and should load, but what the game then does at night 30 (the finale logic, Level.as:340) is untested. The exact first failing statement and error code are **A** until SV3 runs.

**2. Unknown equipped-gun ids.** A new weapon id in section 2 would make the original `equiptWeapon` dereference `null` at Level.as:2937 during Continue (V code; the visible error is A). The same would happen to a modified build if a save names a weapon its table lacks. Unknown ids in section 3 (bought) are silently ignored (V). Unknown entries in the `achievedGuns` key are only compared against known ids in the code I read (`gunAchieved`, Level.as:3104-3118); `foundGun` (Level.as:728-756) was not examined, so that part is A. New weapon ids must also follow the `(id - type) / 3 + 1` scheme (V) or the upgrade math misbehaves.

**3. Modified build vs original build.**

| Direction | Question | Status |
|---|---|---|
| Modified build reads an **original** save | Must behave identically for valid saves | To be proven by the same saves run on both builds (later stage) |
| **Original** reads a **modified-build** save | Safe only if the modified build writes nothing the original mishandles | See table below |

What the original does with data a modified build might write:

| New data | Original behavior | Status |
|---|---|---|
| `saveString` night ≥ 31 | Fails at Continue | V code, A runtime (test SV3) |
| New gun id in section 2 | Fails at Continue | V code, A runtime (SV4) |
| New ids in section 3 | Ignored | V code (SV5 confirms) |
| Appended section 6 or later | Ignored | V code (SV6 confirms) |
| New SharedObject keys | Ignored; removed by Clear Save Data | V code |
| `nightsLasted` above 30 | Blank menu text, Continue enabled | V code |

REC (needs your decision after the tests): append only, never change sections 0–5; keep new weapon ids out of sections 2 and 3 unless the original's failure is acceptable; and choose between **(S-a)** one `saveString` that may exceed night 30 (simple, but a rolled-back original cannot load it) and **(S-b)** keeping `saveString` loadable by the original (nights ≤ 30) while a new key holds the extended state (rollback-safe, more code, two sources of truth). Neither is assumed.

## B3. Test plan (original build only; **NOT EXECUTED**)

No modified build exists, and none is needed: the questions are about the original loader. The same inputs are reused later to prove a candidate behaves identically.

**New folder (must not exist):** `D:\Projects\deadly30_lab\game\spike_0b\SAVE\` with `inputs\` (synthetic `.sol` files and `inputs_manifest.txt`), `run\` (a copy of the original SWF) and `sessions\<Tag>\` (disposable save dir, logs).

**Source of all test data:** only the backup copy `backup_saves\so_deadly301.sol.2026-10-17.bak` (SHA `AFAF490B…56E1`), copied to `SAVE\inputs\SV0_unchanged.sol`. **The real save is never read or written by any test.**

**Synthetic file generator (new PowerShell script, not written yet):** reads the backup, applies one patch, validates (header length = file size − 6; string length field = actual length; terminators intact), writes a new file, and records its SHA-256 and the exact differing byte offsets. Same-length patches need no length fix-up; SV6 needs both length fields updated.

| ID | Patch to the checkpoint | Purpose | Predicted result in the ORIGINAL (all A unless noted) |
|---|---|---|---|
| SV0 | none (byte-identical copy) | Baseline | Menu shows Continue; `LOADED STRING` equals the backup's string; 0 errors (V: the earlier acceptance run loaded this string) |
| SV1 | night `12` → `29` | Boundary below | Loads |
| SV2 | night `12` → `30` | Boundary | Loads (loop 0–29); behavior after load noted |
| SV3 | night `12` → `31` | `waveArray[30]` hazard | TypeError during load; Continue does not complete |
| SV4 | equipped ids `11,10,9` → `99,10,9` | Unknown equipped id | Null dereference during load |
| SV5 | bought ids: `11` → `99` in section 3 | Unknown bought id | Ignored; loads like SV0 (V code) |
| SV6 | a 7th section appended (for example `\|x`) | Trailing section | Ignored; loads like SV0 (V code) |
| SV7 (optional) | drop sections 4–5 | Truncated string | TypeError during load |

**Minimum set (REC):** SV0, SV3, SV4, SV5, SV6. SV1, SV2 and SV7 are optional.

**Run procedure per test (one Ruffle window at a time; same precondition and stop rules as section A5/A9):**
1. Place a fresh copy of the synthetic file at `<sessiondir>\savedir\localhost\Projects\deadly30_lab\game\spike_0b\SAVE\run\deadly30.swf\so_deadly301.sol` (A: path derived from the S3 layout) and record its hash.
2. Launch (NOT EXECUTED):
```
D:\Projects\deadly30_lab\runtime\ruffle\ruffle.exe --graphics dx12 --storage disk --save-directory D:\Projects\deadly30_lab\game\spike_0b\SAVE\sessions\<Tag>\savedir --load-behavior blocking --dummy-external-interface --config D:\Projects\deadly30_lab\game\harness\ruffle_cfg --cache-directory D:\Projects\deadly30_lab\game\harness\ruffle_cfg --no-gui --width 750 --height 460 D:\Projects\deadly30_lab\game\spike_0b\SAVE\run\deadly30.swf
```
   (`RUST_LOG=info`; stdout/stderr redirected to the session folder.) The script refuses to launch unless `--storage disk` is present and `--save-directory` lies under `…\spike_0b\SAVE\sessions\`.
3. `front`; Space (skip intro); wait `NIGHTS LASTED`; take one screenshot first (dry run) to confirm the CONTINUE coordinate; real-cursor click CONTINUE (about stage 140,190; A); wait up to 20 s for `LOADED STRING =`; observe a further 15 s; record every `ERROR` line with the preceding trace line.
4. Graceful close; verify exit; hash the disposable `.sol` again (A: unchanged, since the game only reads and closes it).
5. Evidence: `command.txt`, `ruffle.out.txt`, `ruffle.err.txt`, `result.json` (LOADED STRING text, error list, first error text, screenshots), hashes of the disposable `.sol` before/after.

**Fixed-path decision:** earlier rules used `game\run\deadly30.swf`, which today holds a harness build (19,273,450 bytes, V size only), not the original. Using it for the original would overwrite an existing file. REC: use the new fixed path above (a new file, the same path for every save test, so the path-keyed storage rule is still satisfied) and leave `game\run\deadly30.swf` alone.

## B4. Keeping the real save untouched, and proving it

- **Before and after every session** record, for the real save: full path, length (311), last-write time (2026-10-14 16:30:08), attributes, and the folder listing (one file). This review did only that (metadata), because you asked that the real save not be opened.
- **Content hash:** the recorded SHA-256 is `AFAF490B…56E1`. Re-computing it requires opening the file for reading. Earlier sessions did so (read-only). **Decision for you:** metadata only, or also a read-only hash.
- **No `.sol` anywhere else changes:** before and after, list every `*.sol` under `%APPDATA%` and `%LOCALAPPDATA%` and the lab with size and time; only files under `SAVE\sessions\` may appear or change.
- **Run only Ruffle with an explicit disk directory** inside the lab. Never start the Zinc EXE, never omit `--save-directory` on a disk run. (A: Ruffle's default storage is not the Flash Player folder; the explicit flags make this irrelevant.)
- **Protected set:** the 13-entry hash list, backup manifest (391), `src_baseline` (381), `patches\order.txt` and `hooks_order.txt`, the original SWF and its backups, and the backup save: all identical before and after.

## B5. Evidence and acceptance for Part B

Each test is reported Passed / Failed / Skipped / Unverified. "Passed" means the **test was valid** and the outcome was observed and recorded; a prediction that is not confirmed is a *finding*, not a failure. Valid = new session folder, input hash recorded, flow reached the menu, Continue clicked, graceful close and verified exit, empty stderr, protected set and real-save metadata unchanged.
Required findings: SV0 loaded string equals the backup string; SV3 outcome (first error text, error code, last trace line); SV4 outcome; SV5 and SV6 outcomes; plus the real-save evidence above.

## B6. Rollback

Part B changes no existing file, so there is nothing to roll back. If any protected file or the real save unexpectedly differs: stop at once, report the original output, and do **not** restore anything automatically. Restoration, only with your explicit approval, would come from the verified backups (`backup\Deadly30_game.ORIGINAL.swf`; `backup_saves\so_deadly301.sol.2026-10-17.bak`; the `src_baseline` manifest). For future modified builds the same rule holds: back up the real save first and record its hash; the only way back to a pre-change state of real progress is that backup.

---

# Risks of this preparation

1. Free memory is currently 0.51 GB; without your closing applications, the same OOM panics that spoiled the earlier runs would recur. (V)
2. Ruffle's window close is unpredictable; a stuck window cannot be force-killed without asking. (V)
3. Predictions in B3 are inferences until run; the CONTINUE coordinate and the `.sol` byte layout are estimates. (A)
4. Two samples per build cannot rule out a rare intermittent effect. (A)
5. The earlier `controls2` runs and the OOM panics are missing from the Stage 0b report; until it is amended, that report's "historical record" is incomplete (not edited here). (V)
6. Evidence for an Nights-only recompile does not extend to other classes; each later patch needs its own controlled comparison. (V)

# Decisions that still need your explicit approval

1. **Part A go-ahead:** approve the C5 session, and choose **Design 1** (O, C5) or **Design 2** (O, C5, C5, O; REC).
2. **FFDec recipe check (A-0)**: approve one FFDec run (Java) into a new folder, before the Ruffle session.
3. **New lab scripts:** approve creating `cmp_c5\run_one_v2.ps1` and `driver.ps1` (copies with new output roots and extra read-only instrumentation) and the folder `cmp_c5\`.
4. **Instrumentation:** approve GPU-counter, process-list and memory sampling (read-only).
5. **Shared Ruffle config:** keep `harness\ruffle_cfg` (REC, comparability) or use a fresh config per run.
6. **Memory:** you close heavy applications first; confirm the 2.0 GB threshold (REC) and tell me when ready, because windows will open and the real cursor will move.
7. **Part B go-ahead:** approve the `SAVE\` folder, the generator script, the minimum test set (SV0, SV3, SV4, SV5, SV6; REC) or a larger one.
8. **Fixed run path for save tests:** new path `…\SAVE\run\deadly30.swf` (REC) or overwrite `game\run\deadly30.swf`.
9. **Real-save verification:** metadata only, or also a read-only content hash.
10. **Save design after the results:** S-a vs S-b (section B2), append-only rule, and where new weapon ids may be stored.
11. **Documentation:** amend the Stage 0b report to add the OOM panics and `controls2` runs (I did not edit it).
12. **Git:** per your instruction, no commit, push or branch switch has been made. When you approve committing, I will create a new branch for each stage (for example one for this direction/readiness documentation and a later one for the validation stage), commit only the files you name, and push only if you ask. Nothing about this task changes the three untracked reports in `analysis/`.

Nothing in this document has been built, run or tested.
