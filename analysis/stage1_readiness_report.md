# Deadly 30: Extended Survival Edition — Stage 1 Readiness Review

Status: **PLANNING ONLY. Nothing here is implemented, and nothing here is approved for implementation.**
Written for: the project owner, as the gate before any Stage 1 work.
Date: 2026-10-17. Contains no game code, assets or saves.

Evidence tags:
- **V** = verified in a file or by a read-only command during this review (location given).
- **P** = stated in an existing plan or report and not re-checked here.
- **A** = assumption or inference, not verified.
- **REC** = my recommendation.

## 0. What was read, and naming

"Stage 1" below is **Stage 1 of `stage4_implementation_plan.md` ("Mode flag and save-slot separation")**. The earlier "Stage 1" of `newplan.md` was the audit and is finished. (V)

Inspected: `stage4_implementation_plan.md` (read from branch `stage-4-plan` with `git show`; it is not in the `main` working tree), the Stage 0b report (`game\spike_0b\STAGE0B_STATIC_REPORT.md`), the static scan (`game\spike_0b\scan\SCAN_REPORT.md`), the harness scripts (`build.ps1`, `gates.ps1`, `selftest_results.json`, `smoke.ps1`, `config\recompile_stage0a.txt`), the `src_baseline` call sites named below, and the user's D1–D5 approval message in the session transcript.

There is **no separate Stage 1 plan, prerequisites file or decision-record file**. (V) The only Stage 1 text is §3 "Stage 1" and §0 of the Stage 4 plan. The D1–D5 approvals exist only in the conversation transcript. The project memory note still says the five decisions are "NOT approved yet", which is out of date. (V)

## 1. What Stage 1 is meant to accomplish (P, plan §3)

Guarantee that Classic data can never be touched by Extended code, **before any Extended feature exists**. Concretely: add an `extended` flag and a `saveSlot` to `GameController`; route `getSharedObject` through the slot (Extended = slot 2, file `so_deadly302`); keep display settings and "Clear Save Data" on slot 1; and later add a menu control to choose a mode (Classic by default on every launch). No new gameplay, no new nights.

## 2. Decision status (V, from the user's approval message)

| # | Status | Binding wording that matters for Stage 1 |
|---|---|---|
| D1 separate saves | **Approved** | Separate Extended slot (`so_deadly302`). "Display settings may remain shared only if this can be implemented without risking gameplay-save data." "Before implementing save routing, document every save call site and its intended slot." |
| D2 legacy import | **Approved** | Never auto-convert, move or overwrite a Classic save. A future import must copy, ask for confirmation, and leave the original unchanged. Not part of Stage 1. |
| D3 mode selection | **Approved in principle; implementation deferred** | Classic is the default on every launch. "After Stage 0b, present the UI options and their impact on the original menu for approval." **That presentation has not happened, so the mode toggle is not authorized.** |
| D4 bug fixes | **Approved with guardrails** | Classic keeps original behavior and bugs. Crash-class hardening only if proposed separately and shown not to change valid-save behavior. |
| D5 runtime | **Deferred** | Keep using the verified Ruffle setup. Never modify the original SWF or EXE. |

No authorization to start Stage 1 or any implementation exists. E4 and E5 (Stage 0b) were authorized but have not been run. (V)

## 3. Verified current state

- Original SWF `3B9ABEE6…FC9C`, 19,250,808 bytes; protected hashes tracked in `spike_0b\hashes_*_0b.txt`. (V)
- **Stage 0a done:** build and gate harness. The patch applier passed its self-test (good diff applied, stale diff rejected), as did the tag, source-diff, hook, manifest and protected-files gates. (V, `selftest_results.json`)
- **Stage 0b static work done:** E0–E3 and the reuse scan. Controlled O1/C1 pair: 0 `#1009` errors each, identical 35-line traces. (V, Stage 0b report)
- **Not done:** Stage 0c (Classic finale baseline, shop UI, night 1 dawn, nights after 13), E4 (embedded-font text), E5 (`gotoAndStop` past the last frame), the byte-patched-save recipe (untested), the Classic static oracle script. (V: no such scripts exist in `game\harness`.)
- `game\run\deadly30.swf` is 19,273,450 bytes, so it is **not** the original SWF (a harness build). It was not hashed in this review. (V size only)

## 4. Save call sites and intended slots (D1 prerequisite, draft)

