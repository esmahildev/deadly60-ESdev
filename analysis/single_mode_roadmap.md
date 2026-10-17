# Deadly 30 — Single-Mode Direction: Revised Roadmap (Planning Only)

Status: **PLANNING ONLY. Nothing here is implemented, built, tested or approved for implementation.** The only approved change is the owner's direction: **one game mode, no Classic/Extended split.**
Written for: the project owner, as the gate before any implementation.
Date: 2026-10-17. Contains no game code, assets or saves.

Evidence tags: **V** verified in a file or by a read-only command (location given) · **P** stated in an earlier report/plan, not re-checked here · **A** assumption or inference · **REC** recommendation.

---

## 1. Understanding of the new direction

- One game, recognizably the original Deadly 30: original art, controls, weapons, enemies, balance, menus, HUD, sound style. It is expanded, not redesigned.
- New content (weapons, zombies, nights, traps/defences, upgrades, art/sound) arrives gradually, each item only after its own feasibility review.
- No mode selector, no second save slot for a former Extended mode. Prefer the existing save system; keep existing progress compatible.
- The real save is never overwritten or migrated without explicit approval; all testing uses disposable copies.
- Approved by this message: **only** the single-mode direction. Everything else below is a proposal.

**Consequence worth stating plainly (V/A):** with one mode there is no longer a control group in the same build. Every change affects the only game, so "the original still plays the same" must be proven for the *whole* game, permanently, not for a flag-off branch.

---

## 2. What the previous plans said and what is now obsolete

| Earlier item | Status now | Reason |
|---|---|---|
| D1 separate Classic/Extended saves (`so_deadly302`) | **Obsolete** | No second mode. The owner's new text replaces it with "prefer the existing save system where safe". |
| D2 legacy-save import | **Obsolete** | Nothing to import into; the existing save *is* the save. (Its principle, "never convert, move or overwrite a save without approval", survives, see D-S1 below.) |
| D3 mode-selection UI (interim text toggle) | **Obsolete** | No selector. Removes the need for E4 as a mode-toggle prerequisite (E4 remains useful for other new UI text). |
| D4 bug policy (Classic bugs kept, Extended fixes) | **Needs rewrite** | With one game, any fix changes the only game. REC: original behavior, bugs included, is preserved by default; every individual fix needs separate approval and a before/after test. |
| D5 runtime / end-user deployment | **Still open** | Sharper now: a modified game reading the *same* save as the original means running it on the real save affects real progress. |
| Stage 1 (mode flag and save slots), including `GameController.saveSlot`/`extended`, the sentinel `getSharedObject`, explicit slot-1 arguments, menu toggle | **Obsolete** | All of it existed only to separate modes. |
| My Stage 1 readiness report: slice 1A, findings F1 (constant default parameter), F2 (display keys share the Classic file), F3 (statics leak between modes), F4 (menu re-initialization), acceptance A5–A7 (slot isolation) | **Obsolete** | They solve problems that no longer exist. That report is kept as history and not edited. |
| Stage 1 readiness findings F5 (18-class recompile list is the unproven set), F6 (harness gaps), risks on `#1009`, Ruffle-only evidence, window-close behavior, resource limits | **Still relevant** | Independent of modes. |
| Stage 2 (`finalNight` plumbing, "Classic bit-identical") | **Relevant, renamed "original-equivalence"** | The 30-night sites still need one accessor if the night limit ever moves. |
| Stage 3 (Extended skeleton: nights 31–60, night-30 milestone, finale at 60, loader hardening, Extended-only `nightsLasted` fix) | **Relevant but its premise changes** | The night-30 and ending behavior now affect the only game (decision ND-1). The `nightsLasted` off-by-one can no longer be "Extended-only"; it is preserved unless separately approved. |
| Stages 4–12 (difficulty, events, repair, upgrades, XP, zombies, art, final verification) | **Relevant, regrouped and re-gated** | See roadmap in section 8. |
| Hostile-save and slot tests | **Partly relevant** | Hostile-save tests remain valuable (loader crash-class hardening, D4 rules). Slot tests are dropped. |

---

## 3. Decisions that remain relevant

