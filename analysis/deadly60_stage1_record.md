# Deadly 60 — Rollout Stage 1 record (append-only night data)

Date: 2026-10-17. Branch: `stage-1/append-night-data`. Parent: `stage-0/design-verification` at 90bd037 (approved chain; `main` is untouched).

**Status: the append-only data patch is prepared in the isolated lab and statically verified. It is NOT built.** FFDec, Ruffle, Java and the game were not run. No real save was read. This document is a stage record only; it contains no decompiled code and no patched source.

Labels: **[V]** verified by direct file comparison or reproducible in-memory calculation, **[I]** inferred, **[H]** unverified runtime hypothesis.

## 1. Decisions applied (from the Stage 0 review)

| Item | Decision |
|---|---|
| B1 | Decompiled sources, SWFs and patched files stay in the lab and are never committed to this public repository. Stage branches carry documents only. |
| B2 | Append 30 names at indices **29–58** (nights 31–60). Index 28 stays unchanged. |
| B3 | A checkpoint above night 30 may not load in the unmodified original build. This is an expected compatibility risk, not a verified result. |
| B4 | Each rollout stage branches from the previously approved stage branch; no merge into `main`. |
| B5 / B6 | Round-trip proofs for other files and the save-test setup each need their own approval before they start (not part of this stage). |

## 2. Where the patch lives (NOT in Git)

| Item | Location (lab only) |
|---|---|
| Patched `Nights.as` | `D:\Projects\deadly30_lab\game\d60_rollout\stage1\Nights.as` |
| Lab manifest | `D:\Projects\deadly30_lab\game\d60_rollout\stage1\stage1_manifest.txt` |
| Baseline (untouched, read-only) | `D:\Projects\deadly30_lab\game\src_baseline\com\iggyzuk\deadly30\Nights.as` |

| File | Size | SHA-256 |
|---|---|---|
| Baseline `Nights.as` | 10,474 bytes | `88EB7C280FF4D97E8BEFF2DEC8A81B56B9175A27F665259D355B3D1309D414C3` |
| Patched `Nights.as` | 24,138 bytes | `F9EBFC290F517CBFD9C5BD36B8053CA78E91319729B1913687193EFAA96D8A22` |

**This patch is not backed up in Git.** Its only copies are the lab folder above and the hashes recorded here.

## 3. Exact source change (summary)

Pure insertions relative to the baseline; nothing removed, reordered or reformatted. Line endings (CRLF), encoding (UTF-8, no BOM) and indentation style match the baseline.

1. **Night names** (baseline line 13): the existing 29 names are unchanged and 30 names are appended, so the list has 59 entries. Index 28 is still "Final Night Deadly 30" (the old finale slot, preserved until the atomic switch). Indices 29–58 hold the names of nights 31–60 as listed in the Stage 0 record; index 29 is night 31 ("Run, run, run!") and index 58 is night 60 ("Final Night Deadly 60"). The mapping is `nightNameList[nightCount − 2]` (Level.as:313) [V].
2. **Queues and caps:** after the last original pair (the live finale row, an empty queue with cap 2), 30 `waveArray` rows and 30 `waveAmount` values are appended for indices 30–59 (nights 31–60). Index 59 is an empty queue with cap 2 (the future finale row). Nights 56 and 58 carry the approved options 56-B6 and 58-C8.
3. **Not touched:** row 29, `waveAmount[29]`, name index 28, night 30 data, all other files.

Resulting lengths [V]: names 29 → 59, `waveArray` 30 → 60, `waveAmount` 30 → 60. Patched file: 138 lines (baseline 78).

## 4. Verification results (static, in memory; no tool was run)

| Check | Result |
|---|---|
| Line-level diff vs baseline | Exactly two operations: line 13 extended (the original text is a strict prefix) and 60 lines inserted after baseline line 73. No other difference [V]. |
| Names 0–28 identical to baseline; names[28]; names[29]; names[58] | Identical; "Final Night Deadly 30"; "Run, run, run!"; "Final Night Deadly 60" [V] |
| `waveArray` rows 0–29 and `waveAmount` 0–29 identical to baseline | Yes. Row 29 is still `[]` and `waveAmount[29]` is still 2 [V] |
| Row 59 and `waveAmount[59]` | `[]` and 2 [V] |
| Ids used | all within 0–10 [V]; caps range 2–20 [V] |
| Queue round-trip | Patched rows 30–58 equal the notation in the Stage 0 record, expanded independently [V] |
| Names and caps vs record headers (nights 31–60) | Equal [V] |
| Metrics for nights 31–59 (29 rows) vs the record's table | **0 mismatches** across counts, length, effective bound, total HP, mean, window, heavy-per-window, Tank/RedMan runs, TOPCAP and tail [V] |
| Night 56 | Length 196, RedMen 6, total 138,080, window 17,268, TOPCAP 66,000 [V] |
| Night 58 | Length 203, RedMen 8, total 166,420, window 24,276, TOPCAP 68,000 [V] |
| Baseline file after the work | Hash unchanged [V] |
| Searches for other users of the arrays | `waveArray.length` appears only at Level.as:591; `nightNameList` only at Level.as:313; the loader reads rows `0..nightCount−1` (≤ 29 while the final-night literal is still 30) [V grep] |

Night 30's queue and name are intentionally **not** in this stage (they belong to the Stage 3 atomic switch), so the table row for night 30 is not part of this verification.

## 5. Expected behavior and what remains unverified

- With the final-night literal still 30, nothing reads the appended rows or names, so the patched data should not change the original game's behavior [I]. This is not tested.
- **Compile cleanliness is unverified.** The patched file follows the baseline's statement pattern, but FFDec was not run, so a successful recompile and an unchanged rest-of-SWF are not claimed. A no-op round trip and a build need separate approval.
- Runtime-only questions (Continue behavior, the first-frame `nightOver` handler, cutscene replay on load, queue refill and overshoot, night 30 → 31, the night 60 finale, original-save loading, Ruffle versus real Flash) remain open as listed in the Stage 0 record.

## 6. Not done in this stage

No change to the finale row, `waveAmount[29]`, name index 28, cutscene logic, final-night constants, HUD, menus, medals or save logic. No build, no FFDec, no Ruffle, no game run, no save access. Stages 2–5 have not started.

## 7. Prerequisites before Stage 2 (each needs its own approval)

1. A no-op FFDec round-trip proof for each file that later stages must modify (`Level.as`, `HUD.as`, `MainTimeline.as`, `Medals.as`), stopping at the first failure.
2. The save-test setup proposal (files, generator behavior, synthetic cases, isolation and rollback) before any `SAVE\` folder or script is created.
3. Explicit approval of Stage 2 and its branch `stage-2/cutscene-option-y`, based on this stage's branch.