All 14 `GameController.getSharedObject()` call sites in `src_baseline` use the default argument, i.e. slot 1 today. (V, grep; matches the plan's list.)

| Site | Operation (keys) | Kind | Proposed slot |
|---|---|---|---|
| GameController.as:140 `setResolution` | writes `resolution` | display | 1, explicit |
| GameController.as:295 `setup` | reads `resolution`, `quality` | display | 1, explicit |
| Settings.as:86 | writes `quality` | display | 1, explicit (but see F2) |
| MainTimeline.as:156 (menu) | reads `nightsLasted` | gameplay | current mode |
| Controller.as:195 | writes `nightsLasted` if higher (win-cutscene path per plan, P) | gameplay | current mode |
| Controller.as:226 `loadSavedVariables` | reads `achievedMedals`, `achievedGuns`, `lvl` | gameplay | current mode |
| Controller.as:520 (game over) | writes `nightsLasted` if higher | gameplay | current mode |
| Level.as:1497 | writes `achievedGuns` | gameplay | current mode |
| Level.as:1700 | writes `saveString` (checkpoint) | gameplay | current mode |
| Level.as:1792 | reads `saveString` (Continue) | gameplay | current mode |
| HUD.as:357 | writes `lvl` | gameplay | current mode |
| MedalManager.as:45 | writes `achievedMedals` | gameplay | current mode |
| ClearSaveDataButton.as:19 | reads `size` (enables the button) | clear | 1 (unchanged) |
| ClearSaveDataButton.as:72 | `clear()` then resets in-memory guns/medals | clear | 1 (unchanged) |

## 5. Findings from this review that change or extend the plan

**F1. The plan's routing mechanism probably will not compile as written. (A)** The plan says to "change the default so an omitted argument resolves to `saveSlot`". ActionScript 3 requires default parameter values to be compile-time constants, so `param1:int = saveSlot` is expected to be rejected (FFDec's compiler behavior not tested). REC: use a sentinel: `getSharedObject(param1:int = 0)` and resolve `0` to `saveSlot` inside the body; sites that need slot 1 pass `1` explicitly. With `saveSlot` fixed at 1 the resolved name is `so_deadly301`, identical to today (V: `GameController.as:271-275` builds `"so_" + gameName + param1`).

**F2. Display settings live inside the gameplay SharedObject. (V)** `quality` and `resolution` are keys of the same slot-1 object as `saveString`, `nightsLasted`, `achievedGuns`, `achievedMedals` and `lvl`. Three sites that remain on slot 1 are reachable from an Extended session: GameController.as:140, Settings.as:86, ClearSaveDataButton.as:72. Consequences:
- The plan's acceptance, "after an Extended session the Classic file is byte-identical", can fail whenever the player opens Settings in Extended mode, even though only a display key changed.
- D1 allows shared display settings **only if** this does not risk gameplay-save data. That condition has not been shown to hold.
- Options for the owner: (a) share through slot 1 and change the test to compare gameplay keys instead of the file hash; (b) **REC:** make display settings per-slot, so Extended never writes slot 1 and "slot-1 file hash unchanged" stays a strict, simple test (cost: Extended starts with default quality and the player sets it once); (c) a third SharedObject for display only (also needs a read of the old slot-1 values on first run, more code).

**F3. Statics carry over between modes inside one running process. (V code, A impact)** `Controller.loadSavedVariables` (Controller.as:224-243) assigns `achievedMedals`, `achievedGuns` and `lvl` only when the stored value is truthy, and these statics (`exp`, `lvl`, `achievedMedals`, `achievedGuns`, `cutscenesPlayed`, Controller.as:69-97) are never reset. After a Classic game, choosing Extended on the menu and starting a new game would leave Classic's unlocked guns, medals and level in memory, and later writes (Level.as:1497, MedalManager.as:45, HUD.as:357) would copy them into slot 2 (and the reverse). That is a silent cross-mode leak. `cutscenesPlayed` is read at Level.as:2356-2358; its effect was not examined. REC: reset these statics when the mode changes, and include a Classic-to-Extended-to-Classic switch in one process in the Stage 1 tests. The plan lists replay-state checks only for Stage 12 and the matrix.

**F4. The menu setup runs once per menu entry. (V, MainTimeline.as:154-199)** `nightsLasted` is read, the "You Lasted N Nights" text is set, and Continue is either wired or greyed (a filter plus `mouseEnabled = false`) at that moment. A mode switch must redo all of it in both directions. This only matters once a toggle exists (see D3).

**F5. The harness default recompile list is the unproven one. (V)** `config\recompile_stage0a.txt` lists 18 classes (Level, Controller, HUD, Soldier, …). That is the class set whose earlier runs showed `#1009` errors under confounded conditions. The clean O1/C1 evidence covers only 2 classes (`Nights`, `Deadly30_fla.MainTimeline`). `build.ps1` accepts `-RecompileList` and `-PatchOrder`, so a minimal list and a candidate order file can be used without editing `patches\order.txt`. (V, `build.ps1` parameters)

