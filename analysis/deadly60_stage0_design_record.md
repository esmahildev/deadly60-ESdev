# Deadly 60 — Stage 0 design record (design verification, planning only)

Date: 2026-10-17. Branch: `stage-0/design-verification`, created from `main` (d682686).

**Status: design and planning only. Nothing is implemented.** No gameplay source, SWF, save, README or CHECKLIST was changed. No game, Ruffle, Java, FFDec, build, test or validator was run. This document contains design data, formulas and source line references only; it contains no decompiled code.

Labels: **[V]** verified from source or by a reproducible in-memory calculation, **[I]** inferred from source, **[H]** unverified runtime hypothesis, **[P]** proposal.

"Rollout Stage N" below means the Deadly 60 implementation stages 0–5. Earlier reports on other branches (`stage-1-audit`, `stage-2-plan`, `stage-3-*`, `stage-4-plan`, `stage-5-validation-prep`) use an older numbering and are unrelated to these rollout stages.

---

## 1. Approved design baseline

| # | Decision |
|---|---|
| D1 | Nights 1–29 keep their original data and behavior. Night 30 becomes an ordinary night. Night 60 is the sole finale and keeps the existing timer, helicopter, boarding, cutscene 6 and menu flow. Only enemy ids 0–10 are used. |
| D2 | Nights 30–34 stay exactly as proposed below (calm opening is intentional). |
| D3 | **Night 56 = option 56-B6**: remove the first RedMan of the `8,8` pair in blocks 1–4, keep the pair in block 5. **Night 58 = option 58-C8**: `8x1` in blocks 1–2, `8x2` in blocks 3–5. |
| D4 | **Cutscene Option Y**: at the night-end handler (Level.as:2711) the `nightCount % 5` branch applies only up to night 25; nights 30–55 take the ordinary `addDay()` immediately (no 120-frame pause). Ids 7–11 are never dispatched; victory cutscene 6 stays on its separate helicopter path. Continue behavior and the first-frame `nightOver` re-run are **runtime-unverified**. |
| D5 | Save format and checkpoint structure unchanged (same SharedObject, same `saveString` sections). Read as: one `saveString` that may hold a night above 30 (see 6.2). No real save is read or modified. |
| D6 | The `nightsLasted` counter is not changed. Menu thresholds simply move from 30 to 60. |

## 2. Night 30–60 table (design values)

Notation: `i`xK = id i repeated K times; `(a,b,c)xK` = pattern a,b,c repeated K times; `[ ... ]xR` = group repeated R times; ` | ` separates consecutive segments. Ids: 0 Green, 1 Blue, 2 Red, 3 White, 4 Fatty, 5 Tank, 6 Dog, 7 Bird, 8 RedMan, 9 BigDog, 10 RedFatty.

### 2.1 Ordered FIFO queues

