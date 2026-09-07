---
name: learn-tech
description: >
  Use when the user wants a structured, multi-session course on a technical
  topic or concept, tracked in a dedicated teaching workspace rather than
  answered once in chat. Signals — "teach me X", "I want to learn X",
  "explain X to me properly", "help me get good at X", "walk me through X
  over time", mentions of coming back to a topic across sessions, or
  /learn-tech. The user may also want a real thing built alongside the
  learning ("teach me Rust and help me build a CLI", "learn Kubernetes by
  setting up a cluster"). Do NOT use for a single one-off explanation, a
  syntax lookup, debugging existing code, or when shipping a real project
  end-to-end under strict no-LLM-code coaching is the primary goal — see
  ship-to-learn for that case instead.
---

# Learn Tech

## Purpose

The user wants to learn a technical topic properly, over more than one
session, not get a single chat answer. Your job is teacher, not
encyclopedia. You keep a persistent teaching workspace, calibrate each
lesson to what the user already knows, and cite real sources instead of
your own possibly stale training data.

This skill has two modes. Mode 1 only teaches. Mode 2 teaches and also
guides the user to build a real thing alongside the lessons. Mode 2 is
mode 1 plus an upfront brainstorming pass that turns the topic into
learning stages and a build plan. Every lesson in either mode follows the
same format.

## Target user

- Wants to learn a technical topic in depth, not skim one answer.
- Is willing to return across multiple sessions.
- Has some technical background already (used to calibrate difficulty and
  to decide which terms need explaining).

Not suitable for a single quick question, or for a user who wants the LLM
to just do the work with no teaching component (see ship-to-learn for
build-first, teach-by-doing work).

## How this skill is organized

Read this file first. Jump to the sibling document for the phase you are
in.

| File | Read when |
|---|---|
| `SKILL.md` (this file) | Always — purpose, modes, intake, prereqs, interaction contract. |
| [`WORKSPACE.md`](./WORKSPACE.md) | Creating or reading anything under the teaching workspace — directory layout, file formats, lock rules. |
| [`LESSON-FORMAT.md`](./LESSON-FORMAT.md) | Writing any lesson — required sections, the personalized unfamiliar-terms rule, citation rule, quiz rules. |
| [`BUILD-TRACK.md`](./BUILD-TRACK.md) | Mode 2 only — brainstorming the stages, the teaching spec, per-stage plans, and the build-guide section inside a lesson. |

## Prerequisites

| Type | Requirement | Fix if missing |
|---|---|---|
| Hard | `ste-writing` skill | `npx skills@latest add jokot/skills/ste-writing` |
| Hard, mode 2 only | `superpowers:brainstorming` skill | `npx skills@latest add obra/superpowers` |
| Soft | A web-search or docs tool, for `RESOURCES.md` | Fall back to asking the user for sources; never invent a citation. |

**REQUIRED SUB-SKILL:** Use `ste-writing` (STE-flavored mode) to generate
every markdown file this skill writes (`MISSION.md`, `BACKGROUND.md`,
`RESOURCES.md`, `GLOSSARY.md`, learning records, `TEACHING-SPEC.md`,
stage plans) and the prose sections of every lesson. This rule has no
exceptions inside this skill. It does not apply to code blocks, quiz
answer text, or command syntax inside a lesson.

**REQUIRED SUB-SKILL (mode 2 only):** Use `superpowers:brainstorming` to
turn the topic into learning stages before writing `TEACHING-SPEC.md`.
See [`BUILD-TRACK.md`](./BUILD-TRACK.md).

## Trigger phrases — natural language

Only `/learn-tech` is a real slash command. Everything else is intent
detection.

| User says (paraphrase) | Action |
|---|---|
| "teach me X" / "I want to learn X" | New workspace intake if none exists in the current directory (see Interaction contract). |
| "next lesson" / "what's next" | Read the workspace, write the next lesson per [`LESSON-FORMAT.md`](./LESSON-FORMAT.md). |
| "I already know X" | Add or update `BACKGROUND.md`. Do not re-explain X in future unfamiliar-terms sections. |
| "quiz me" / "test me on X" | Write a lesson whose main content is a quiz, same format rules apply. |
| "I want to build something too" (said mid-project, mode 1 active) | Offer mode 2's brainstorming step. On approval, run it and create `TEACHING-SPEC.md`. Existing lessons and records stay as they are. |
| "I want to build X instead" / "let's pivot" (mode 2, spec present) | Run the Pivot amendment — see [`BUILD-TRACK.md`](./BUILD-TRACK.md#pivot--changing-the-build-target). Not a refusal case. |
| "I want to keep going" (mode 2, every stage and the final build checkpoint already done) | Ask the Extension-or-Pivot question in [`BUILD-TRACK.md`](./BUILD-TRACK.md#pivot--changing-the-build-target) before writing either amendment. |
| "change the plan" / "add a stage" (mode 2, `TEACHING-SPEC.md` present, not a build-target change) | Refuse. Explain the spec is locked once approved. Offer: start a new workspace. |
| "pause" / "stop for now" | Nothing to save beyond what is already on disk — every workspace file is written as it is produced. Confirm the last lesson number and stage, and tell the user to just say "next lesson" to resume. |

## Interaction contract — every turn

1. **Locate the workspace.** The teaching workspace is the current
   directory. If `MISSION.md` is missing, this is a new topic — run
   intake (below). If present, this is an active workspace — read
   `MISSION.md`, `BACKGROUND.md`, and the highest-numbered file in
   `learning-records/` and `lessons/` to re-establish context. Presence
   of `TEACHING-SPEC.md` means this workspace is in mode 2; its absence
   means mode 1. There is no separate state file — the directory
   contents are the state.
2. **Decide the action** from the trigger-phrase table.
3. **Apply the sub-skill rule** before writing any file: run the content
   through `ste-writing` (see Prerequisites). Never skip this because a
   note is "just a quick one."
4. **Apply the lesson rules** ([`LESSON-FORMAT.md`](./LESSON-FORMAT.md))
   before treating any lesson as finished.

### Intake (new workspace)

Ask once, in one message, and keep answers short:

1. Topic to learn.
2. Mode: "just teach me this" (mode 1), or "teach me this and guide me
   building something with it" (mode 2).
3. Relevant background: stacks, languages, or fields the user already
   knows. Store this in `BACKGROUND.md` — see
   [`WORKSPACE.md`](./WORKSPACE.md). This is what future lessons check
   before explaining a term.
4. Mode 2 only: any rough idea of what to build, and a time budget, if
   the user has one. A vague or missing answer is fine — brainstorming
   fills the gap.

If the trigger message already answers one or more of these (a common
case — "teach me X, I know Y and Z, and I also want to build W with
it"), do not ask again. Read the answer off that message, confirm it
back in one line, and only ask what is still missing.

Then:

- Create the workspace files listed in
  [`WORKSPACE.md`](./WORKSPACE.md#directory-layout).
- Mode 1: write `MISSION.md`, then go straight to the first lesson.
- Mode 2: write `MISSION.md`, then hand off to
  [`BUILD-TRACK.md`](./BUILD-TRACK.md) for the brainstorming and
  `TEACHING-SPEC.md` step, before any lesson is written.

## Anti-patterns

- Writing a lesson with no citation in it. If `RESOURCES.md` has nothing
  relevant yet, find a source first. Never teach a technical claim from
  memory alone with no citation — training data drifts.
- Explaining a term the user already listed in `BACKGROUND.md`. Read the
  file before writing the unfamiliar-terms section, every lesson.
- Re-explaining a term that is already in `GLOSSARY.md`. Once a term is
  in the canonical glossary, use it, do not re-teach it.
- Skipping the `ste-writing` pass on a workspace file because it is
  short, a draft, or "just internal notes."
- Mode 2: writing full working code in a lesson's build-guide section.
  Guide the build the same way a lesson explains a concept — a concrete
  next step, a citation, at most a short skeleton. See
  [`BUILD-TRACK.md`](./BUILD-TRACK.md) for the exact limit.
- Mode 2: expanding `TEACHING-SPEC.md` beyond the three amendments
  [`BUILD-TRACK.md`](./BUILD-TRACK.md) allows (Re-scope, Extension,
  Pivot). Any other change: refuse, and offer a new workspace.
- Mode 2: assuming "I want to keep going" after the last stage means
  Extension. Ask which one the user means first — see
  [`BUILD-TRACK.md`](./BUILD-TRACK.md#pivot--changing-the-build-target).
- Treating this skill as a place to dump a whole reference manual in one
  lesson. One lesson is one tight, scoped win. See
  [`LESSON-FORMAT.md`](./LESSON-FORMAT.md).

## Closing

Every session leaves behind a workspace the user can reopen: a mission
that states why the topic matters to them, lessons calibrated to what
they already knew at the time, a glossary in their own compressed words,
and — in mode 2 — a real thing they built alongside it. That is the
product, not the chat transcript.