**F6. Harness gaps for Stage 1. (V)**
- The patch path (`git apply` → recompile → gates) has been self-tested but never run end to end on a real patch.
- The source-diff gate allows extra lines through a flat allow-list of normalized residual lines (`AllowedFile`), which is coarser than "only these hunks".
- `smoke.ps1` uses `--storage memory`, has no Continue step and no save-isolation check. No disk-save test script exists.
- The Classic static oracle (plan §6.1) is not implemented. Stage 1 does not touch wave data, but Stage 2 will need it.

## 6. Smallest sensible first step: "Stage 1, slice 1A" (REC)

A behavior-neutral change to **one class**, `GameController.as`:
- add `extended:Boolean = false` and `saveSlot:int = 1` beside `survival`/`gameContinue` (GameController.as:55-57, V);
- change `getSharedObject` to the sentinel form in F1, so every existing call still resolves to slot 1.

Why this first: it is the smallest real code change, nothing can select slot 2 in a production build, and it proves the whole path (non-empty patch → build → gates → controlled runtime comparison → protected-hash check) on a change that should not alter any behavior. It deliberately excludes the menu toggle (D3 not authorized), the display/clear explicit-slot edits (F2 undecided; not needed while `saveSlot` is always 1), and all Extended gameplay.

Files and systems affected:
- Game source: `GameController.as` only, via a private patch directory under `game\patches\` (never committed). Recompiled with `-RecompileList` set to that single class. `DoABC2` is the only tag expected to change.
- Lab only: a candidate patch directory, a candidate order file, a build under `game\builds\`, and test outputs. `patches\order.txt` stays empty until acceptance.
- Not touched: the original SWF/EXE, `src_baseline`, backups, the real save, Level.as and every other class.
- Slot 2 could be exercised only in a separate `TESTHOOK` test build that sets `saveSlot = 2`; that hook is never in a production build (hook gate).

Later slices (each separately approved): 1B explicit slot-1 arguments at the display and clear sites, plus the F2 decision; 1C mode-switch state reset (F3) with the D3 UI; 1D menu text and Continue re-evaluation.

## 7. Technical risks that remain after O1/C1

1. **`#1009` is not closed (V).** One clean pair is one sample per build. C2–C5 and the 18-class build (21–36 errors earlier, confounded) were never re-tested under the controlled protocol. `GameController` recompiled alone has no evidence either way. Hypothesis only: the earlier errors came from load or overlap.
2. **FFDec's compiler is experimental (P).** The Stage 3 rebuild left 15 compiler-wrapper residual lines in 18 classes. `GameController` has static initializers and adding static fields changes class-initialization code. (A: low risk, but untested.)
3. **Ruffle-only evidence (P).** Behavior in the real Flash runtime is unverified. SharedObject persistence differs: Ruffle writes only on graceful exit and keys by SWF path; the real save lives under `%APPDATA%\Macromedia\Flash Player\#SharedObjects\…\so_deadly301.sol` (V, `gates.ps1:76`). Whether the wrapped EXE would create `so_deadly302.sol` correctly is unverified (D5 deferred).
4. **F2 and F3** above (shared display keys; cross-mode statics).
5. **Unpredictable window close (V).** Save tests need a graceful close; a failed close cannot be force-killed without asking, and a hard kill loses data.
6. **Missing tooling and baselines** (F6; Stage 0c not done). There is no recorded Classic Continue or slot-1 baseline on disk storage at the fixed path.
7. **Resources (V).** About 7.9 GB RAM; a window uses 0.5–0.9 GB; the harness requires ≥ 2 GB free before a launch.

## 8. How Classic, the original files and existing saves stay protected

- The original SWF and EXE are never modified or replaced. Every build is a new file with a manifest; the fixed test path is a copy. (P, V)
- Every candidate is a gated diff against `src_baseline`: tag gate (only `DoABC2`), source-diff gate (only listed hunks plus known wrappers), manifest gate, production hook gate, protected-files gate before and after every session. (V, harness)
- With `saveSlot` fixed at 1, slice 1A cannot change which file Classic reads or writes; this is checked, not assumed (section 9).
- Classic is the default and the only reachable mode until D3 is approved; the selection is kept in memory only, so Extended never persists a mode choice. (P; D3 text)
- No automatic migration or import of Classic data (D2). The real save is never opened for writing; tests use disposable copies. (P; standing rule)
- "Clear Save Data" is unchanged (slot 1 only). Note that from an Extended context it still clears the **Classic** save plus display keys; that is today's behavior, not a new risk, but the owner should know it before the toggle ships. (V, ClearSaveDataButton.as:72-77)
- Classic bugs are preserved (D4). Menu visuals are not changed by slice 1A.

## 9. Test and rollback strategy (REC)

Acceptance criteria for slice 1A (each reported as Passed / Failed / Skipped / Unverified, with commands and hashes):