| ID | Decision | Status |
|---|---|---|
| D-S1 | The real save (`so_deadly301.sol`) is never opened for writing, overwritten or migrated without explicit approval. All tests use disposable copies. | Standing rule (P, owner) |
| D-S2 | Save format is extended only by **appending** data and never by changing the meaning of existing sections (see section 6). | REC, needs approval |
| D-R1 | Rollback = rebuild from the previous patch set; original SWF/EXE, baseline export, backups and the real-save backup stay read-only and hash-verified. | Standing (P) |
| D-G1 | Gates on every build: tag gate (only `DoABC2`), source-diff gate, manifest gate, production hook gate, protected-files gate before and after every session. | Standing (V, harness) |
| D-T1 | Test hooks use the `TESTHOOK` marker and exist only in disposable test builds. | Standing (P) |
| D-C1 | Run one Ruffle window at a time, verify exit, check memory first, no force-kill without asking. | Standing (P) |
| D-R2 | Report results as Passed / Failed / Skipped / Unverified with exact commands and hashes. | Standing (P) |

### New decisions the owner must make (nothing below is assumed)

| ID | Question | Why it matters | My recommendation (REC) |
|---|---|---|---|
| ND-1 | **What happens at night 30 and at the ending?** In the original, night 30 starts a 5:00 countdown, the helicopter arrives, cutscene 6 plays and the game returns to the menu (V: Level.as:340, Controller.as:170-179 and 245-278, per the audit). One game cannot both end at 30 and continue to 60. Options: (a) night 30 stays the original ending, and nights 31+ exist only as a continuation after it; (b) night 30 becomes a milestone and the helicopter finale moves to the last night (the plan so far); (c) keep 30 nights and only add content inside them. | Changes or preserves the original ending for everybody. Also drives the menu text (`nightsLasted == 30` shows "Survivors have escaped!", V: MainTimeline.as:163-170) and Medal 18's text. | Decide before any night-limit work. (a) preserves the original ending exactly but needs a continuation mechanism; (b) matches the earlier plan but replaces the original ending; (c) is the most faithful and the least ambitious. |
| ND-2 | **Where may new content appear?** If new weapons, zombies or upgrades can appear in nights 1–29, the original difficulty curve and balance change. | The owner requires the original curve and balance to stay. | New content gated to nights 31+ (or to an explicit opt-in purchase) by default; anything earlier needs per-item approval. |
| ND-3 | **When, if ever, may a modified build touch the real save?** | Section 6 shows saves written by a new build can break the original game. | Not before the build is accepted; then only after a verified backup and explicit approval. |
| ND-4 | **Who produces new art and sound, and what is the style bar?** | The tools cannot create art; they can only place or replace it (section 5). Code-drawn visuals may not match hand-drawn style. | Owner supplies or approves assets; decide per feature whether code-drawn visuals are acceptable. |
| ND-5 | **Recompile policy:** minimal list of only the touched classes, or the harness's 18-class default. | The 18-class set is the one that showed `#1009` errors under confounded conditions (V). | Minimal list per patch. |
| ND-6 | **Deployment (D5).** | See above. | Defer, but decide before any real-save test. |
| ND-7 | **Which feasibility experiments (section 7) are authorized, one by one.** | Each needs Ruffle or FFDec runs, and several need a lot of memory. | Authorize individually. |

---

## 4. Verified facts this plan relies on

