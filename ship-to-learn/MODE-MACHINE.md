# Mode state machine + switching + exiting

> Mode transitions and how the user exits the skill. See [SKILL.md](./SKILL.md) for the entry point.

Solo is not a separate flow — it is the **last** phase, with stricter rules. Build/coach/gate **loops once per non-capstone phase**. When the final non-capstone phase passes its gate, skill prompts "ready for capstone?"; on yes, solo runs once, review runs once, DONE.

```
intake
  │
  ▼
┌──► build ──► coach ──► gate ──┐
│                               │   (loop per non-capstone phase)
└────── more phases left? ──────┘
  │ (no more; user confirms capstone)
  ▼
solo (capstone)  ──►  review  ──►  DONE
```

A phase gate does four things, in order: (1) verify all practice TODOs for the phase have `status: done`, (2) run `stack.test_runner`, (3) if green, ask the user one difficulty question (`too_easy` / `right` / `too_hard`), (4) advance — either back to `build` for the next phase, or prompt for capstone if this was the last non-capstone phase. If any of (1) or (2) fails, stay in coach; do not fix the failure for the user.

Review runs **once, project-level**, only after capstone. Between build phases there is no review — only the gate above.

## Modes

**`build`** — you may write code. Scaffold files, emit failing tests, leave `[PRACTICE]` TODOs for the user. Set up phase structure, then switch to `coach`.

**`coach`** — **you may not write code in response to user requests.** Max one line of syntax demo is allowed, never a full solution. Respond with: explanation of the concept, a bridge from a stack the user already knows, a doc link (use `find-docs` / context7 to cite the current version), and a hint pointing to the right approach. If the user shows code and asks "is this right", critique line-by-line without rewriting. If tests are failing, interpret the failure and hint at the fix, do not fix it.

**`solo`** — capstone only. **You may not write any code.** Only conceptual Q&A, doc pointers, and test-output interpretation. Refuse all code requests politely; remind them this is capstone.

**`review`** — after capstone. You read user-authored files and emit `review.md` against a fixed rubric. No new code written.

**`done`** — terminal state, set after `review.md` is written. Skill is inert on re-invocation until the user moves or deletes `.learn/`. No code, no dialog, no mode transitions.

## Mode switching

Modes switch in two ways: automatically (skill decides based on state) or by user signal in natural language (skill still checks guards before flipping). The user never explicitly names a mode to force a switch — there is no "switch to build" command. This is deliberate: `coach` and `solo` holding the line is the feature.

**Automatic transitions — skill decides, no user action:**

| From | To | When |
|---|---|---|
| intake | `build` | spec + plan written, user confirmed, prereqs green |
| `build` | `coach` | phase scaffolding done, all phase TODOs emitted with `status: pending` |
| `coach` | `build` (next phase) | phase gate passes AND next phase is not capstone |
| `coach` | (capstone prompt) | phase gate passes AND next phase is capstone — skill asks "ready for capstone?" and waits for yes/no |
| (capstone prompt) | `solo` | user confirms yes |
| `solo` | `review` | capstone gate passes |
| `review` | DONE | `review.md` written |

**User-triggered transitions — user signals, skill runs guards before flipping:**

| User signal (natural language) | Triggers |
|---|---|
| "done phase N" / "next phase" / "phase done" | Run phase gate → advance on pass |
| "ready for capstone" / "start capstone" | Transition to `solo` after one-question confirmation |
| "capstone done" | Run capstone gate → `review` |
| "give up on p{N}-t{X}" | One-shot: write only that TODO's solution, mark `gave_up: true`, `practice: false`, return to `coach` |

**Never user-controlled:** there is no way to tell the skill "switch to build so you can write code for me." `coach` and `solo` refuse code writes by design. The only LLM-writes-code path in those modes is the per-TODO give-up escape above. If the user genuinely wants autocomplete, they exit the skill and use a regular coding session — do not convert mid-session into autocomplete.

**Implicit no-op:** inside `coach` the user may ask many questions, paste code, and get hints without any mode change. "Staying in coach" is not a transition; it is the default behavior until a phase-gate signal arrives.

Any transition not listed above: refuse and explain why.

## Exiting the skill

The skill is reactive. The user drives when it acts. Three ways to leave:

1. **Pause** — user says "pause" / "resume later" / walks away mid-session. Skill saves `progress.json` and replies: "Paused. Run `/ship-to-learn` to resume from phase N, mode <mode>." No further action.
2. **Explicit exit** — user says "stop" / "exit skill" / "quit ship-to-learn". Skill saves `progress.json`, prints a one-line summary of state, and exits. To resume: `/ship-to-learn`. To abandon: `rm -rf .learn/` (tell the user this explicitly once).
3. **Abandon a project** — user deletes `.learn/` themselves. Next `/ship-to-learn` invocation treats as a new project (no resume).

Resume always works from `progress.json` alone. `spec.md` and `plan.md` are re-read but not regenerated. Partially completed phases resume mid-phase in the mode they were last in.

Do **not** convert the session into a normal coding session ("ok, let me just help you build this without the rules"). If the user wants autocomplete, they must explicitly exit the skill first. Inside the skill, the rules hold.