| # | Check | Pass condition |
|---|---|---|
| A1 | Tag gate | Exactly 1 differing tag (`DoABC2`) vs the original |
| A2 | Source-diff gate | Residual lines = known Stage 3 wrappers + the listed 1A hunks only |
| A3 | Hook / manifest gates (prod) | 0 hook markers; manifest matches |
| A4 | Controlled runtime comparison, original vs 1A, one window at a time (O1/C1 protocol: memory check, 10 s menu dwell, new game to game over, bounded graceful close, no force-kill) | 0 errors; trace sequences identical to the original's |
| A5 | Classic Continue from a disposable copy of the real-save backup, disk storage, fixed path | Same `LOADED STRING` / flow as the original on the same path |
| A6 | Slot-1 isolation: hash of the slot-1 file after an equivalent Classic session, original vs 1A | Equal |
| A7 | (Test flavor only) `saveSlot = 2` build: a game-over or checkpoint write lands in `so_deadly302`, slot-1 hash unchanged | Slot 2 written, slot 1 unchanged |
| A8 | Protected hashes (13 entries), backup manifest, `src_baseline`, real-save hash and timestamp | Identical before and after |

A5–A7 need a new disk-save test script (does not exist yet) and, for A7, a test-flavor hook patch.

Rollback: nothing is promoted until acceptance, so rollback is "do not promote". Candidate builds use a separate order file; `patches\order.txt` is edited only with explicit approval after acceptance. Going back later is rebuilding from the previous order file and re-running its gates. Original, baseline, backups and real save are never modified. If any check fails: stop, report the original output, retry at most with a changed approach (max 3), then stop.

## 10. Unresolved decisions and prerequisites

| # | Item | Needs |
|---|---|---|
| U1 | Authorization to start Stage 1 at all | Owner |
| U2 | Display settings: shared (a), per-slot (b, REC) or separate object (c) (F2) | Owner |
| U3 | D3: the UI options and their effect on the original menu, still to be presented; E4 (embedded font) not run | Owner approval; E4 runtime |
| U4 | Does Stage 1 need all of Stage 0c? REC: slice 1A needs only the disk-storage Classic Continue and slot-1 baselines (new script); the finale, shop and nights > 13 baselines are needed from Stage 2/3 onward. The plan currently lists 0c as a prerequisite for everything. | Owner |
| U5 | Approval for a disk-save test script and a `TESTHOOK` test patch (the hooks order file is protected) | Owner |
| U6 | Recompile policy: minimal list (REC) vs the 18-class default (F5) | Owner |
| U7 | Whether to extend the `#1009` comparison (C2–C5, 18-class build, a repeat pair) before or after the first patch | Owner (currently not permitted) |
| U8 | Editing `patches\order.txt` on acceptance, or keeping a candidate order file | Owner |
| U9 | A written decisions record in the repo docs (D1–D5 currently only in the transcript; memory note is stale) | Owner approval; I would update the memory note |
| U10 | Git: a `stage-1-…` branch for this report and later docs; commit only approved files; no push or merge unless asked | Owner |
| U11 | D5 (end-user runtime) stays open until release testing | Owner, later |

## 11. Features to defer (current evidence does not show them feasible or safe)

- **New classes, characters, art or sounds for new weapons, enemies and traps.** The CLI cannot add classes or characters (E1, E2c, E3 proven). Reuse is at most 1 sound (`gameRestarts`), 2 small bitmaps and 2 generic clip hosts, with sprite import untested (scan report).
- **Mode-selection art.** A programmatic text toggle needs E4 first; art needs a proven workflow.
- **Fence levels above 5** (E5 not run), **enemy tint variants** (E3c not run), **XML add-a-character** (not attempted; the XML route re-encodes 142 tags and is not an acceptable patch route).
- **Replacing used assets** (shared SWF changes Classic; fidelity tested on a small sample only).
- **Stage 3 onward** (nights 31–60, night-30 milestone, finale at 60, loader hardening, events, upgrades, XP): they involve `Level.as` (3,597 lines) and many more recompiled classes, with the Classic finale, shop and nights > 13 never baselined and the `#1009` question open.
- **Legacy import** (D2), **Extended-only bug fixes** (D4, each proposed separately), **save-format versioning** (Stage 3+).
- **Any claim about the real Flash runtime** or the wrapped EXE (D5).

## 12. Recommendation in one paragraph

Do not start implementation yet. When you are ready, the safest path is to (1) decide U2, U4, U6 and U7, (2) approve a lab-only disk-save test script and baseline capture, then (3) approve slice 1A on `GameController.as` alone, with a minimal recompile list and a candidate order file, checked by the A1–A8 criteria above. The mode toggle, the state reset (F3) and the display/clear edits come only after D3 and U2 are decided. Nothing in this report has been run, built or tested.