- Original SWF `3B9ABEE6…FC9C`, 19,250,808 bytes; 2,364 top-level tags; all code is in one `DoABC2` tag. (V, Stage 0b report)
- Recompiling classes with FFDec `-replace` changes only `DoABC2`. The controlled O1/C1 pair (recompiled `Nights` + `Deadly30_fla.MainTimeline`, unmodified source) had 0 `#1009` errors and identical 35-line traces. It does not prove other classes or modified code are safe. (V)
- The earlier error runs (C1–C5, harness build A, E1d) were confounded and are inconclusive; C2–C5 and the 18-class build were never re-tested under the controlled protocol. In particular **C5 ("Nights only") had 36 errors in the confounded batch and has not been re-run cleanly.** (V)
- FFDec CLI cannot add classes (`-importScript` ignores new files, `-replace` refuses unknown names), cannot add characters (`-importShapes/-Images/-Sounds` report "0 imported"), and cannot duplicate characters (`-replaceCharacter` swaps ids). It can replace shapes, images and sounds by id with one-tag precision, with some re-encoding loss. (V, E1–E3)
- An internal helper class declared inside an existing class file compiled and printed its trace at runtime in Ruffle (one probe). (V, E1d)
- The reuse scan found **no unreferenced characters**; weak candidates are 1 sound (`gameRestarts`, id 1801), 2 small opaque bitmaps (444, 445), generic clip hosts 434/441 and an NG cluster, with sprite import untested. (V, scan report)
- Wave data is code (`Nights.addNights`, 30 rows, 29 names, V: Nights.as:11-73). Enemy stats are linear in `nightCount` with no cap (P).
- Save format: one SharedObject per slot named `"so_" + gameName + slot`; today everything uses slot 1 (14 call sites, V). The checkpoint `saveString` is `|`-separated; the loader indexes sections 0–5 unconditionally (V: Level.as:1805-1811) and ignores anything after section 5.
- Bought-weapon ids (section 3) are matched against `Controller.weaponArray`; an unknown id is silently ignored (V: Level.as:1851-1862). Section 2 ids go to `equiptWeapon`; behavior with an unknown id was not examined.
- The loader's last loop reads `waveArray[0 .. nightCount-1]` and calls `.length` on each row (V: Level.as:1888-1899). `waveArray` has 30 rows today.
- `Controller.loadSavedVariables` runs at `Controller.init` and `restart` (V: Controller.as:125, 147) and reads `achievedMedals`, `achievedGuns`, `lvl` from the same SharedObject.
- Ruffle writes SharedObjects only on a graceful exit and keys them by SWF path; the real save lives in Flash Player's `#SharedObjects` folder (V/P).
- Harness: the patch applier passed a self-test but has never run on a real patch; there is no disk-save test script; the Classic static oracle script does not exist; `patches\order.txt` and `hooks_order.txt` are empty and protected. (V)

---

## 5. Realistic implementation approaches, compared (no option is assumed feasible)

| Approach | Good for | Evidence today | Main risks / unknowns |
|---|---|---|---|
| **A. Modify existing classes** (recompile with `-replace`) | Everything that is logic or data: new nights, stats, enemy parameters, upgrades, repair, trap behavior, event rules | O1/C1 clean for `Nights` + `MainTimeline` (V). Helper classes inside existing files work (one probe, V). | Experimental compiler (residual wrapper lines, P). `#1009` question open for other class sets. `Level.as` is 3,597 lines and touches most systems. |
| **B. Reuse existing assets** (same clips with different stats, tints, frame choices) | Zombie "variants", reskinned defences, extra sound uses | Scan: almost nothing is spare (V). Tint at runtime is **not tested** (E3c). | New enemies would look like existing ones. Judge against the style/identity requirement. Existing sounds can be reused in new roles in code. |
| **C. Code-drawn visuals** (Sprite/Graphics, filters, tints at runtime) | Simple effects, traps, bars, markers, bullet/muzzle effects, overlays | **Not tested.** No reason found yet why it cannot run in Flash/Ruffle. | May not match the hand-drawn look (ND-4). Rendering parity between Ruffle and Flash unverified. |
| **D. Replace existing assets by id** | Never for assets Classic-era gameplay uses | Works in isolation (E2), with alpha/sound re-encoding loss (V) | Changes the only game. Only acceptable for assets that are verifiably never shown in nights 1–29, which the scan cannot prove at runtime. |
| **E. XML round trip to add characters** | Possibly new shapes/images/sounds | Round trip re-encodes 142 tags (V). Content equal at export level (V). Adding a character not attempted; needs a 144 MB XML and about 3 GB Java heap. | Memory-heavy on this laptop; changes many tags; how Ruffle/Flash render the re-encoded tags is unchecked. The earlier plan judged it unacceptable as a patch route. Only as a separate, disposable experiment. |
| **F. Other SWF editing** (a small tool of ours that *appends* new tags without touching existing ones; `-abcmerge`; sprite import) | Adding characters while leaving all existing tags byte-identical | Not tried. We already have a validated tag parser (V). `-abcmerge` exists in FFDec help (V) but its behavior was not explored. | New *classes* still need compiled ABC from some compiler, which we have not evaluated. Pure research until a disposable experiment says otherwise. |
| **G. Defer or simplify** | Anything where A–F fail | Always available | Scope reduction needs the owner's agreement. |