```
N  cap  name / purpose / queue
30  11  Thought we were done? (regular)
        0x8 | (0,1)x10 | [(2,0,1)x6 5x1]x4 | (3,2,0)x10 | [(3,0)x8 5x1]x3 | 3x8
31  18  Run, run, run! (swarm)
        1x12 | (1,6)x12 | (1,10,6)x10 | (9,6)x10 | (2,1,10,3)x10 | (9,9,10,1)x8 | (3,1,6)x12
32  12  The White Tide returns (regular)
        3x10 | (3,3,2)x12 | [(3,0)x8 5x1]x3 | (3,3,3,7)x6 | (3,2,3,2,1)x8 | 3x10
33  11  Mind the Tank (mixed)
        [(2,3)x8 5x2]x5 | (3,3,2,2,0)x6 | [(2,3,10)x8 5x1]x3
34  19  Look up! Look down! (swarm)
        (7,6)x10 | (3,9,7)x10 | (3,2,9)x8 | (7,7,6,3)x10 | (9,10,3)x10 | (3,7,9,10)x8 | (4,4,9)x6
35   5  Tanks and bloaters (heavy, milestone)
        (2,3)x6 | [5x4 (2,10)x3]x8 | 5x5
36  13  Ketchup, mustard and gas (regular)
        (2,4)x16 | (2,2,10)x14 | (2,4,4,10)x10 | [(2,10,2)x8 5x1]x3 | (3,2,10)x10
37  14  The Soviets are back (mixed)
        [(3,0,1,2,6)x6 8x1]x6
38  19  Gonzo's whole pack (swarm)
        (6,9)x12 | (9,9,10)x8 | (3,6,3,6,9)x10 | (10,2)x10 | (6,9,3)x12 | (3,2)x8
39  12  Giants no like Germans, part 2 (mixed)
        [(3,1,0,7)x6 5x2]x4 | [(1,0,7,1)x6 8x1]x3 | (0,1)x10
40   7  The Giants gather (heavy, milestone)
        (2,3)x8 | [9x3 5x3 8x1]x10 | 10x10
41  15  Where are they coming from? Again! (regular)
        (0,1,7)x12 | (2,3)x14 | (4,0,1,3)x10 | (10,3)x8 | [(3,2,0)x8 5x1]x3
42  20  A whole lotta fatties (swarm)
        (4,2,3)x14 | (10,4,2)x12 | (4,4,10,10,3)x8 | (1,10,2)x12 | (4,3,2)x16
43  12  Red giants on parade (mixed)
        [(2,4,10)x8 8x1]x6 | (2,3)x8
44  16  Birds of prey (regular)
        (7,2)x12 | (7,7,2,6)x10 | (2,9,0)x14 | (7,7,3)x8 | [(2,3)x10 5x1]x3
45   6  The Wall (heavy, milestone)
        (3,2)x6 | [8x2 5x2 (2,3)x3]x8 | 8x2
46  20  Who let the hellhounds out? (swarm)
        9x10 | (6,9,10)x14 | (1,9,6)x10 | (10,10,9)x8 | (6,3)x12 | (9,3,10)x10 | (3,2)x10
47  13  Steel and rust (mixed)
        [(1,6,10,0)x7 5x1 (1,2,3)x5 8x1]x5
48  17  Tidal wave (regular)
        (0,1)x20 | (2,3)x20 | (1,2,3)x12 | 3x16 | [(3,2)x8 5x2]x3
49  14  Heavy weather (mixed)
        (7,7,3)x8 | [(9,6,3)x6 8x1 (3,2,10)x5 5x2]x4 | (7,3)x10
50   4  No man's land (heavy, milestone)
        (2,3)x4 | [8x3 5x1 (3,2)x2 5x1 10x2]x8 | 8x3
51  20  Don't look back (swarm)
        (0,2,10)x14 | (6,0,6,3)x10 | (9,10,0,6)x8 | (2,10,2)x12 | (10,9,6,3)x10
52  15  Rust and bone (mixed)
        [(3,3,9,10)x4 8x1 (3,3,9,10)x4 8x1 3x6 5x1]x4
53  18  The White Tide, third wave (regular)
        3x12 | (3,3,2)x14 | (3,2,7)x10 | (3,3,3,0)x10 | (3,2,1)x12 | [(3,2)x10 5x1]x3
54  20  Overrun (swarm)
        (0,1)x16 | (1,6,10)x12 | (2,3,4)x14 | (9,10,6,1)x10 | (3,2,0,1,10)x10
55   4  The last wall (heavy, milestone)
        (3,2)x6 | [8x4 (3,2)x2 5x2 (9,10)x2]x7 | 8x4
56  14  Cold iron (mixed)  -- option 56-B6
        [(9,3,10,2)x6 5x2 (9,6,10)x4 8x1]x4 | [(9,3,10,2)x6 5x2 (9,6,10)x4 8x2]x1
57  18  Almost home... almost (regular)
        (3,6,9,10)x10 | (7,7,3,9)x8 | (3,10,6,2)x10 | (0,1,3,2)x10 | [(3,9,10)x6 5x1]x2 | [(3,9,10)x6 8x1]x2
58  12  Everything at once (mixed, pre-finale climax)  -- option 58-C8
        [(3,2,0,1)x5 (6,9,10)x3 5x2 8x1 (3,2)x4]x2 | [(3,2,0,1)x5 (6,9,10)x3 5x2 8x2 (3,2)x4]x3
59  15  The calm before the storm (breather)
        (0,1)x10 | (2,3)x10 | (7,6,4)x8 | (3,2,1)x10 | (0,1,2,3)x8 | 3x8
60   2  Final Night Deadly 60 (finale, logic unchanged)
        row [] (empty); refilled from endlessWave by the existing schedule
```

