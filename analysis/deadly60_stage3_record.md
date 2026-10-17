# Deadly 60 — Rollout Stage 3 record (atomic 60-night switch) — STATUS: BLOCKED, NOT BUILT

Date: 2026-10-17. Branch: `stage-3/atomic-60-night-switch`, created from `stage-2/cutscene-option-y` at 6fdb3793d2c2a082298fe322468128f1806213b1. Not merged into `main`.

**This is a blocker record, not a success record.** The Stage 3 patch is prepared and statically checked in the isolated lab, but **no build was run and no Java/FFDec process was started**, because the mandatory 2.00 GiB free-memory gate failed on every reading. No gate (tag, source-diff, manifest, hook) has been run for Stage 3. Nothing about runtime behavior is verified. Patches, SWFs and lab artifacts stay out of Git; this record contains no source code.

## 1. Starting state (verified)

Branch and commit as above, clean tree, remote head of `stage-2/cutscene-option-y` identical, Stage 0 and Stage 1 branches are ancestors, no existing `stage-3/*` ref. Stage 1 `Nights.as` hash `F9EBFC29…8A22` equals its manifest; Stage 2 SWF `38A06158…D1A6` and diff `298BAA9E…A87F` unchanged. Baseline hashes of the four files this stage changes are unchanged: `Level.as` `BB2CE71F…E272`, `HUD.as` `CB8466FE…03A8`, `Deadly30_fla/MainTimeline.as` `2082ED9E…858F`, `Nights.as` `88EB7C28…14C3`.

## 2. What the patch contains (lab only: `D:\Projects\deadly30_lab\game\d60_rollout\stage3\`)

Four unified diffs applied in order to a copy of the baseline (private order file `order_stage3.txt`; the protected `patches\order.txt` is untouched):

| Diff | SHA-256 | Change |
|---|---|---|
| `01_Level.diff` | `04FE7EB7CC6C83726050392C7874831889165C1A1FE5743961999478BF2486AF` | six single-line edits in `Level.as`: the final-night literal 30 → 60 at the five known sites (timer creation at line 340, the placing-key check at 984, the always-alarmed ambient-spawn expression at 1172, and the two final-night gates at 2695 and 2781) plus the Stage 2 cutscene bound at 2711 |
| `02_HUD.diff` | `CC9E0412A1452444603C3AA71F5497E521AA1410083A7653D7C94C05966E32FF` | `HUD.as:122`, HUD clock hidden on night 60 instead of 30 |
| `03_MainTimeline.diff` | `BA89CA63D2FEBF50D86E322BCE309387FB5649ED727C28763D790B995E843E44` | `Deadly30_fla/MainTimeline.as` lines 163 and 167, menu thresholds 30 → 60 |
| `04_Nights.diff` | `22F1C88B7DDFFBD784CE0757DF437EF31600C49674E8D548E210FD483C7F7EF6` | Stage 1 additions (nights 31–60 queues and caps, names at indices 29–58) **plus** the atomic replacement of the old finale placeholder: row 29 and `waveAmount[29]` now hold night 30, and name index 28 is "Thought we were done?" |

Each site was read in context before editing and the neighbouring lines were asserted. Unrelated uses of the literals were not changed (nights 1–2 spawn rule at Level.as:1129, tip at 1598, medal at 2400, `Room.as` night checks, the HUD frame number 30, the `nightsLasted >= 1` Continue gate). `Enemy.as`, `Medals.as` and all other files are untouched.

**Night 30 data source (not invented):** the committed Stage 0 record (section 2.1, commit 90bd037): name "Thought we were done?", cap 11, queue of 193 entries. Verified against the final file.

## 3. Static verification completed (no Java, no build)

- `git apply --check` and apply of all four diffs on a disposable scratch copy succeeded, and the result equals the in-memory targets byte for byte [V]. The scratch-applied `Nights.as` hashes to `9B3ED8698EB037BBB4107AFA87C7DEE1FAE359B20C737205C504A8BC765A209E`.
- `Level.as`: exactly six differing lines at 340, 984, 1172, 2695, 2711 and 2781; `HUD.as`: one; `MainTimeline.as`: two [V].
- `Nights.as`: names 29 → 59, rows and caps 30 → 60; names 0–27 and rows/caps 0–28 identical to the baseline; name 28 is the new night 30 name; rows/caps 30–59 and names 29–58 identical to the Stage 1 patch; row 59 is `[]` with cap 2 (future finale row) [V].
- Metrics of nights 30–59 read back from the final `Nights.as` match the Stage 0 table: 30 rows, **0 mismatches**; night 56 (option 56-B6): length 196, 6 RedMen; night 58 (option 58-C8): length 203, 8 RedMen [V].
- 19 protected items from the Stage 2 snapshot (real save excluded) are unchanged [V].
- A pre-computed allowed-residual list for the source-diff gate (81 entries, simulated with the harness's own normalization rules, plus one pre-registered decompiler-inversion variant for the cutscene bound) is prepared in the lab. It has not been exercised.

## 4. The blocker: memory gate

Required: at least 2.00 GiB (2,097,152 KB) free physical memory before every Java process, with a fail-closed check. Readings: 0.93 GiB at the start, then 1.02, 1.02, 1.07, 1.16 (three rechecks 20 s apart) and 1.21 GiB at the last attempt. **All FAIL. Java was not started.** The gate was not bypassed. Largest memory users at the time were the browser and editor processes.

## 5. Remaining work (needs the gate to pass)

Build with the existing `harness\build.ps1` (`-PatchOrder`, `-OutRoot`, new run id), then the manifest, hook, tag and source-diff gates individually (never `run_gates.ps1`, which hashes the real save): up to 4 Java runs, each preceded by a fresh fail-closed memory check. The build script's own `java -version` call starts a second Java process inside the same script and cannot be gated separately without modifying the harness; this limitation will be recorded. If all gates pass, this record will be replaced by the success record.

## 6. Limitations

- Nothing is compiled; FFDec's AS3 compiler is experimental, and a successful compile would not prove identical runtime behavior.
- B5 Lite remains an evidence-based deferral, **not** a per-file round-trip pass.
- Runtime behavior (night 30 → 31, the night 60 finale, Continue/load, cutscenes, original-save loading) is unverified and reserved for Stage 5.
- No real save was touched, no SAVE folder, generator or launcher exists, and Ruffle and the game were not launched.