REC: prefer **A**, then **B/C** for visuals, treat **E/F** as research spikes, and say "not possible" when evidence does not support a feature.

---

## 6. Save compatibility and rollback in a single mode

**Verified/derived (V + A):**
1. A save from the **original** game loads in a build that only appends logic: no format change.
2. A save written by a **new** build is a problem for the **original** game (this matters for rollback):
   - A checkpoint with `nightCount` above 30 would make the original loader read `waveArray[30]`, which does not exist (V code; the crash is an inference, A, not tested).
   - New content ids written into existing id lists could break the original: unknown bought-weapon ids are ignored (V), but unknown equipped-gun ids and `achievedGuns` entries were not examined (A).
3. `Clear Save Data` calls `clear()` on the whole SharedObject (V: ClearSaveDataButton.as:72), so any new keys stored beside the old ones are removed with them. That is consistent with today's meaning, but it must be documented.
4. No version field exists today.

**REC:**
- **Append only.** New data goes after section 5 of `saveString` or into new SharedObject keys; never change the meaning of sections 0–5 or push new ids into the old id lists. Missing data means a safe default.
- Add a small version marker with the first appended data; a build that finds none behaves exactly as today.
- Until the build is accepted, every save-related test uses a disposable copy of a **backup** of the real save, on a fixed SWF path with a fresh disposable save directory.
- Rollback for code: rebuild from the previous patch set (no change to the original files). Rollback for data: the verified backup of the real save is the only safe way back once a new build has written to it; this is why ND-3 requires explicit approval.
- Loader crash-class hardening (empty or short string, `nightCount` beyond the wave table) is allowed only as a separately proposed change shown not to alter valid-save behavior (D4 wording kept).

---

## 7. Technical feasibility checks required before implementation

None of these is authorized yet; each needs your explicit go.

