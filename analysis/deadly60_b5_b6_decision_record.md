# Deadly 60 — B5 / B6 decision record (documentation only)

Date: 2026-10-17. Branch: `stage-1/b5-b6-decisions`, based on `stage-1/prerequisites-b5-b6` (70bcdfc). Not merged into `main`.

Records the owner's decisions on `analysis/deadly60_prereq_b5_b6_proposal.md`. **No test, build, FFDec/Java run, Ruffle launch, script, `SAVE` folder or save access was performed for this record.** It contains no game source, SWF, decompiled code, save data or lab artifact.

## 1. B5 — Lite option accepted as an evidence-based deferral (not a test pass)

**Decision:** accept the existing aggregate no-op evidence as the current baseline, with the limitations below. The per-file round-trips for `Level`, `HUD`, `MainTimeline` and `Medals` are **deferred, not performed and not passed**.

What the baseline consists of (from the proposal's read-only audit; none of it was re-run):

- The 18-class empty-patch builds `S0a_A` and `S0a_B` (the set includes all four targets) produce byte-identical SWFs (SHA-256 `0B24B469…B699`).
- Tag level: only the code tag `DoABC2` differs from the original; every other tag is identical.
- Decompiled-source level: 15 documented residual lines after normalization (`Level.as` 12, `Controller.as` 2, `MainTimeline.as` 1); `HUD.as` and `Medals.as` have none.
- One earlier observation of an 18-class build reaching the menu.

**Limitations (explicit):**

1. This aggregate evidence is **not equivalent** to four separately isolated per-file round-trips. It cannot show which single class would fail if one did.
2. No individual per-file round-trip is claimed to have passed.
3. Equality of the code tag is not byte-for-byte (it is recompiled); the source-level evidence is decompiled-text equivalence, not bytecode equality. The captured FFDec help shows no bytecode-comparison option.
4. Runtime equivalence per class is not established.
5. The Stage 1 patched `Nights.as` has **not** been compiled; its compile cleanliness stays unverified.
6. FFDec 26.3.0 is documented, not re-confirmed by running it now.

**Not done (as instructed):** no Java or FFDec run; no B5 driver script; no per-class builds or exports; no compile of the patched `Nights.as`; baseline source, SWFs and lab artifacts untouched.

**Consequence for later stages:** a stage that edits `Level.as`, `HUD.as`, `MainTimeline.as` or `Medals.as` still has to pass the build-time gates (tag gate, source-diff gate with the stage's allowed hunks, manifest, protected files) on its own build before any result is trusted. If any of them fails, work on that file stops and is reported.

## 2. B6 — Option P1 approved as a proposal only

**Decision:** the proposed future strategy is **fully synthetic save files generated from scratch** (P1): no user data is read, and the backup copy of the real save is not used as a source. This is approval of the plan only.

- The real-save **metadata check stays disabled.** The real save is not opened, read, hashed, enumerated, copied, modified or overwritten, and its metadata is not checked.
- No `d60_rollout\SAVE\` folder exists or is created. No generator, launcher or test script exists or is created. No save test is run.
- **Still unverified:** the synthetic shared-object byte format and whether the original build loads a generated file. The first live case (SV0) is the earliest point at which that can be learned.
- The proposed isolation design (explicit `--storage disk` with a save directory inside the session folder, path allowlists, recorded evidence) is a **plan**. It must not be described as proven until it has been tested and reviewed. The stated limit remains: absence of a read cannot be proven absolutely without operating-system file auditing, which is not proposed.
- The existing harness protected-files gate hashes the real save and must not be reused as-is.

## 3. Memory gate

The 2.00 GiB free-memory minimum before any Ruffle (and, by the earlier FFDec practice, any Java) run remains in force. Free memory was about 1.39 GiB earlier in this review and 1.04 GiB when this record was prepared. The gate is **not met**. It is not bypassed. No Ruffle launch and no runtime validation were attempted.

## 4. Current state

| Item | State |
|---|---|
| Stage 0 record | committed and pushed (`stage-0/design-verification`, 90bd037) |
| Stage 1 data patch | prepared in the lab and statically verified; not built; compile unverified (`stage-1/append-night-data`, d11894b) |
| B5 | Lite accepted as deferral; per-file proof not performed |
| B6 | P1 approved for planning only; nothing created or run |
| Real-save metadata check | off |
| Tests or runtime operations executed | none |
| Stage 2 (`stage-2/cutscene-option-y`) | **not authorized, not started, branch not created** |

## 5. Still required before any further work

- Separate explicit authorization for Stage 2.
- Separate approvals for any later creation of the `SAVE` folder or scripts, any Java/FFDec run, and any Ruffle run (with the memory gate met).
- Patches, SWFs, decompiled sources, real saves and lab artifacts stay out of Git.
