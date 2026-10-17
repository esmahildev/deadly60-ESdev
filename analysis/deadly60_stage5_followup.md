# Deadly 60 — Stage 5 follow-up: the `#1009` errors and routes to test later nights

Date: 2026-10-17. Branch: `stage-5/followup`, created from `stage-5/runtime-acceptance` at 89200e3c810973d9d9d21014f2ec0683f7c6e933 (clean tree; remote head of the Stage 5 branch identical; no earlier history rewritten). Not merged into `main`. This is documentation only: no game code, patch, build or save was touched. SWFs, logs, screenshots and lab artifacts stay in the lab and are not in Git.

Note on the request: the Stage 5 commit hash quoted in the follow-up request (`89200e3c810973d9d21014f2ec0683f7c6e933`) has 38 characters and is not a valid hash; the actual commit is the 40-character value above.

Labels: **[V]** observed in a run or read from source, **[I]** inferred, **[H]** unverified hypothesis.

## 1. The intermittent `#1009` error is explained, and it is not a candidate regression

The Stage 5 record reported 20 `#1009` errors at the menu in 1 of 5 candidate runs and 0 of 5 original/no-op runs, cause undetermined. That comparison was **confounded by input timing**. The earlier record is not rewritten; this section supersedes its finding F1.

**Evidence from the existing runs [V].** In the one run with errors, the intro played out by itself (menu 13.0 s after start) and the Space key reached the menu 15 s after start; the first error came 0.11 s after the key press. In all nine other earlier runs Space skipped the intro (menu about 5 s after start). So the two groups were never in the same condition.

**Mechanism from the unmodified source [V, read from `Intro.as` and `Transition.as`].** The intro registers a Space/Enter/Esc key listener on the stage and removes it only when a key is pressed. When the intro ends by itself it calls its skip routine through its own frame script, which starts a screen transition whose callback stops the intro tune and sets the tune variable to null, but the key listener stays attached. A later Space press at the menu calls the skip routine again; its callback now calls `stop()` on the null tune variable and throws `#1009`. The transition component runs the callback at frame 10 and clears it only after it returns normally, so the failed callback is invoked again on every loop of the transition clip (19 frames at 30 fps = 0.633 s) until another transition replaces it. This matches the observed stack (`Transition/frame10` → `Transition/changeState` → an anonymous function), the cadence and the end of the errors at the next menu action. Neither `Intro.as` nor `Transition.as` is among the 18 recompiled classes or any patched file.

**Controlled test [V].** New runs, same launch conditions as before, one window at a time, memory at launch 3.10–3.26 GiB, graceful close on the first request every time, empty stderr. Condition LATE_SPACE: wait until the menu has appeared by itself, then press Space. Condition NO_SPACE: wait for the menu and press nothing.

| Condition | SWF | Runs | Runs with errors | Errors per run | Mean gap between errors |
|---|---|---|---|---|---|
| LATE_SPACE | original | 3 | **3** | 26, 26, 26 | 0.633 s |
| LATE_SPACE | candidate (Stage 4) | 3 | **3** | 26, 26, 26 | 0.633 s |
| NO_SPACE | original | 1 | 0 | 0 | none |
| NO_SPACE | candidate | 1 | 0 | 0 | none |

Together with the earlier runs: 0 errors in every run where Space arrived while the intro was still playing (9 runs), 0 in both no-Space runs, and 26 identical errors in each of the six new late-Space runs on both SWFs. The earlier full-flow candidate run was a seventh late-Space case: its 20 errors stopped when the New Game click started a new transition, which replaced the failing callback. Error counts, cadence and stack shape are identical for the original and the candidate.

