# Deadly 60 — Post-Stage 1 prerequisites: B5 (FFDec round-trip) and B6 (synthetic save tests)

Date: 2026-10-17. Branch: `stage-1/prerequisites-b5-b6`, based on `stage-1/append-night-data` (d11894b). Not merged into `main`.

**Status: audit and proposals only. Nothing in this document has been executed.** No FFDec, Java, Ruffle, game, build, test or script was run. No `SAVE` directory or generator exists. No real save was read, listed, copied or hashed during this work. No baseline, patched or lab file was modified. This record contains no decompiled code and no game files.

Labels: **[V]** verified now by read-only inspection or hashing of lab files, **[D]** taken from an earlier committed or lab record and not re-run now, **[I]** inferred, **[P]** proposal, **[H]** unverified runtime hypothesis.

---

## Part A — B5: FFDec no-op round-trip

### A1. What exists (read-only inspection)

| Item | Finding |
|---|---|
| FFDec location | `D:\Projects\deadly30_lab\tools\ffdec\` (`ffdec.jar`, `ffdec.bat`, `lib\`); download archive `tools\ffdec_download\ffdec_26.3.0.zip` [V] |
| `ffdec.jar` SHA-256 | `1587E60B2EC2C0E4F5721FD5F2AAAF2379F6D416569DE99054B5C5D1E9BA841C`, equal to the hash pinned in the harness [V] |
| Archive SHA-256 | `35F4930EB7C380AFE66F2117F90B006DEAC0631473AD7500BB39C78F68645ECD` [V] |
| FFDec version | 26.3.0 [D: Stage 3 report, harness pin]. Not re-confirmed by running the tool. The jar manifest only shows it was built with a Temurin 1.8.0_504 JDK. |
| Runtime Java | Java 1.8.0_503 [D]. A `java.exe` launcher stub exists on PATH [V]; it was not executed. |
| Original SWF | `backup\Deadly30_game.ORIGINAL.swf` and `game\spike_0b\E1\input.swf`, both 19,250,808 bytes, SHA-256 `3B9ABEE6E4E760522D43CC2BD4CB5D3C5F54E44CDC22C397B0F5313C0AC3FC9C` [V] |
| Baseline sources of the four targets | match the 381-entry `src_baseline.sha256` manifest [V]: `Level.as` 123,230 B `BB2CE71F…E272`; `HUD.as` 20,247 B `CB8466FE…03A8`; `Deadly30_fla\MainTimeline.as` 7,322 B `2082ED9E…858F`; `Medals.as` 5,914 B `C58D10EA…5A39` |
| Existing harness | `game\harness\` (`build.ps1`, `gates.ps1`, `run_gates.ps1`, `lib\common.ps1`, `config\recompile_stage0a.txt`) [V]. The recompile list holds 18 classes, **including all four targets** [V]. |
| Prior no-op builds | `builds\S0a_A` and `S0a_B` (empty patch set, 18 classes) both produce `deadly30.swf` SHA-256 `0B24B469…B699`, 19,273,450 bytes: two builds, identical bytes [V] |
| Expected residuals | `harness\baseline\expected_residuals_stage3.txt`: 15 normalized lines after the harness removes compiler wrappers: `Level.as` 12 (all inside one `DisplacementMapFilter` builder, not near the planned edit sites), `Controller.as` 2, `MainTimeline.as` 1 (a metadata attribute line). **`HUD.as` and `Medals.as` have none.** [V] |
| Earlier evidence | Stage 3 report [D]: 18-class rebuild differs from the original in exactly one SWF tag, `DoABC2`; the other 24,031 tag lines are identical. C5 check [D, lab `verify\`]: `Nights.as` alone, 2364 tags aligned, one differing tag (`DoABC2`). A harness 18-class build reached the menu with empty stderr in one run [D: committed validation-prep record]. |

### A2. Assessment

- The four targets are **already covered by the aggregate no-op evidence**: tag-level (every non-code tag identical, reproducible) and decompiled-source level (only the 15 documented residual lines). That is the same recompile set later stages will use.
- What does **not** exist is a **separate per-file** run (one class replaced at a time) for `Level`, `HUD`, `MainTimeline` and `Medals`. The 16-class per-class runs described in the Stage 3 report do not name which classes were run, so I do not count them [D, unclear].
- Evidence strength, clearly separated:
  - **Byte-for-byte:** all non-`DoABC2` tags; and run-to-run reproducibility of the 18-class output. The code tag cannot be byte-identical (it is recompiled; 305,230 → 307,149 bytes for `Nights.as` alone).
  - **Decompiled-source equivalence (not byte equality):** re-export compared with the baseline after normalization, accepting only the documented residuals.
  - **Not available in the captured FFDec help:** a bytecode (p-code) comparison option [V: no such option found in the saved help text].
  - **Runtime:** not established per class; one menu-reach observation only.
- Stage builds recompile the whole 18-class set together, so the closest proof of what later stages do is the existing 18-class evidence. A per-file run adds fault isolation (which class would fail), not extra assurance about the combined build.

### A3. Why the round-trip was **not** performed

The authorization for this task excludes Java-based processes, and FFDec is a Java program. Running it would exceed the approved scope. Also:
- Free memory now is about 1.39 GiB, below the 2.00 GiB gate used for earlier FFDec/Ruffle runs [V].
- The existing harness cannot be used unmodified under the current rules: `gates.ps1` (protected-files gate) **hashes the real save file**, and `build.ps1` runs `java -version` for its manifest. Both conflict with the current restrictions [V: code reading].

### A4. Proposed procedure (plan only) [P]

Order: `Level` → `HUD` → `Deadly30_fla.MainTimeline` → `Medals`. Stop at the first failure or unexpected change; do not continue to the next file.

New lab-only location (does not exist yet): `D:\Projects\deadly30_lab\game\d60_rollout\b5\<RunId>\<Class>\` (never an existing folder; the driver refuses if it exists). A driver script would be created in `d60_rollout\b5\` (the existing harness is not modified).

Per file:
1. **Preflight (no Java):** hashes equal the values in A1 (original SWF, FFDec jar, target baseline source); run folder absent; free memory ≥ 2,097,152 KB (recheck up to 3 times, 20 s apart, then stop); no `java` or `ruffle` process.
2. **Copy** `input.swf` to the run folder and re-check its hash.
3. **Build (one Java run):** `java -Xmx3g -jar ...\ffdec.jar -replace <copy> <out.swf> <class> <baseline source>`; the exact command is shown before it runs.
4. **Gate 1 (tags):** every tag except `DoABC2` byte-identical to the original (own tag parser, as in the C5 check); exactly one differing tag, `DoABC2`.
5. **Gate 2 (source):** re-export all scripts with FFDec (second Java run) and compare with the baseline using the harness normalization; allowed residuals only: `Level` 12, `MainTimeline` 1 (+ `Controller` 2 only if present), `HUD`/`Medals` 0. Anything extra is a failure.
6. **Record:** output size and SHA-256, `DoABC2` length delta, residual list, exit codes, seconds, memory readings, protected-set hashes before/after (**real-save hashing excluded**).
7. Side effect to expect: FFDec rewrites its own user configuration file outside the lab on each run (seen in the C5 check). It is recorded, not restored.

Optional extra [P]: one more run compiling the **Stage 1 patched `Nights.as`** alone (the Stage 1 record lists compile cleanliness as unverified), under the same gates, with the one expected extra difference (the appended data).

Cost: up to 8 Java runs (4 builds, 4 exports), roughly 6–40 s each (documented timings). **Lite alternative:** accept the existing 18-class evidence for `HUD` and `Medals` (zero residuals) and review the `Level`/`MainTimeline` residuals as unrelated to the planned edit sites; no new Java run.

Isolation and rollback: inputs are copies with hash checks; outputs go only to new unique folders; nothing existing is overwritten; protected files (original SWF and backups, `src_baseline`, `builds`, harness files, tools) are hashed before and after. Rollback is to ignore the run folder; any deletion needs separate approval.

---

## Part B — B6: synthetic save-test proposal (plan only; nothing created or run)

### B1. Principle

Tests use only disposable files inside one new lab folder. The real save is never opened, listed, copied, hashed or written. The Ruffle run uses `--storage disk` with an explicit `--save-directory` inside the session folder; Ruffle does not use the Flash Player shared-object folder [D: Stage 3 report; to be confirmed by a log line in a preflight]. The earlier committed plan (`first_validation_preparation.md`, commit 72b640e) already defines the harness style and the original-build cases SV0–SV7; this proposal reuses it and adds the unified-build cases.

### B2. Proposed paths (none exist yet)

```
D:\Projects\deadly30_lab\game\d60_rollout\SAVE\
  ISOLATION.txt                 plain statement of the boundaries below
  inputs\                       synthetic .sol files + inputs_manifest.txt (name, case, SHA-256, size)
  run\                          fixed-path copies of the SWFs under test (hash-checked copies)
  tools\                        gen_synthetic_saves.ps1, run_save_session.ps1, verify_isolation.ps1
  sessions\<Tag>\               one new folder per run: savedir\ (Ruffle storage root), command.txt,
                                stdout.txt, stderr.txt, result.json, hashes_before/after.txt
