# Deadly 60 — Rollout Stage 2 record (Cutscene Option Y)

Date: 2026-10-17. Branch: `stage-2/cutscene-option-y`, created from `stage-1/b5-b6-decisions` at 9b76132745bd81cb16dbcafc3cf119b0f49b097d. Not merged into `main`.

**Result: the Stage 2 build compiled and all four static gates passed. Runtime behavior is NOT verified.** No Ruffle, game, save or SAVE-folder activity took place. The patch, the SWF and all lab artifacts stay in the isolated lab and are not in Git. This record contains no decompiled source beyond the one-condition description below.

## 1. Starting state (verified before any change)

Branch `stage-1/b5-b6-decisions` at 9b76132, clean tree, remote head identical, `stage-1/append-night-data` an ancestor, no existing `stage-2/*` branch (the older `stage-2-plan` branch is unrelated and untouched). Stage 1 patch `Nights.as` hash `F9EBFC29…8A22` equals its manifest. Free memory 2.85 GiB at that point.

## 2. Memory gate

Rechecked immediately before the first Java process: **2.91 GiB (≥ 2.00 GiB), gate passed**, and the check was enforced in the script that launched the build.

Disclosure: for the two later Java steps (tag gate, source gate) a memory read was coded into my gate-runner command but, because of a bug in that ad-hoc command (the check returned an array, so the stop condition could never fire), the check was **not enforceable and its value was not recorded**. The machine was not memory-starved (the build moments earlier ran at 2.91 GiB) but this is a process defect, not evidence of the gate.

## 3. The change

Only `com/iggyzuk/deadly30/Level.as`, only the night-end handler at line 2711: the condition `nightCount % 5 == 0` gained `&& nightCount <= 25`. The original line was present exactly once in the baseline; the unrelated music check in `Enemy.as` was not touched. Intended behavior [I, not runtime-tested]: nights 5–25 keep their cutscenes; later nights (30–55 once Stage 3 makes them reachable) take the existing `else` path to `addDay()` immediately.

Applying the one-line diff to a disposable scratch copy succeeded (`git apply --check` and apply, exit 0) and produced exactly one differing line (line 2711) out of 3,599. This only shows the patch applies; it is not a compile result.

## 4. Build and evidence (lab only)

| Item | Value |
|---|---|
| Lab folder | `D:\Projects\deadly30_lab\game\d60_rollout\stage2\` (patch, private order file, allowed-hunk file, scratch copy, builds, gate results, protected-hash snapshot) |
| Diff file | 381 bytes, SHA-256 `298BAA9E1C84E2480CE33211694DBF2A03EA981CD4965AE2B00AB4787DEFA87F` |
| Baseline `Level.as` | `BB2CE71F63E356A7AE4D38B6EDE0AFC69E2CD16146886AA16D01FD5E62E4E272` (unchanged) |
| Patched `Level.as` used in the build | `B11A00F0DF615968626182703C0463007734E04A8E3DED38D7BBD555B0D8C77F` |
| Output SWF | run `S2_OPTION_Y_1`, 19,273,464 bytes, SHA-256 `38A061580062EB8AEB520D64A4304E69C486B030702B59D2BF07318A6568D1A6` |
| Tools actually run | Java 1.8.0_503 (HotSpot 64-bit, from the build's own `java -version`); FFDec jar SHA-256 `1587E60B…841C`, equal to the pin. The label "26.3.0" comes from the harness pin, not from querying the tool. |
| Build | existing `harness\build.ps1`, flavor `prod`, 18-class recompile list, one patch, via `-PatchOrder` / `-OutRoot` (protected `patches\order.txt` untouched); exit 0, about 28 s |
| Java runs | 4 of 4 authorized (compile, `java -version` inside the build manifest, tag dump, script export) |

## 5. Gate results (each run individually; the real-save gate and `run_gates.ps1` were not used)

| Gate | Result |
|---|---|
| Manifest | PASS: 1 patch, 18 recompiled, 381 source files |
| Hook marker | PASS: 0 hits (production flavor) |
| SWF tag | PASS: baseline 24,033 tag lines, candidate 24,033; only `DoABC2` differs |
| Decompiled-source diff | PASS: 381 files; 17 residual lines = the 15 documented residuals plus exactly 2 expected lines; 0 unexpected; 0 expected-but-absent |

The two extra residual lines are the old and new rendering of the handler condition. I pre-registered the allowed lines before the run (the intended rendering plus one decompiler-inversion variant); the observed rendering matched the intended one. The re-decompiled handler shows the new condition and the unchanged `if`/`else` structure around it [V, read from the export].

Independent byte-level check (own tag parser, in memory): 2,364 tags in each of original, Stage 2 and the earlier 18-class no-op build; **exactly one tag differs from the original (`DoABC2`, 305,236 → 327,892 bytes)** and exactly one differs from the no-op build (`DoABC2`, 327,878 → 327,892; +14 bytes). All other tags are byte-identical.

Protected-file check: 19 hashed items (original SWF and copy, FFDec jar, `patches\order.txt` and `hooks_order.txt`, harness scripts and baselines, Stage 1 patch and manifest, earlier no-op SWFs, backup manifest file, baseline `Level.as` and `Nights.as`) were identical before and after. The real save was not hashed or opened.

## 6. Limitations

- **Runtime unverified.** No runtime test was run. The change only becomes observable after Stage 3 (while the final-night literal is 30, nights above 30 are unreachable); runtime acceptance is reserved for Stage 5 (first-frame `nightOver` on Continue, no cutscene or pause on nights 30–55, nights 5–25 unchanged).
- **B5 Lite is a deferral, not a per-file round-trip pass.** `Level.as` was verified only through this build's own gates and the earlier aggregate 18-class evidence.
- FFDec's AS3 compiler is flagged experimental; compile success plus source-level equivalence modulo documented residuals is not proof of identical behavior.
- The Stage 1 patched `Nights.as` was not part of this build and is still uncompiled.
- Minor harness note: the build manifest's patch-directory field records a PowerShell object rather than plain text. It does not affect the gates.

## 7. Not done

No Stage 3 work: final-night constant, finale row, `waveAmount`, night names, HUD, menus, medals and save logic are untouched. No SAVE folder, generator or launcher. No Ruffle or game launch.