| ID | Check | Type | Answers | Go / no-go criterion |
|---|---|---|---|---|
| FC-0 | Tooling with no game change: disk-storage save test script, baseline traces of the original at the fixed path (Continue from a disposable copy of the real-save backup), Classic static oracle script (wave rows, caps, names, enemy stat formulas for nights 1–30) | Harness work + Ruffle runs | Can we prove "unchanged" for save and data? | Script reproduces the original's traces twice |
| FC-1 | Controlled `#1009` follow-up: re-run **C5 (Nights only)** against O1 under the O1/C1 protocol, then any exact class set a patch will use; consider a repeat pair | Ruffle, one window at a time | Is a Nights-only recompile clean? | 0 errors and identical traces |
| FC-2 | Static review of weapons: table at Level.as:1508-1519, `Soldier.as` behaviors, `Shop.as:416` slot alignment, blueprint/arm frames | Read-only | Is a new weapon data-driven or art-bound? | A written yes/no per dependency |
| FC-3 | Static review of enemies: id 0–10 creation path, stat formulas, hit-box/label structure, `Enemy.as:66` music rule | Read-only | Can a variant be added by parameters/tint alone? | Written answer |
| FC-4 | E3c runtime tint of an existing enemy symbol | Ruffle, test build with `TESTHOOK` | Are "variant zombies" possible without new art? | Tint visible, 0 errors |
| FC-5 | E4 (programmatic TextField with an embedded game font) and E5 (`gotoAndStop` past a clip's last frame) | Ruffle, test build | Can new UI text and extra fence/upgrade levels be shown? | Renders as expected |
| FC-6 | Code-drawn visual probe (one trap-like shape, one HUD element) compared with the original style | Ruffle, test build + your visual judgment | Is approach C acceptable (ND-4)? | Owner accepts the look |
| FC-7 | Sprite import and symbol re-binding in the FFDec CLI | FFDec CLI, disposable | Can the NG hosts or existing sprites be reused? | Tag-level diff shows only intended tags |
| FC-8 | Original loader vs a disposable save at night 30/31 | Ruffle, disposable save (needs the byte-patch recipe, untested) | Confirms or refutes the rollback hazard in section 6 | Observed behavior recorded |
| FC-9 | Research spike for adding characters: XML route (memory-safe plan, RAM free, nothing else running) and/or an append-only tag tool | Disposable, memory-limited | Is any route to *new* art/classes realistic? | Tag diff shows only added tags and a runtime proof |
| FC-10 | Check in the actual Flash runtime (the owner's wrapped EXE) before any release | Owner | Does the final build behave the same outside Ruffle? | Owner confirmation |

---

## 8. Smallest useful first implementation slice (REC)

**Prerequisite (no game code): FC-0 and FC-1.** The first slice should not be built until the baselines and the C5 control exist, otherwise there is nothing to compare against.

**Slice S1 — `Nights.finalNight()` accessor, unused and behavior-neutral.**
- **Change:** in `com/iggyzuk/deadly30/Nights.as`, add one public static function returning the current final night, `30`. No caller uses it.
- **Why this:** it is the smallest real patch; it exercises the whole path (patch apply → recompile → gates → controlled comparison → protected-hash check); it touches the class with the best clean-run evidence (`Nights` was part of the clean C1 build, V); it is the seed of later night-limit plumbing, so it is not throwaway work. (`Nights.as` has no static initializers, V: Nights.as:1-11, so adding one static function does not change class initialization.)
- **Exact files and classes affected:**
  - Game source: `com/iggyzuk/deadly30/Nights.as` only (patched in the build's working copy of `src_baseline`; `src_baseline` itself is never edited).
  - Recompile list: `com.iggyzuk.deadly30.Nights` only (a one-line config file, not the 18-class default).
  - Lab only, never committed: a patch directory under `game\patches\`, a candidate order file (so `patches\order.txt` stays empty), a build directory, test outputs.
  - Not touched: every other class, all assets, the original SWF/EXE, the real save, `patches\order.txt`.
- **Not in S1:** the plumbing at the 30-night sites (that is S2), any behavior, any new class, any asset, any save change.

**Slice S2 (separate approval):** replace each hard-coded night-30 site with `Nights.finalNight()`, still returning 30. Sites (V per the audit/plan, to be re-read before editing): Level.as:313, 340, 984, 1172, 1888-1899, 2695, 2781; HUD.as:122; MainTimeline.as:163-170; Medals.as:39 (text). This touches `Level.as`, `HUD`, `MainTimeline`, `Medals` and so needs FC-1 for that class set first. Left alone until ND-1 is decided: Level.as:2571/2575/2711 and Controller.as:245-278 (cutscene ids).

### Acceptance criteria for S1 (and the base for every later slice)

| # | Check | Pass condition |
|---|---|---|
| A1 | Tag gate | Exactly 1 differing tag (`DoABC2`) vs the original |
| A2 | Source-diff gate | Residual lines = known Stage 3 wrapper lines + the listed S1 hunk only |
| A3 | Hook / manifest gates (production) | 0 hook markers; manifest matches |
| A4 | Controlled runtime comparison, original vs S1, one window at a time (memory check, 10 s menu dwell, new game, unattended to game over, return to menu, bounded graceful close) | 0 `ERROR` lines; the complete trace sequences identical |
| A5 | Static oracle (once FC-0 exists) | Wave rows, caps, names and enemy stat formulas for nights 1–30 equal to the original |
| A6 | Continue from a disposable copy of the real-save backup on the fixed path, disk storage | Same `LOADED STRING` and flow as the original on the same path |
| A7 | Save write: a checkpoint written by S1 compared with the original's for the same starting state | Byte-identical `saveString` |
| A8 | Protected hashes (13 entries), backup manifest (391), `src_baseline` (381), real-save hash and timestamp | Identical before and after |

### "Original gameplay is intact" suite for later slices (REC)
Beyond A1–A8: (i) per-night wave digests and spawn stats for a spread of nights (1, 2, 5, 10, 15, 20, 25, 29) from the same test hook patched into baseline and candidate; (ii) shop prices and weapon table equal; (iii) menu, HUD and in-game screenshots at fixed points compared by hash where deterministic and by your visual review otherwise (`Math.random` is not seedable, V, so assertions are structural); (iv) frame-time/performance at the heaviest wave not worse than the original; (v) any new content is absent from nights 1–29 unless ND-2 says otherwise.

---

## 9. Phased roadmap for new content (REC, every phase needs its own approval)

| Phase | Content | Depends on | Notes |
|---|---|---|---|
| R0 | Tooling and baselines (FC-0, FC-1); decisions ND-1, ND-2, ND-3, ND-5 | none | No game change |
| R1 | S1 accessor | R0 | Pipeline proof |
| R2 | Night-limit plumbing, still 30 (S2) | R1, FC-1 for the class set | Behavior-neutral |
| R3 | Feasibility spikes FC-2 … FC-9, each go/no-go, disposable only | R0 | Decides which later phases exist |
| R4 | Nights beyond 30: wave data, names, stat compression/caps, night-30 handling per ND-1, loader crash-class hardening if approved | R2, ND-1/ND-2 | Data and logic only; lowest risk of the content phases |
| R5 | Zombie variants from existing sprites (tints/parameters), then behavior variants inside existing classes | R3 (FC-3, FC-4) | "New types" without new art; judge against the style requirement |
| R6 | Defences: repair, reinforcement, trap-like behavior using existing Box/Turret/fence clips and code-drawn effects | R3 (FC-5, FC-6) | Fence levels above 5 need E5 |
| R7 | Progression and upgrades (stats, shop numbers, appended save data) | R3, section 6 rules | Appended save data only |
| R8 | New weapons | R3 (FC-2 and an art route from FC-6/FC-9) | Hardest: shop, blueprint, arm/gun frames and sounds are art-bound until proven otherwise; may be reduced to rebalanced use of existing weapon art or deferred |
| R9 | New art/sound in the original style | FC-9 and ND-4 | Only if a workflow is proven; otherwise cut |
| R10 | Balance, performance, full regression, release gate, owner check in the real Flash runtime (FC-10) | all above | Final |

Mapping to the old numbering: old Stage 1 → removed; old 2 → R2; old 3 → R4 (+ ND-1); old 4–5 → R4/R5; old 6–7 → R6; old 8–9 → R7/R8; old 10 → R5; old 11 → R9/R8; old 12 → R10.

---

## 10. Remaining technical risks

1. **`#1009` is unresolved.** One clean pair, one sample per build; C2–C5 and the 18-class build not re-tested under control. Each new class set needs its own controlled comparison. (V)
2. **Experimental compiler.** Silent behavior differences are possible in any recompiled class; mitigated only by gates and runtime comparison. (P)
3. **Single-mode blast radius.** Every defect now reaches the only game; there is no flag-off fallback. (A, from the design)
4. **Rollback of saves.** Progress written by a new build may not load in the original (section 6). (V code, A behavior)
5. **Ruffle-only evidence.** Flash-runtime behavior, including save handling, is unverified (D5, FC-10). (P)
6. **No route to new art or classes is proven.** Weapons and any new-looking enemy depend on it. (V)
7. **Style fidelity of code-drawn visuals** is a judgment call that only you can make. (A)
8. **Test infrastructure gaps and unpredictable window close** limit how much can be proven automatically. (V)
9. **Resources:** about 7.9 GB RAM; XML experiments need more than is normally free. (V)
10. **Unseedable randomness** means "identical gameplay" is shown through structure (traces, tables, counts), not through identical random runs. (V)

---

## 11. What I recommend next (still only a recommendation)

1. Make decisions **ND-1** (night-30/ending), **ND-2** (where new content may appear) and **ND-5** (recompile list).
2. Authorize **R0**: FC-0 and FC-1 (tooling + the C5 control), no game change.
3. Then authorize **S1**.
4. Authorize feasibility spikes one at a time; weapons and new art stay deferred until R3 results exist.

Nothing in this document has been built, run or tested.