Night names: index = night − 2 in `nightNameList` (Level.as:313). Existing indices 0–28 are nights 2–30 and stay unchanged until the atomic switch (index 28, "Final Night Deadly 30", is night 30's slot).

### 2.2 Metrics (HP at that night's own stats)

Window = maximum sum of HP over any `cap` consecutive queue entries (assumes kills in spawn order). Mean = cap x total / length (an approximation). TOPCAP = sum of the `cap` highest-HP entries of the whole row (an idealized worst-case accumulation, [I]; the original maximum is 70,000 at night 28). Eff = length + cap − 1 (valid only without room-transition requeue, Level.as:1033-1039). Tail = (cap−1) extra Greens if the queue empties mid-loop ([I]).

| N | Counts (id:n) | Len | Eff ≤ | Total HP | Mean | Window | Heavy/win | Run T/R | TOPCAP | Tail |
|---|---|---|---|---|---|---|---|---|---|---|
| 30 | 0:76 1:34 2:34 3:42 5:7 | 193 | 203 | 85,270 | 4,860 | 7,305 | 1 | 1/0 | 22,780 | 3,000 |
| 31 | 1:64 2:10 3:22 6:44 9:26 10:28 | 194 | 211 | 51,434 | 4,772 | 6,229 | 0 | 0/0 | 8,082 | 5,185 |
| 32 | 0:24 1:8 2:28 3:102 5:3 7:6 | 171 | 182 | 77,142 | 5,413 | 7,411 | 1 | 1/0 | 13,077 | 3,410 |
| 33 | 0:6 2:76 3:76 5:13 10:24 | 195 | 205 | 112,310 | 6,335 | 9,978 | 2 | 2/0 | 33,000 | 3,150 |
| 34 | 2:8 3:46 4:12 6:20 7:48 9:42 10:18 | 194 | 212 | 51,756 | 5,069 | 7,331 | 0 | 0/0 | 8,759 | 5,760 |
| 35 | 2:30 3:6 5:37 10:24 | 97 | 101 | 131,910 | 6,799 | 15,000 | 5 | 5/0 | 15,000 | 1,300 |
| 36 | 2:112 3:10 4:36 5:3 10:58 | 219 | 231 | 79,084 | 4,694 | 7,695 | 1 | 1/0 | 13,690 | 3,960 |
| 37 | 0:36 1:36 2:36 3:36 6:36 8:6 | 186 | 199 | 103,236 | 7,770 | 11,501 | 1 | 0/1 | 45,784 | 4,355 |
| 38 | 2:18 3:40 6:44 9:50 10:18 | 170 | 188 | 52,702 | 5,890 | 8,246 | 0 | 0/0 | 9,063 | 6,120 |
| 39 | 0:52 1:70 3:24 5:8 7:42 8:3 | 199 | 210 | 101,910 | 6,145 | 10,555 | 2 | 2/1 | 45,481 | 3,795 |
| 40 | 2:8 3:8 5:30 8:10 9:30 10:10 | 96 | 102 | 178,250 | 12,997 | 16,840 | 4 | 3/1 | 49,000 | 2,100 |
| 41 | 0:46 1:22 2:38 3:56 4:10 5:3 7:12 10:8 | 195 | 209 | 81,500 | 6,269 | 9,085 | 1 | 1/0 | 14,868 | 4,970 |
| 42 | 1:12 2:54 3:38 4:58 10:40 | 202 | 221 | 64,410 | 6,377 | 7,391 | 0 | 0/0 | 9,860 | 6,840 |
| 43 | 2:56 3:8 4:48 8:6 10:48 | 166 | 177 | 90,008 | 6,507 | 12,197 | 1 | 0/1 | 44,982 | 4,015 |
| 44 | 0:14 2:66 3:38 5:3 6:10 7:48 9:14 | 193 | 208 | 77,290 | 6,407 | 10,200 | 1 | 1/0 | 15,513 | 5,550 |
| 45 | 2:30 3:30 5:16 8:18 | 94 | 99 | 202,950 | 12,954 | 20,965 | 4 | 2/2 | 42,000 | 1,875 |
| 46 | 1:10 2:10 3:32 6:36 9:52 10:40 | 180 | 199 | 56,724 | 6,303 | 9,730 | 0 | 0/0 | 10,180 | 7,220 |
| 47 | 0:35 1:60 2:25 3:25 5:5 6:35 8:5 10:35 | 225 | 237 | 123,360 | 7,127 | 12,208 | 1 | 1/1 | 51,539 | 4,620 |
| 48 | 0:20 1:32 2:56 3:72 5:6 | 186 | 202 | 99,824 | 9,124 | 13,440 | 2 | 2/0 | 23,687 | 6,240 |
| 49 | 2:20 3:62 5:8 6:24 7:26 8:4 9:24 10:20 | 188 | 201 | 115,794 | 8,623 | 12,597 | 2 | 2/1 | 53,042 | 5,135 |
| 50 | 2:20 3:20 5:16 8:27 10:16 | 99 | 102 | 261,500 | 10,566 | 24,000 | 4 | 1/3 | 28,000 | 1,200 |
| 51 | 0:32 2:38 3:20 6:38 9:18 10:44 | 190 | 209 | 67,276 | 7,082 | 8,444 | 0 | 0/0 | 10,580 | 7,695 |
| 52 | 3:88 5:4 8:8 9:32 10:32 | 164 | 178 | 133,624 | 12,222 | 16,219 | 2 | 1/1 | 69,599 | 5,740 |
| 53 | 0:10 1:12 2:66 3:122 5:3 7:10 | 223 | 240 | 117,064 | 9,449 | 11,769 | 1 | 1/0 | 17,055 | 7,055 |
| 54 | 0:26 1:48 2:24 3:24 4:14 6:22 9:10 10:32 | 200 | 219 | 71,276 | 7,128 | 8,399 | 0 | 0/0 | 10,820 | 7,980 |
| 55 | 2:20 3:20 5:14 8:32 9:14 10:14 | 114 | 117 | 295,300 | 10,361 | 28,000 | 4 | 2/4 | 28,000 | 1,275 |
| **56** | 2:30 3:30 5:10 6:20 **8:6** 9:50 10:50 | **196** | **209** | **138,080** | 9,863 | **17,268** | 2 | 2/2 | **66,000** | 5,590 |
| 57 | 0:10 1:10 2:20 3:62 5:2 6:20 7:16 8:2 9:42 10:44 | 228 | 245 | 105,902 | 8,361 | 13,682 | 1 | 1/1 | 27,742 | 7,395 |
| **58** | 0:25 1:25 2:45 3:45 5:10 6:15 **8:8** 9:15 10:15 | **203** | **214** | **166,420** | 9,838 | **24,276** | 4 | 2/2 | **68,000** | 4,840 |
| 59 | 0:18 1:28 2:28 3:36 4:8 6:8 7:8 | 134 | 148 | 57,322 | 6,417 | 8,100 | 0 | 0/0 | 8,415 | 6,230 |

Highest TOPCAP in the table: night 52 at 69,599 (below the original maximum of 70,000). All values in this table were recomputed in memory from the queue notation above using HP formulas parsed from the source files, and match the earlier audit (0 differences) except nights 56 and 58, which carry the approved edits.

### 2.3 Original-game reference [V]

- Maximum window over the original nights 1–29: **28,000** at night 29 (cap 4, RedMan run of 38). Next: 23,351 (n24), 18,550 (n26), 17,496 (n28), 16,863 (n21), 15,609 (n19), 15,000 (n27). Original windows for nights 22–25 are 6,195 / 4,910 / 23,351 / 10,300.
- The original finale's last phase (cap 26) reaches 36,920 (night 30 stats) and 38,870 at night 60 stats.
- Original TOPCAP maximum: 70,000 (n28), then 67,000 (n25), 59,000 (n21), 57,700 (n24).
- 28,000 is a **provisional design constraint**, not proof of balance. The window assumes kills in spawn order; TOPCAP is an idealized bound; ambient spawns are excluded from every figure.

## 3. Source dependency map (read-only inspection of the baseline decompile)

| Site | Role | Rollout stage that touches it |
|---|---|---|
| Nights.as: `waveArray`/`waveAmount` (30 rows), `nightNameList` (29 names), row 29 = `[]` cap 2 (live finale row) | queue data | S1 append-only, S3 replace row 29 / name index 28 |
| Level.as:340, 984, 1172, 2695, 2781; HUD.as:122 | final-night literal `30` | S3 (all at once) |
| Level.as:2711 (`% 5` branch) | night-end cutscene gate | S2 (Option Y) |
| Level.as:2566-2583, 2575 | cutscene dispatch (only reached via 2715) | unchanged |
| Controller.as:245-283 | `addCutscene` ids 0–6 only | unchanged |
| Deadly30_fla/MainTimeline.as:163-170 | menu text thresholds 30 | S3 |
| Medals.as:39 | medal 18 text | S4 |
| Level.as:313 | night intro text `nightNameList[nightCount − 2]` | unchanged |
| Level.as:1785-1904 | checkpoint loader (unguarded; rows 0..nightCount−1) | unchanged |
| Level.as:2390-2436, 1594-1705 | `addDay` / `addCheckPoint` | unchanged |
| Helicopter.as:92-112, 144-187; Controller.as:179-211 | finale boarding, cutscene 6, menu | unchanged |
| Level.as:2778-2906, 3525-3536 | finale schedule, timer, `endlessWave` | unchanged |

## 4. The three critical issues

### Issue A — Stage 1 must not break the existing finale [V]

Row 29 and `waveAmount[29]` (empty queue, cap 2) are the **live finale row**. Replacing them before the final-night logic is switched would break the unmodified game's ending. Therefore:

- **S1 is append-only.** Append `waveArray`/`waveAmount` entries for indices **30–59** (nights 31–60); index 59 is `[]` with cap 2 (the future finale row). Leave row 29, `waveAmount[29]` and name index 28 untouched.
- Append the matching names. **Discrepancy found:** the approval text says names for indices 30–58. By the source (`nightNameList[nightCount − 2]`, Level.as:313) night 31 is index **29** and night 60 is index 58, so the names to append are indices **29–58** (30 names: nights 31–59 plus "Final Night Deadly 60"). Appending only 30–58 would leave index 29 undefined and night 31 would show an undefined name. This record uses 29–58 and asks for confirmation (section 9).
- Resulting lengths: `waveArray` 60, `waveAmount` 60, `nightNameList` 59.
- At S1 the final-night literal is still 30, so rows and names beyond index 29/28 are unused by the game (the loader loops only to `nightCount − 1`, at most 29). S1 is behavior-neutral [I].
- **S3 (atomic):** replace row 29, `waveAmount[29]` and name index 28 with the night-30 data, and switch the literal at all sites together.

### Issue B — Backup and save-testing plan (planning only; no execution authorized)

No real save has been read in this task. Existing assets and rules (see `first_validation_preparation.md` on branch `stage-5-validation-prep`, commit 72b640e):

1. **Original project and SWF.** The lab (`D:\Projects\deadly30_lab`) already holds the original SWF backup, the `src_baseline` decompile and a hash manifest of protected files. Nothing is rebuilt over them; every modified build goes to a new file under `builds\`. The repository itself holds documents only (see Issue C).
2. **Real save protection.** The real save is never opened, copied or written by any step. A backup copy of it was made earlier (hash recorded in the stage-5 document). All test inputs come from synthetic files derived from that backup copy, and any new copy of anything save-related needs separate approval.
3. **Isolated testing.** Save tests use a disposable per-session save directory under a new `SAVE\sessions\<Tag>\` folder with `--storage disk --save-directory <that folder>`; the launcher refuses to start otherwise. One Ruffle window at a time, graceful close only, free memory at least 2,097,152 KB (recheck up to 3 times, 20 s apart, then stop), no force-kill.
4. **Restore path.** Because tests never touch the real save, nothing needs restoring. If any protected hash or backup differs afterwards, the procedure is: stop, report the original output, restore nothing automatically, and restore only from the verified backups with explicit approval.
5. **Checkpoints before and after night 30.**
   - Original SWF (already specified): SV0 (unchanged copy), SV1 (night 29), SV2 (night 30), SV3 (night 31, expected failure), SV4–SV6 (unknown ids / extra section).
   - Unified build (to be specified for Stage 5): UV0 (SV0 loads), UV1 (checkpoint at 29 continues into an ordinary night 30), UV2 (checkpoint at 30), UV3 (checkpoints at 34, 35, 55 and 59; 35 and 55 exercise Option Y), UV4 (menu key `nightsLasted` = 30 shows the new text and Continue works).
6. **Original save without risking the only copy.** Only the synthetic copy derived from the backup is loaded (SV0/UV0).
7. **Not claimed:** save compatibility is unverified until those tests pass and are reported as Passed / Failed / Skipped / Unverified.

### Issue C — Git workflow and the conflict it exposes

Facts [V]: remote `origin` = `https://github.com/esmahildev/deadly60-ESdev.git` (public); `main` = d682686; working tree was clean; no stash; no CI workflows (`.github` absent) and no active git hooks; existing branches are listed in section 8.

**Conflict to resolve (blocker B1).** The repository `.gitignore` deliberately excludes decompiled sources (`analysis/as3_scripts/`), SWFs, executables, `patched/`, `backup/`, `build/` and saves, and the standing project rule is that patches and game sources are never committed. Rollout Stages 1–4 produce exactly such material (a patched `Nights.as`, patched SWFs). They cannot be committed without publishing third-party game material in a public repository. Proposed resolution [P]: stage branches carry **documents only** (stage record, data spec, check results with hashes); patched sources and builds stay in the lab under `patches\` and `builds\`. Where a stage has no repo-visible file change, no commit is made (no empty commits).

Workflow rules applied: one stage, one branch; each stage branch is created from its explicitly approved parent (proposed chain below); after each approved change the diff is reviewed, only authorized checks are run, and the result is committed and pushed to that branch; the real push result is reported and nothing is claimed unverified; never merge into `main`, force-push, reset, delete branches or commit secrets.

## 5. Corrected rollout sequence (each stage needs separate approval)

| Stage | Branch | Proposed parent | Scope | Repo change | Lab change | Checks |
|---|---|---|---|---|---|---|
| 0 | `stage-0/design-verification` | `main` (d682686) | this record | this document | none | static only |
| 1 | `stage-1/append-night-data` | stage-0 branch | append-only data and names (Issue A) | stage record + data spec | `Nights.as` patch, new build | array lengths 60/60/59; original rows byte-identical; queue round-trip and metrics vs section 2; build hash |
| 2 | `stage-2/cutscene-option-y` | stage-1 branch | only the Level.as:2711 bound | stage record | `Level.as` patch | decompile diff = one condition; no-op round trip of `Level.as` proven first |
| 3 | `stage-3/night-60-switch` | stage-2 branch | atomic switch: sites in section 3 + row 29 / name index 28 + menu thresholds | stage record | `Level.as`, `HUD.as`, `MainTimeline.as`, `Nights.as` patches | all sites changed together; static diff; no partial state ever built |
| 4 | `stage-4/text-and-medals` | stage-3 branch | medal 18 wording and approved text | stage record | `Medals.as` patch | text-only diff |
| 5 | `stage-5/runtime-acceptance` | stage-4 branch | individually approved runtime checks | results record | session evidence | memory gate for every Ruffle run |

Prerequisite for S2–S4 [P]: only the `Nights.as` replacement recipe has been proven with FFDec so far (C5 recipe). Replacing `Level.as`, `HUD.as`, `MainTimeline.as` and `Medals.as` must first be shown to round-trip cleanly (recompile risk is **unverified**). This needs its own approval.

## 6. Verification requirements for the Night 56 / 58 edits

Before S1 is built, recompute and compare (any mismatch means stop and report, never silently correct):

| Night | Edit (exact) | Len | Total HP | Window | TOPCAP |
|---|---|---|---|---|---|
| 56 | `[... 8x2]x5` → `[... 8x1]x4 \| [... 8x2]x1`; block lengths 39, 39, 39, 39, 40; RedMen per block 1, 1, 1, 1, 2 | 196 | 138,080 | 17,268 | 66,000 (6 RedMen + 8 Tanks) |
| 58 | `[... 8x2 ...]x5` → `[... 8x1 ...]x2 \| [... 8x2 ...]x3`; block lengths 40, 40, 41, 41, 41; RedMen per block 1, 1, 2, 2, 2 | 203 | 166,420 | 24,276 | 68,000 (8 RedMen + 4 Tanks) |

Verified in memory this task [V]: each new queue equals the current queue with exactly one RedMan removed per listed block (4 and 2 entries, 28,000 and 14,000 HP removed); HP formulas were parsed from the source files and agree with the audit table; TOPCAP closed forms agree.

## 7. Runtime-only questions (unresolved; static inspection does not settle them)

- Continue behavior and the first-frame `nightOver` handler (basis of the Continue cutscene replay and of the `nightsLasted` value) [I].
- Cutscene replay after loading a night 5–25 checkpoint [I].
- Kill-driven refill, tail overshoot (undefined → id 0) and room-transition requeue [I].
- Night 30 → 31 progression with Option Y; dispatch behavior of ids 6–11 without the bound [I].
- Night 60: timer, caps, helicopter, boarding, cutscene 6, menu text and `nightsLasted` for uninterrupted and continued wins.
- Original-save loading, and checkpoints at dawn 29, 30, 34, 35, 55, 59.
- Differences between Ruffle and real Flash.

## 8. Repository state at Stage 0

Branches (local = remote): `main` d682686; `stage-1-audit` 73e8946; `stage-2-plan` b7972fa; `stage-3-feasibility` 337b126; `stage-3b-runtime` 80d482c; `stage-3b-acceptance` 74d841b; `stage-4-plan` 370edb5; `stage-5-validation-prep` 72b640e (main + one document). The older branches diverge from baseline bd8d3ba, not from `main`. No existing branch is named `stage-0`, so `stage-0/design-verification` creates no ref conflict.

## 9. Blockers and decisions needed

| # | Item |
|---|---|
| B1 | Commit policy for patches and builds (Issue C): confirm documents-only on stage branches, or direct otherwise. |
| B2 | Name indices: confirm names are appended at indices **29–58** (not 30–58) (Issue A). |
| B3 | Decision 4 is read as choosing one `saveString` that may hold a night above 30. An unmodified original game cannot continue such a save (expected failure, SV3). Confirm this rollback limitation is accepted. |
| B4 | Parent chain for later stages (section 5), and approval of each stage separately. |
| B5 | Approval for a no-op round-trip proof for `Level.as`, `HUD.as`, `MainTimeline.as`, `Medals.as` before S2. |
| B6 | Approval of the save-test set (SV0, SV3, SV4, SV5, SV6 minimum; UV series for the unified build) and the `SAVE\` folder / generator script. |
