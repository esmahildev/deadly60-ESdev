# Deadly 60 — Rollout Stage 3 record (atomic 60-night switch)

**STATUS: BUILT; 3 of 4 static gates PASSED; the SOURCE-DIFF GATE FAILED. Stage 3 is NOT marked passed. Runtime behavior is NOT verified.**

Date: 2026-10-17. Branch: `stage-3/atomic-60-night-switch`, created from `stage-2/cutscene-option-y` at 6fdb3793d2c2a082298fe322468128f1806213b1. Not merged into `main`. This record replaces the earlier blocker record (660fdf4), which covered the period when the memory gate was failing. The patch, SWF and lab artifacts stay in the isolated lab and are not in Git; this record contains no source code.

## 1. Starting state and memory gate

Clean tree; HEAD equal to the blocker-record commit; the four patch diffs equal the hashes recorded in the blocker record; the Stage 1 patch, the Stage 2 outputs and the baseline files unchanged. A 23-item protected-hash snapshot (real save excluded) was taken before the build.

Memory gate (simple fail-closed check that stops if the reading fails or is below 2,097,152 KB), taken immediately before each Java step: **2.96 GiB** before the build, **2.93 GiB** before the tag gate, **2.87 GiB** before the source gate. All passed. Limitation: the build script itself starts a second Java process (`java -version`, for its manifest) inside the same script; that process cannot be gated separately without modifying the harness.

## 2. The patch (lab only: `D:\Projects\deadly30_lab\game\d60_rollout\stage3\`)

| Diff | SHA-256 | Change |
|---|---|---|
| `01_Level.diff` | `04FE7EB7CC6C83726050392C7874831889165C1A1FE5743961999478BF2486AF` | six single-line edits in `Level.as`: final-night literal 30 → 60 at lines 340, 984, 1172, 2695, 2781, and the Stage 2 cutscene bound at 2711 |
| `02_HUD.diff` | `CC9E0412A1452444603C3AA71F5497E521AA1410083A7653D7C94C05966E32FF` | `HUD.as:122` |
| `03_MainTimeline.diff` | `BA89CA63D2FEBF50D86E322BCE309387FB5649ED727C28763D790B995E843E44` | `MainTimeline.as` lines 163 and 167 |
| `04_Nights.diff` | `22F1C88B7DDFFBD784CE0757DF437EF31600C49674E8D548E210FD483C7F7EF6` | Stage 1 data (nights 31–60 queues, caps, names at indices 29–58) plus the atomic replacement of the old finale placeholder with night 30 (row 29, cap 11, name "Thought we were done?", taken from the committed Stage 0 record) |

Stage 1's appended data **is included** in this build. Night 56 (option 56-B6) and night 58 (option 58-C8) are unchanged. `Enemy.as`, `Medals.as` and all other files are untouched. Static pre-checks (see the earlier record contents): `git apply` on a scratch copy reproduces the targets byte for byte; the final `Nights.as` matches the Stage 0 table for nights 30–59 with 0 mismatches.

## 3. Build (existing `harness\build.ps1`, flavor `prod`, 18-class recompile list)

| Item | Value |
|---|---|
| Run | `S3_ATOMIC_1` via `-PatchOrder` (private order file) and `-OutRoot`; exit code 0; about 25 s |
| Output | `D:\Projects\deadly30_lab\game\d60_rollout\stage3\builds\S3_ATOMIC_1\deadly30.swf`, 19,285,356 bytes, SHA-256 `FC7199207C0E8EA544999C1A55EB89336F76188EB327DFCF66A4512FE380E9EF` |
| Tools | Java 1.8.0_503 (from the build's own `java -version`); FFDec jar SHA-256 `1587E60B…841C` (equal to the pin; the "26.3.0" label is the harness pin) |
| Recompiled sources (SHA-256) | Level `336BEC9D…39F8`, HUD `C7C07E99…056A`, MainTimeline `53DD97EA…9E09`, Nights `9B3ED869…209E` |
| Java runs | 4 of the bounded budget (compile, `java -version`, tag dump, script export) |

## 4. Gate results (each run individually; `run_gates.ps1` and any real-save hashing were not used)

| Gate | Result |
|---|---|
| Manifest | **PASS**: 4 patches, 18 recompiled, 381 source files |
| Hook marker | **PASS**: 0 hits |
| SWF tag | **PASS**: only `DoABC2` differs from the original |
| Decompiled-source diff | **FAIL**: 96 residual lines; the 15 documented residuals are present; 79 lines matched my pre-registered allowed list; **2 unexpected lines, both in `Nights.as`** |

Byte-level tag check (own parser, in memory): 2,364 tags in each of original, 18-class no-op build, Stage 2 and Stage 3; **exactly one tag differs from each of them (`DoABC2`: 305,236 original / 327,878 no-op / 327,892 Stage 2 → 339,784 bytes)**; every other tag is byte-identical. Protected files: all 23 snapshot items identical after the build.

## 5. Why the source-diff gate failed (diagnostic after the failure; it does not change the result)

The two unexpected lines are the decompiler's rendering of the long night-name array statement in `Nights.as`: FFDec printed it across **two physical lines** (about 1,043 and 434 characters), whereas my pre-registered allowed list assumed a single line. Without any further Java run, from the saved export:

- Names (59), rows (60) and caps (60) parsed from the exported `Nights.as` are **equal** to the Stage 3 source target.
- All other lines of the exported `Nights.as` are identical to the target (multiset comparison).
- The other three files' extra residuals are exactly the 18 intended lines (Level 12, HUD 2, MainTimeline 4); nothing unplanned.

So the failure appears to be a prediction error about line wrapping, not an unintended code difference. **It has not been waived.** No allowed list was amended and the gate was not re-run. Resolving it needs a decision: either re-evaluate the saved export (no Java) with an explicitly reviewed amended allowed list that adds the two wrapped lines, or another agreed treatment.

## 6. Limitations

- Source-diff gate failed (above); Stage 3 is not a pass until that is resolved by an explicit decision.
- Runtime behavior is unverified (night 30 → 31, the night 60 finale, Continue/load, cutscenes, original-save loading); runtime acceptance is reserved for Stage 5. FFDec's AS3 compiler is experimental.
- B5 Lite remains an evidence-based deferral, **not** a per-file round-trip pass.
- No real save was opened, hashed or copied; no SAVE folder, generator or launcher exists; Ruffle and the game were not launched.