```
Everything is under `d60_rollout\SAVE\`; nothing is placed in `src_baseline`, `builds`, `harness`, `backup*` or the repository.

### B3. Generator behavior (intended)

- **Preferred source (P1): fully synthetic, no user data.** The generator builds a Flash shared-object file from scratch using the game's own `saveString` layout (sections `|`-separated: 0 = night, scrap, medkits, armor, fence, boots, ammo×3, kills; 1 = soldier skins; 2 = equipped gun ids; 3 = bought weapon ids; 4 = boxes; 5 = turrets) plus the other keys the game reads (`nightsLasted`, `lvl`, `quality`, `resolution`, `achievedMedals`, `achievedGuns`). Layout is from the game source [V earlier]; the container byte format and that the original build accepts the generated file are **unverified** until case SV0 runs.
- **Alternative (P2): patch the existing backup copy** of the real save (as in the older plan). This reads a copy of user data and needs your explicit approval; P1 avoids it.
- Behavior: takes a case name; writes one new file in `inputs\`; refuses to overwrite; validates its own output by re-parsing it (length fields, terminators); records SHA-256 and size in `inputs_manifest.txt`; keeps an audit log of every path it opens; has an allowlist that rejects any path outside `d60_rollout\SAVE\` (and, for P1, any path under the user profile).

### B4. Test cases and expected outcomes (predictions, not results)

On the **original SWF** (baseline questions, no patched build needed):

| Case | Content | Expected [I] |
|---|---|---|
| SV0 | valid checkpoint, night 12, `nightsLasted` ≥ 1 | Continue enabled; loads; trace shows `LOADED STRING`, then `DAY!` and `SAVE STRING =` shortly after (would confirm the first-frame `nightOver` → `addDay()` re-run) |
| SV1 / SV2 | night 29 / night 30 | loads (loader loops rows 0–28 / 0–29); night 30 behavior after load is untested |
| SV3 | night 31 | TypeError at the missing `waveArray[30]` during Continue |
| SV4 | unknown equipped gun id | failure during load (null dereference expected) |
| SV5 | unknown bought id | ignored; loads like SV0 |
| SV6 | a 7th section appended | ignored; loads like SV0 |
| SV8 / SV9 (optional) | night 5 / night 10 checkpoint | cutscene replays on load if not already played this session |

On the **unified build** (only after the relevant stages exist and are separately approved):

| Case | Content | Expected [H] |
|---|---|---|
| UV0 | the SV0 file | loads, same as original |
| UV1 | night 29 | continues into an ordinary night 30 (no finale, no cutscene) |
| UV2 | night 30 | loads; night 31 follows normally |
| UV3 | nights 34, 35, 55, 59 | no freeze; with Option Y, nights 35 and 55 go straight to the day with no pause and no cutscene |
| UV4 | menu key `nightsLasted` = 30 | menu text and Continue behave per the new thresholds |

Minimum first set: SV0, SV3, SV4, SV5, SV6 (as in the committed plan). Each result is reported Passed / Failed / Skipped / Unverified; a prediction that is not confirmed is a finding, not a failure.

### B5. Proving the real save is not touched

1. **By construction:** no script contains the real save path; the generator and launcher allowlists reject it; the launcher refuses to start unless `--storage disk` and a `--save-directory` under `...\SAVE\sessions\` are present.
2. **Recorded evidence:** the exact command line, the Ruffle log line that names its storage folder, a file listing of the session folder before/after, and the generator's path audit log.
3. **Existing harness gate not reused:** the protected-files gate hashes the real save; a new gate set must exclude it.
4. **Optional, default off:** a metadata-only check (size, last-write time, attributes; no content read, no hash) of the real save before and after. This needs your explicit approval because it touches the real file's metadata.
5. **Stated limit:** absence of a read cannot be proven absolutely without operating-system file auditing, which would change system policy and is not proposed. The claim would be "isolated by construction, nothing contradicting it observed".

### B6. Isolation, rollback and cleanup

- One Ruffle window at a time; graceful close only, never force-kill; free memory ≥ 2,097,152 KB (3 rechecks, 20 s apart); stop on any unexpected process or window.
- Writes only under `d60_rollout\SAVE\`. The original SWF, its backups, `src_baseline`, `builds`, harness files, tools and the backup save copy are hashed before and after (real save excluded).
- On any mismatch: stop, report the original output, restore nothing automatically; restoration only from verified backups and only with your approval.
- Cleanup: none automatic. Session folders stay as evidence; deleting them needs separate approval.

### B7. Evidence recorded per session

Tag; SWF path and SHA-256 before/after; input `.sol` SHA-256 and the post-run `.sol` SHA-256; command line; Ruffle binary hash; memory readings; stdout/stderr and the relevant `trace` lines; exit code; window-closed verification; protected-set diff; classification (valid/invalid) and result.

### B8. Review before execution

1. You review and approve this plan (including P1 versus P2).
2. Script creation is a separate approval; each script then gets a static audit (findings listed and fixed) before any run.
3. A preflight-only mode (no Ruffle) is run first, as was done for the earlier C5 driver.
4. First live run is SV0 only; continue only if it is valid.

---

## Approvals required (nothing proceeds without them)

| # | Approval |
|---|---|
| 1 | Run FFDec (Java 8) for B5 and choose the scope: per-file (4 builds + 4 exports; recommended for fault isolation) or lite (no new run). |
| 2 | Create a B5 driver script under `d60_rollout\b5\` (new, lab only) that excludes real-save hashing and `java -version`. |
| 3 | Optional: a compile check of the Stage 1 patched `Nights.as` alone. |
| 4 | Free memory raised to ≥ 2.00 GiB before any Java run (currently about 1.39 GiB). |
| 5 | B6: approve this plan and choose P1 (recommended) or P2; separately approve creating `SAVE\`, the generator and the launcher; keep the real-save metadata check off unless you say otherwise. |
| 6 | Stage 2 itself (branch `stage-2/cutscene-option-y`) is not started and needs its own approval. |
