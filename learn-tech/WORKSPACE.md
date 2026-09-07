# Workspace — directory layout + file formats

> The teaching workspace is the current directory. There is no separate
> state file: the directory contents are the state. See
> [SKILL.md](./SKILL.md) for the entry point.

## Directory layout

```
MISSION.md              # why the user wants this topic — locked after creation
BACKGROUND.md           # what the user already knows — updated freely
RESOURCES.md            # curated high-trust sources — updated freely
GLOSSARY.md             # canonical mastered terms — updated freely, in place
learning-records/
  0001-<slug>.md         # append-only, one per genuine learning event
lessons/
  0001-<slug>.html       # append-only, one per lesson, numbered continuously
reference/
  <slug>.html            # cheat sheets and reference docs pulled out of lessons
assets/
  <shared components>    # stylesheets, quiz widgets, diagram helpers
NOTES.md                 # scratchpad — user preferences, working notes

# mode 2 only — see BUILD-TRACK.md
TEACHING-SPEC.md         # learning stages — locked after brainstorming + approval
plans/
  stage-1-<slug>.md       # lessons + build tasks for that stage, checkbox-tracked
build/                    # the real thing the user is building
```

## Invariants

- `MISSION.md` is **locked** the moment it is written. Revise only when
  the user's goal genuinely changes — confirm with the user first, and
  add a learning record noting the change (see below). Do not amend it
  silently.
- `BACKGROUND.md` is **not** locked. Append or edit it any time the user
  states new prior knowledge.
- `learning-records/` is **append-only**. New files only, never rewritten.
  Numbering is sequential: scan the directory, take the highest number,
  add one.
- `lessons/` is **append-only**, numbered sequentially across the whole
  workspace regardless of stage.
- `GLOSSARY.md` is edited **in place** as understanding deepens — see
  Rules below. This is the one file in the workspace that is expected to
  change after it is first written.
- `TEACHING-SPEC.md` (mode 2) is **locked** the instant the user approves
  it. See [BUILD-TRACK.md](./BUILD-TRACK.md) for the lock moment and the
  three exceptions (Re-scope, Extension, Pivot).
- Every file in this list, except lesson HTML code blocks and quiz
  answer text, is generated with the `ste-writing` skill. See
  [SKILL.md](./SKILL.md#prerequisites).

## `MISSION.md` format

```md
# Mission: {Topic}

## Why
{1-3 sentences. The real-world outcome the user wants. Avoid abstract
framing like "to understand X" — push for what changes in their work or
life once they have this skill.}

## Success looks like
- {A specific, observable thing the user will be able to do}
- {Another specific thing}

## Constraints
- {Time, prior commitments, learning preferences — anything that bounds
  the approach}

## Out of scope
- {Adjacent topics the user does not want to chase right now}
```

Rules:

- One mission per workspace. Two unrelated topics are two directories.
- Concrete beats abstract: "ship a Rust CLI to my team by Q1" beats
  "learn Rust."
- If the user cannot say why, ask before writing anything. A vague
  mission is worse than no mission — it cannot calibrate anything.
- Keep it to one screen. Past that, it has become a plan, not a compass.

## `BACKGROUND.md` format

```md
# Background

## Known well
- {Language, framework, or field} — {how deep: "shipped production code
  for 3 years", "read the docs once", "used it in one class project"}

## Explicitly not known
- {A term or concept the user flagged as unfamiliar, so it is not
  skipped by accident}
```

Rules:

- This file is what [`LESSON-FORMAT.md`](./LESSON-FORMAT.md)'s
  unfamiliar-terms section checks before deciding whether to explain a
  term. Read it before writing that section, every lesson.
- Depth matters more than the list. "Knows Python" and "shipped a Django
  app for four years" calibrate very differently.
- Update it the moment the user states new prior knowledge, even
  mid-lesson. Do not wait for the next session.

## `RESOURCES.md` format

```md
# {Topic} Resources

## Knowledge
- [{Title}]({url}) — {one line: what it covers, when to use it}

## Wisdom (Communities)
- [{Community name}]({url}) — {one line: what it is good for}

## Gaps
- {An area with no good source found yet}
```

Rules:

- High-trust only: primary sources, recognized experts, peer-reviewed or
  official docs, well-moderated communities. Leave out marketing dressed
  as education.
- Annotate every entry with one line. A bare link is useless later.
- Prune a source that turns out wrong or shallow — do not leave it for
  reference.
- If the user has opted out of joining a community, record that here so
  future sessions do not keep proposing one.
- An empty subsection is not an empty heading. Write one line under it,
  for example "No community picked yet" or "No source found for this
  area yet" — the same convention `## Gaps` already uses.

## `GLOSSARY.md` format

```md
# {Topic} Glossary

{One or two sentences on what this glossary covers.}

## Terms

**{Term}**:
{One or two sentences. What the term IS, not what it does.}
_Avoid_: {other names for the same thing, to not use instead}
```

Rules:

- Add a term only once the user demonstrates they understand it, not the
  moment it is introduced. A term the lesson only just used belongs in
  that lesson's unfamiliar-terms section, not here yet — see
  [LESSON-FORMAT.md](./LESSON-FORMAT.md).
- Pick one name per concept and list the rest as aliases to avoid.
- Use the glossary's own terms inside later definitions.
- Group under subheadings once natural clusters appear.
- Revise a stale definition in place. Do not leave it wrong because the
  user has moved on.

## Learning record format

```md
# {Short title of what was learned or established}

{1-3 sentences: what was learned, or what prior knowledge was
established, and why it changes what to teach next.}
```

Write one when:

1. The user shows real understanding of something non-trivial — not
   just exposure to it.
2. The user states prior knowledge **mid-project**, after
   `BACKGROUND.md` already exists — cross-reference into
   `BACKGROUND.md` too.
3. A misconception gets corrected. These predict future stumbling
   blocks on related topics.
4. The mission shifts. Cross-link to `MISSION.md` and update it.

Do not write one for material merely covered, or as a session log. Do
not write one for the background the user states during intake — that
goes straight into `BACKGROUND.md` and nowhere else; a record restating
it the same session is redundant. Learning records are decision-grade,
not a diary.

## `NOTES.md`

Freeform. Record the user's stated teaching preferences here (pace,
format, tone) so future lessons keep applying them without re-asking.
