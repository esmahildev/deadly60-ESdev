# Deadly 60 — Rollout Stage 5 record (runtime acceptance, partial)

**Result: PARTIAL. Only a fresh-launch smoke flow and a small A/B stability series could be run safely. Night 30 → 31 and the Night 60 finale were NOT observed. The 60-night campaign is NOT runtime-verified.** Save and Continue behavior is untested. The original EXE wrapper was not tested.

Date: 2026-10-17. Branch: `stage-5/runtime-acceptance`, created from `stage-4/text-and-medal-wording` at 05204197ebbe4713c59eb123ddb9faa13fa2e628 (clean tree and remote head verified; Stage 0–4 ancestry intact; no earlier stage-5 branch). Not merged into `main`. SWFs, run logs, screenshots and lab artifacts stay in the lab and are not in Git.

## 1. Artifact and runtime

| Item | Value |
|---|---|
| Candidate | `D:\Projects\deadly30_lab\game\d60_rollout\stage4\builds\S4_TEXT_1\deadly30.swf`, 19,285,387 bytes, SHA-256 `155A5167…304D`; verified before every launch and unchanged after (cumulative Stage 1–4 build) |
| Controls | original `E1\input.swf` (`3B9ABEE6…FC9C`); no-op 18-class rebuild `builds\S0a_A\deadly30.swf` (`0B24B469…B699`) |
| Runtime | Ruffle desktop (the lab's pinned nightly, `ruffle.exe` SHA-256 `A4C96B92…43E7`); the only runtime available |
| Launch | `--graphics dx12 --storage memory --load-behavior blocking --dummy-external-interface --no-gui --width 750 --height 460`, the same flags as the earlier valid original runs; working directory the lab harness folder; one window at a time; `RUST_LOG=info`; stdout/stderr captured per run |
| Input | the lab's existing `inputctl.ps1` (hash-pinned): keys and screenshots go to the Ruffle window by handle; the menu click is a real-cursor click guarded to that window |
| Gates | raw free memory ≥ 2,097,152 KB checked before every launch (all passed, 2.81–3.02 GiB); no live Ruffle/java process before any launch; graceful close only, never killed |

**Isolation basis (stated limit).** The game ran inside Ruffle with the in-memory shared-object backend, so nothing is written to disk by the game, and Ruffle does not use the Flash Player shared-object folder where the real save lives (documented in the Stage 3 feasibility report and consistent with earlier runs whose protected-state checks showed the real save unchanged). This was **not independently proven in this stage**: no real save, save metadata or backup copy was opened, read, hashed, listed or copied, so nothing could be compared. No `SAVE` folder, synthetic-save generator or launcher exists. The original EXE wrapper was not launched and not inspected (it could touch the real save), and no Flash Player or projector exists in the lab.

## 2. Runs performed (evidence under `D:\Projects\deadly30_lab\game\d60_rollout\stage5\runs\`)

| Tag | SWF | Flow | Outcome |
|---|---|---|---|
| `S5_R1_FRESH_1` | candidate | menu → New Game → intro cutscene → skip → level → unattended until the soldier died → game over → credits → menu | flow completed (all seven steps); 35 trace lines **identical to the valid original reference**; 0 stderr; **20 AVM2 `#1009` errors, all during the menu phase** (finding F1); closed gracefully |
| `S5_CTL_O_1` | original | menu only (16 s dwell) | 0 errors; first close request ignored for 60 s, second graceful request worked (finding F2) |
| `S5_CTL_N_1` | no-op rebuild | menu only | 0 errors |
| `S5_CTL_S4_2` | candidate (repeat) | menu only | 0 errors |
| `S5_AB_S4_1..3`, `S5_AB_O_1..3` | candidate / original, interleaved | menu only | 0 errors in all six |

## 3. Results R1–R8

| ID | Probe | Result | Evidence and limits |
|---|---|---|---|
| R1 | First update after a fresh isolated launch; night counter | **PASS (fresh launch only)** | Observed: the key-marker and full 35-line trace sequence equals the original's; no `NIGHT!`, `DAY!`, `SAVE STRING`, `LOADED STRING` or `NEW ENEMY` trace occurred; the in-game HUD screenshot shows "NIGHT 1". The Level constructor ran the full rebuilt `Nights` data function (59 names, 60 rows) with no error. **Not covered:** the first update after a Continue/load (the post-load `addDay` re-run stays an inference) and any later night. |
| R2 | Cutscenes for nights 5–25 | **UNTESTED** | Observed only the intro cutscene (id 0): added and removed correctly on the candidate. Replay-on-load and nights 5–25 need a saved state or a test build, neither authorized. |
| R3 | Queue refill, tail overshoot, room-transition requeue | **BLOCKED / UNTESTED** | The unattended soldier died before any controlled fight; nothing was observed about wave behavior. |
| R4 | Cutscene Option Y after night 25 | **BLOCKED** | Nights above 25 are unreachable without a synthetic save or a test build. |
| R5 | Night 30 → 31 (queue, cap, name, data) | **BLOCKED** | Same reason; **not observed.** |
| R6 | Night 60 finale, helicopter, boarding, cutscene 6, menu text, `nightsLasted` | **BLOCKED** | **Not observed.** |
| R7 | Save / Continue compatibility | **UNTESTED** | No save fixture authorized. Observed only: with an empty in-memory store the menu's Continue button is greyed and "NIGHTS LASTED" is 0, same as the original. No backward-compatibility claim. |
| R8 | Behavior in Ruffle vs a real Flash runtime / EXE wrapper | **BLOCKED (Flash/EXE); Ruffle comparison partial** | Ruffle only: candidate and original produced the same trace and the same menu behavior, except finding F1. No Flash runtime is installed and the EXE wrapper was not run. |

Not observed either: the Stage 4 medal text on the medals screen (UNTESTED), the menu texts at 60, and the HUD clock on night 60.

## 4. Findings

**F1 — intermittent `#1009` errors at the menu (candidate, 1 of 5 runs; original and no-op rebuild 0 of 5).** In `S5_R1_FRESH_1` twenty identical errors ("Cannot access a property or method of a null object reference") were logged every 0.633 s from 2.4 s after the menu appeared until just before the New Game click. The stack was `Transition/frame10` → `Transition/changeState` (the transition component's frame scripts, not a class changed by Stages 1–4). The game kept running and played the flow to the end with the same trace as the original. Four further candidate menu runs (one control, three interleaved) and five original/no-op runs showed none. Free memory at launch was 2.8–3.0 GiB in every run (the lowest value during the full-flow run was 1.89 GiB, in the control menu runs 2.05–2.1 GiB) and load latency was 2.0–2.4 s in the runs where it was measured, so memory pressure is unlikely to explain it, though it is not excluded. Cause **undetermined**: the sample is too small to say whether the rate is higher than for the original; it resembles the `#1009` errors recorded in earlier batches whose controlled re-test never ran. Not a gameplay-visible failure in this run; no fix was attempted. A larger A/B sample would be needed before attributing it to the patches or to Ruffle.

**F2 — close behavior.** In one control run (`S5_CTL_O_1`, original) the first graceful close request was ignored for 60 s and the window closed on the next graceful request. The process was never killed and no stray window remained after any run.

## 5. Environment notes and limitations

- Ruffle only; Ruffle is a re-implementation, so its results do not prove behavior in Adobe's player or in the original EXE wrapper.
- The automated menu click moves the real mouse cursor (guarded to the Ruffle window) and the window is made topmost during a run.
- Side effects outside the run folders: four screenshot PNGs in the harness `runtime_tmp\shots` folder and normal updates to Ruffle's own config folder.
- Static validation (Stages 1–4) is unchanged; B5 Lite remains a deferral, not a per-file round-trip pass.

## 6. What is needed to continue (not done, needs separate approval)

1. A way to reach nights 4–5, 24–25, 29–31, 34–35, 55 and 59–60 in Ruffle: either the approved isolated synthetic saves (B6 P1: new `SAVE` folder, generator, launcher), or a disposable test-hook build, or user-assisted manual play. Without one of these R2–R6 stay unobserved.
2. A larger A/B sample (original vs candidate, menu and early game) to settle F1.
3. A safe way to test a real Flash runtime or the EXE wrapper (it must not be able to touch the real save), if R8 is wanted.