**Conclusion.** The `#1009` loop is a **pre-existing bug of the original game** (the intro's key listener is never removed when the intro ends naturally), triggered by pressing Space/Enter/Esc at the menu after the intro played out. It is reproduced identically by the original SWF and by the candidate, so it is not caused by Stages 1–4, and nothing was patched to hide it. Limits: observed in Ruffle only; whether Adobe's player logs the same error is unverified [H]; the effect on a human player (a silent per-loop error while the menu stays usable) was not exercised beyond the menu dwell.

## 2. Routes to test the later nights (comparison only; nothing implemented or run)

Targets: nights 5, 25, 30, 31, 35, 55 and 60, plus Continue/load behavior.

Ruffle runs with in-memory storage, so game progress and checkpoints exist only inside one process. A day is 5,850 frames (195 s) unless the player sleeps in the safe room to skip it [V from source].

| | 1. User-assisted manual progression | 2. Disposable test-hook build | 3. B6 synthetic saves (P1, from scratch) |
|---|---|---|---|
| Tests the unmodified candidate | yes | **no** (a modified, hooked copy) | yes |
| Touches saves | no (checkpoints stay in the process memory) | no | no real save; synthetic files in a new disposable folder |
| Game code changes | none | yes (test-only patch, separate flavor, test-marker gates) | none |
| Reaches nights 5, 25, 30, 31, 35 | yes, by playing; the user plays ~5 nights in an estimated 15–30 min, ~30 nights in an estimated 1.5–2.5 h [H] | quickly, per build | quickly, per file |
| Reaches nights 55 and 60 | only after hours of play (est. 2–4+ h in one continuous session) [H] | quickly | quickly (the finale itself still takes ≥ 5 min of play plus boarding) |
| Continue / load path | yes, within one session (death → menu → Continue from the in-memory checkpoint) [I] | only if the hook feeds a built-in test string through the real loader | yes, the real disk path |
| Original-save compatibility | no | no | yes (original-build cases SV0–SV6) |
| Effort and risk | no setup; long human time; a mistake or a crash loses all progress | new patches, one build per target (about 25 s each plus gates); risk that a hook masks a defect; separate acceptance of a modified binary | new generator and launcher with static audits; main risk is that the hand-built file format is rejected (unverified) |
| Approval needed | none beyond a go-ahead | design approval for the hook build | approval of the `SAVE` folder, generator and launcher (the proposal was approved only as a plan) |

**Recommendation [P].**
1. Primary: **option 3** (B6 P1). It tests the exact candidate, covers Continue/load and original-save compatibility, and reaches every target night in seconds. Approve the setup step by step: `SAVE` folder, generator, static audit, preflight, then SV0 on the original build first (this also shows whether the hand-built file format loads at all).
2. In parallel, optional and free of risk: **option 1 for the early nights only**. A person playing the candidate in Ruffle to night 5 (about 15–30 minutes) would give the first observation of a real night, of the night 5 cutscene and of a night-end flow without any new tooling. It cannot reach the later nights in reasonable time.
3. Fallback: **option 2** only if option 3's file format cannot be made to load. It needs its own design review, because it tests a different binary than the candidate.
4. The finale (R6) needs a person (or a long assisted run) in any case: a synthetic dawn-59 save only gets the game to night 60; the countdown, the helicopter and boarding must still be played.

## 3. Status of behaviors (unchanged by this follow-up unless noted)

| Behavior | Status |
|---|---|
| Fresh launch, intro, menu, new game, level creation, game over, return to menu on the candidate | **Verified** in Ruffle (trace identical to the original) |
| Menu `#1009` loop | **Explained**: original-game bug, reproduced identically on the original; not a regression |
| Cutscenes for nights 5–25 (replay on load) | **Untested** |
| Wave queue refill, tail overshoot, room-transition requeue | **Untested / blocked** |
| Option Y cutscene bound after night 25 | **Blocked** (unreachable without a save fixture, a test build or long play) |
| Night 30 → 31, night 60 finale (timer, helicopter, boarding, cutscene 6, menu text, `nightsLasted`) | **Not observed** |
| Medal 18 text on the medals screen | **Untested** |
| Save / Continue compatibility, original saves, dawn-30+ checkpoints | **Untested** |
| Original Flash player / EXE wrapper | **Not tested** (none installed; the wrapper could touch the real save) |

Static validation of Stages 1–4 is unchanged (B5 Lite remains a deferral). **Deadly 60 is not runtime-verified.**

## 4. Recommended next step and approval needed

Approve option 3 in stages, or tell me to wait for a manual playtest of the early nights (option 1), or choose option 2. No save, hook, launcher or test file will be created or run without that approval. Free memory of at least 2.00 GiB stays a precondition for every Ruffle run.
