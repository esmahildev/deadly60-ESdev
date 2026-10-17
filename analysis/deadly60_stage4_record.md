# Deadly 60 — Rollout Stage 4 record (text and Medal 18 wording)

**Result: the Stage 4 build compiled and all four static gates passed. Runtime behavior is NOT verified.** Static validation only.

Date: 2026-10-17. Branch: `stage-4/text-and-medal-wording`, created from `stage-3/atomic-60-night-switch` at c96aff0442bf067325754dcfa004e6d431e4897e (Stage 3 head; remote head and clean tree verified first). Not merged into `main`. The patch, SWF and lab artifacts stay in the isolated lab and are not in Git; this record contains no source code beyond the quoted user-visible text.

## 1. Scope and wording source

Stage 4 is text only. The only approved text change found in the project records is the **Medal 18 description**:

| Item | Value |
|---|---|
| Location | `Medals.as` line 39, medal list entry id 18, title "Survivor" |
| Old text | "Survive all 30 days and nights" |
| New text | "Survive all 60 days and nights" |

Support for this change: the project plan (`analysis/newplan.md`, Feature 1: expand the game to 60 days and nights), the committed Stage 0 record (Medals.as:39 "medal 18 text" assigned to this stage, "medal 18 wording" in the stage table), and the owner's design decision that menu, HUD and medal wording reflect the 60-night campaign. Disclosure: the committed records name the site and the intent but do not spell out the replacement string. The replacement used here is the minimal numeric substitution in the original's own wording; it was first proposed in the earlier design review and is recorded here for the first time as the applied text. No other wording was changed.

**Deliberately not changed** (no approved change in the records): the context-menu copyright string "Deadly 30 © 2012" (GameController.as), the title art, the menu texts "You Lasted N Nights" and "Survivors have escaped!" (their thresholds were already moved in Stage 3), and every other medal. The scan of all game-code string literals containing "30" found only the medal text, the copyright string, the old finale name (changed in Stage 3) and a frame-rate metadata value.

**Localization coverage:** the game has a single language. No dictionary, locale or translation structure exists in the game code (the only hit, `German.as`, is a soldier class). So exactly one string is affected and nothing needs translation.

## 2. The patch (lab only: `D:\Projects\deadly30_lab\game\d60_rollout\stage4\`)

| Item | Value |
|---|---|
| Diff | `patch\05_Medals.diff`, 2,323 bytes, SHA-256 `32ABFB928451512E6A3B076C99885CA4A14C0D83C9820D5ADAE8D99875913CD7` |
| Change | one line (39) of `Medals.as`; at character level exactly one digit changed ("3" → "6"), line length unchanged |
| Private order file | `order_stage4.txt`: the Stage 3 patch folder first, then the Stage 4 folder (a cumulative build; the protected `patches\order.txt` is untouched) |
| Allowed-residual list | `allowed_stage4.txt`, 85 entries = the reviewed Stage 3 amended list (83) plus the two intended Medals lines, registered before the build |

Verification on a disposable scratch copy: `git apply --check` and apply succeeded, the result equals the intended target byte for byte, 156 lines before and after, one differing line. The medal list keeps its 25 entries with identical ids, order and titles; the only description that differs is entry 18. Medal identifiers, unlock conditions, award logic (the helicopter's `addMedal(18)`), scoring, wave data, night limits, save structures and cutscene logic are untouched.

## 3. Build (existing `harness\build.ps1`, flavor `prod`, 18-class list)

| Item | Value |
|---|---|
| Run | `S4_TEXT_1`, `-PatchOrder` / `-OutRoot` with the private order file; exit code 0; about 25 s |
| Output | `D:\Projects\deadly30_lab\game\d60_rollout\stage4\builds\S4_TEXT_1\deadly30.swf`, 19,285,387 bytes, SHA-256 `155A51679DC1E4BFD73B4BBCDFCCFE772F955E9C5C148470DA84956B8DB7304D` |
| Patches applied | five: the four Stage 3 diffs and `05_Medals.diff` (Stage 1 data, Stage 2 cutscene bound and Stage 3 switch are included) |
| Tools | Java 1.8.0_503 (from the build's own `java -version`); FFDec jar SHA-256 `1587E60B…841C` (the harness pin) |
| Recompiled `Medals` source | `35E4892D…A565`; the Level, HUD, MainTimeline and Nights sources are identical to the Stage 3 build's |
| Java runs | 4 (compile, `java -version` inside the build manifest, tag dump, script export) |

## 4. Memory gate (simple fail-closed check before each Java step)

2.94 GiB before the build (the build's internal `java -version` cannot be gated separately without modifying the harness), 2.94 GiB before the tag gate, 2.86 GiB before the source gate. All passed (required 2.00 GiB). Free memory afterwards 2.94 GiB, 0 java/ruffle processes.

## 5. Gate results (each run individually; `run_gates.ps1` and real-save hashing were not used)

| Gate | Result |
|---|---|
| Manifest | PASS: 5 patches, 18 recompiled, 381 source files |
| Hook marker | PASS: 0 hits over the five diffs and the build sources |
| SWF tag | PASS: only `DoABC2` differs from the original |
| Decompiled-source diff | PASS: 381 files, 98 residual lines = the 15 documented + the 79 reviewed Stage 3 lines + the 2 Medals lines + 2 Stage 3 amended lines; 0 unexpected, 0 expected-but-absent |

Additional checks:
- Byte-level tags (own parser): exactly one tag, `DoABC2`, differs from the original (305,236 → 339,815 bytes) and exactly one from the Stage 3 build (339,784 → 339,815). The extra 31 bytes equal the new 30-character string plus its length byte, so the old string most likely remains unused in the constant pool [I].
- Whole decompiled tree compared with the Stage 3 export: **only `Medals.as` differs** among the 381 files.
- The decompiled `Medals.as`: 25 entries, equal to the target; differs from the baseline only in the description of entry 18; every other line identical, including the line wrap at the end of the list (that wrap already exists in the baseline decompile).
- Protected files: 26 hashed items (original SWF and copy, FFDec jar, harness scripts and baselines, protected order files, Stage 1–3 patches, lists and SWFs, baseline sources) identical before and after. The real save was not opened, hashed or listed.

## 6. Limitations

- **Runtime is not verified.** Nothing was run in Ruffle or the game; how the new text renders in the medal screen is untested.
- B5 Lite remains an evidence-based deferral; **no per-file FFDec round-trip pass is claimed** (the stage relies on the build's own gates and the aggregate 18-class evidence).
- The Stage 3 source-diff result rests on its reviewed amended allow-list (documented in the Stage 3 record); this build's list inherits it.
- FFDec's AS3 compiler is experimental; a successful compile does not prove identical runtime behavior.
- No SAVE folder, generator, launcher or save access; no Ruffle or game launch; Stage 5 has not started.
