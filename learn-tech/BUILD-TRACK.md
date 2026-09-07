# Build track — mode 2 only

> Mode 2 is mode 1 plus an upfront brainstorming pass and a build guide
> inside each lesson. See [SKILL.md](./SKILL.md) for the entry point.

Presence of `TEACHING-SPEC.md` in the workspace is what marks it as mode
2. There is no other state to track.

## 1. Brainstorm the stages

**REQUIRED SUB-SKILL:** invoke `superpowers:brainstorming` right after
`MISSION.md` is written and before any lesson exists. The goal of this
pass is not a single project plan — it is a sequence of **learning
stages**, each one a competency level the user climbs through, with a
real build task attached to each.

Example shape (topic: Kubernetes, mission: run the team's staging
cluster):

- Stage 1: core objects and `kubectl` — deploy one pod by hand.
- Stage 2: workloads and services — turn it into a Deployment behind a
  Service.
- Stage 3: config and secrets — externalize the app's configuration.
- Stage 4: operating the cluster — add a health check and a rollout.

Stop the brainstorming pass once stages are scoped and the user has
reviewed them. Do not let it drift into designing the lessons
themselves — that happens per stage, later, in `plans/`.

If the user gave a time budget at intake, use it to size the stage
count, not the depth of any one stage: a tight budget means fewer
stages covering only the mission's core path, not the same stages
compressed. Cut scope into `## Out of scope` rather than rushing a
stage.

## 2. Write and lock `TEACHING-SPEC.md`

```md
# Teaching Spec: {Topic}

**Mission:** {one line, cross-referencing MISSION.md}
**Build:** {what the user is building overall, in one or two sentences}

## Stages

### Stage 1: {title}
{1-2 sentences: the competency this stage builds, and the build task
that proves it.}

### Stage 2: {title}
...

## Out of scope
- {Explicitly excluded, so scope requests later have a clear answer}

Created: {ISO date} — locked.
```

Present it to the user for approval. Loop on edits until approved — this
is the only point at which the file may still change freely.

**Lock moment:** the instant the user gives an explicit affirmative
reply to the presented spec ("looks good", "approved", "yes", or
equivalent). Writing the file is not the same as locking it — if the
turn ends before that reply arrives, add one line at the bottom instead
of a lock date: "draft, not yet approved." Treat it as still open on
the next turn, and ask for approval again before writing any lesson
against it. After a real approval:

- No new stages may be added.
- No stage's build task may be widened.
- The only three allowed amendments are:
  1. `## Re-scope at stage N` — appended only when the user says the
     current stage is clearly wrong-sized (too easy or too hard) for two
     lessons running. Splits or merges only the *not-yet-started*
     remainder, the same way ship-to-learn re-plans a phase.
  2. `## Extension` — appended every time the user finishes every stage
     and wants to keep going on the same build target. This adds
     stages, it does not reopen old ones.
  3. `## Pivot at stage N` — appended when the build target itself
     changes. See Pivot below for the trigger and what it does to the
     stage in progress.
- Any other change request: refuse, explain the lock, offer a new
  workspace for a different scope.

## 3. Per-stage plan

Before the first lesson of a stage, write `plans/stage-N-<slug>.md`:

```md
# Stage N: {title}

**Build task:** {what gets built this stage, concretely — files,
components, or steps}

## Lessons
- [ ] Lesson: {concept 1} — teaches {X}, builds toward {build task}
- [ ] Lesson: {concept 2} — teaches {Y}
- [ ] Build checkpoint: {the concrete deliverable for this stage}
```

Rules:

- 3-6 lessons per stage is a reasonable range. A stage needing more
  probably hides a second stage — split it before writing the plan.
- Check off a lesson's box the moment its HTML file exists in
  `lessons/`. Check off the build checkpoint once the user confirms the
  deliverable works. This is the only tracking mechanism — there is no
  separate progress file.
- When every box in the current stage's plan is checked, write the next
  stage's plan and continue. When the last stage's build checkpoint is
  checked, tell the user the workspace is done, and point at
  `GLOSSARY.md` and `learning-records/` as the compressed take-away. If
  the user then says they want to keep going, do not assume which kind
  — ask the Extension-or-Pivot question in Pivot below before writing
  either amendment.

## Pivot — changing the build target

A pivot changes what the workspace builds, without reopening stages
already finished. Mode 2's one guarantee holds through a pivot: the
workspace still ends with one real, finished build. A pivot changes the
target, never that guarantee.

**Trigger:**

- Mid-project, before every stage is finished: only an explicit request
  fires a pivot — "I want to build X instead," "let's pivot to Y." Do
  not infer a pivot from the user drifting off-topic in chat. Ask them
  to say it plainly first.
- At the end of the last stage, once the final build checkpoint is
  checked: wanting to keep going does not by itself say which kind.
  Before writing either amendment, ask the user directly — keep
  building the same thing (Extension), or take a different approach
  (Pivot)?

**On a pivot:**

`N` is the stage in progress when the pivot fires. If the pivot fires
after the last stage's build checkpoint is already checked, `N` is that
finished stage's number.

1. Confirm the new build target with the user in one line before
   writing anything.
2. Append to `TEACHING-SPEC.md`:

   ```md
   ## Pivot at stage N
   {ISO date}. Old build target: {one line}. New build target: {one
   line}. Why: {one line, in the user's words}.
   ```

3. Any stage listed under `## Stages` after stage N that has no plan
   file yet is superseded. Leave its entry in `TEACHING-SPEC.md` as a
   record — do not delete it — but do not open a plan file for it.
   Every stage from here on follows the new build target, numbered
   continuing the sequence (`N+1`, `N+2`, ...).
4. If stage N's own plan file, `plans/stage-N-<slug>.md`, has unchecked
   boxes when the pivot fires, add one line under its `## Lessons`
   heading: "Interrupted by pivot — see `## Pivot at stage N` in
   TEACHING-SPEC.md." Leave the checked boxes as they are.
5. Write the next stage's plan, `plans/stage-<N+1>-<slug>.md`, as a
   continuation of stage N, not a disconnected restart. State in the
   plan what carries over from stage N's `build/` output and what
   changes.
6. Leave `build/` as it is. Nothing gets deleted or archived — the new
   stage's plan is the record of what changed and why.

Extension and Pivot may each be appended more than once, with no cap
and no warning threshold, for the life of a workspace. Repeated pivots
are a normal part of learning a topic, not a problem to flag.

## Build guide section

Inside every mode-2 lesson (see
[LESSON-FORMAT.md](./LESSON-FORMAT.md#required-sections)), add:

```md
## Build guide

{The concrete next step toward this stage's build task, tied to the
concept just taught. One citation from RESOURCES.md for any
stack-specific syntax or command used.}
```

Limit, same spirit as ship-to-learn's coach mode, lighter-weight:

- A guide, not a solution. Describe the step, name the files or commands
  involved, cite the doc for the idiom — do not hand over full working
  code for the user's part of the task.
- A short skeleton (a function signature, a config key with no value, a
  file with a comment marking where the user's part goes) is fine. A
  complete, ready-to-run implementation of the step is not.
- If the user pastes their attempt and asks "is this right," critique by
  line reference. Do not paste back a corrected version — point at the
  line, name the concept, cite the doc.
- If the user is stuck and asks directly for the code, explain the
  concept a second way or point at a different doc first. Only write the
  working code if the user still insists after that — and say plainly
  that this step was written for them, not by them, so the learning
  record reflects it accurately.
