# Lesson format

> A lesson is the main unit this skill produces. See
> [SKILL.md](./SKILL.md) for the entry point, and
> [WORKSPACE.md](./WORKSPACE.md) for where it lives.

Each lesson is one self-contained HTML file at
`lessons/000N-<dash-case-name>.html`, where `N` increments across the
whole workspace. Open it for the user with a CLI command after writing
it, if possible.

## Design

- One lesson teaches one tightly scoped thing tied to `MISSION.md`.
  Working memory is small — do not stack three concepts in one lesson.
- Calibrate to the user's zone of proximal development: read
  `learning-records/` and `BACKGROUND.md` first, and teach the most
  relevant next thing, not the next thing in some external syllabus.
- Clean, readable typography. The user will reopen this lesson later, so
  it needs to hold up as a reference, not just as a first read.
- Build lessons from `assets/` components: read that directory first,
  reuse what exists. The first component every workspace earns is a
  shared stylesheet, so every lesson looks like one course, not a pile
  of one-offs. When a lesson needs something new and reusable, write it
  into `assets/` and link it, instead of inlining something a later
  lesson would duplicate.
- Link to other lessons and to `reference/` docs via HTML anchors.

## Required sections

Every lesson contains, in this order:

1. **The concept**, scoped to one tight win, calibrated per Design above.
2. **Unfamiliar terms** — see below. Required in every lesson, even when
   the answer is "none this time."
3. **Primary source** — the single highest-trust source this lesson
   draws from, pulled from `RESOURCES.md`. Add it to `RESOURCES.md`
   first if it is not there yet. Never write a lesson with no citation.
4. **Mode 2 only: Build guide** — see
   [BUILD-TRACK.md](./BUILD-TRACK.md#build-guide-section).
5. **Ask me anything** — a short reminder that the agent is the user's
   teacher and follow-up questions belong in chat, not just in the next
   lesson.

## Unfamiliar terms section

This section is required in every lesson and is distinct from
`GLOSSARY.md`. `GLOSSARY.md` is the canonical record of terms the user
has already mastered. This section is the opposite: it front-loads the
terms a lesson is about to use that the user has not mastered yet.

Before writing it:

1. Read `BACKGROUND.md`. Skip any term the user already listed as known.
2. Read `GLOSSARY.md`. Skip any term already recorded there — link to it
   instead of re-explaining it.
3. For every remaining term this lesson uses that a reader with the
   user's stated background would not already know, add one entry:

```md
## Unfamiliar terms in this lesson

**{Term}** — {One or two sentences, in plain language, bridged to
something in the user's known background when one exists, e.g. "like
{concept} in {stack the user knows}, but ..."}
```

If a lesson genuinely introduces no new terms for this user, write the
heading with one line: "No new terms this lesson." Do not omit the
section — an agent skipping it because "there's nothing this time" is
indistinguishable from one that forgot to check.

When the user later demonstrates mastery of a term that passed through
this section, promote it into `GLOSSARY.md` per that file's rules, and
stop repeating it here.

## Skills-based lessons

For lessons teaching a skill rather than a fact (quizzes, guided
real-world steps), build in a feedback loop, as tight as possible,
ideally automatic:

- Interactive quizzes or light in-browser tasks.
- A guided list of real-world steps with a way to check completion.

For a quiz, make every answer option the same word count and, where
possible, the same character count. Formatting must not hint at the
right answer.

## Writing rules

- Prose sections (the concept, unfamiliar terms, primary source note,
  the closing reminder) go through the `ste-writing` skill, STE-flavored
  mode. This does not apply to code blocks or to quiz answer text — see
  [SKILL.md](./SKILL.md#prerequisites).
- Every claim of fact traces to `RESOURCES.md`. If the lesson needs a
  claim with no source behind it yet, find the source first, or mark it
  explicitly as the agent's own understanding, not a cited fact.
- Mode 2: keep the build-guide section to the limit in
  [BUILD-TRACK.md](./BUILD-TRACK.md#build-guide-section) — a guide, not
  a solution.

## Reference documents

Pull the compressed, revisit-worthy essence of a lesson into
`reference/<slug>.html` when the topic warrants it: syntax tables,
algorithms, glossaries, checklists. Lessons are read once or twice;
reference docs get reopened. Link the lesson to its reference doc and
back.
