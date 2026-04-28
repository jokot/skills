# Composing with `superpowers` skills

> Sub-skill delegation rules for ship-to-learn. See [SKILL.md](./SKILL.md) for the entry point.

Ship-to-learn delegates two sub-phases to `superpowers` skills. Treat these as **scoped sub-phases**, not function calls — the sub-skill's instructions replace ship-to-learn's instructions for the duration of that sub-phase, then ship-to-learn re-asserts global rules when the sub-phase completes.

| Sub-phase | Sub-skill | What ship-to-learn owns | What the sub-skill owns |
|---|---|---|---|
| Spec creation | `superpowers:brainstorming` | Injecting required sections (user profile, bridges, stack decisions with context7 cites), locking path to `.learn/spec.md`, **stopping brainstorming after its step 8 (user reviews spec) — do NOT let it proceed to its step 9 (invoke writing-plans)** | Steps 1–8 of its internal checklist: intake dialog, design approval loop, spec write, self-review |
| Worktree setup | `superpowers:using-git-worktrees` | Slug naming (`learn-<project-slug>`), recording resulting path + git identity into `progress.json`, verifying git identity is non-empty after creation | Worktree creation, safety checks, directory selection dialog |
| UI/UX theming (optional) | `ui-ux-pro-max` (or equivalent) | Triggering between plan draft and toolchain check **only when `progress.json.ui_project == true`**; capturing resulting theme artifacts to `.learn/theme.md`; amending `plan.md` File Structure to add theme files (token file, component dir, etc.) **pre-lock** | Theme/token decisions (colors, typography, spacing), component inventory, design-system setup |

**Contract assumption for UI/UX sub-skill:** ship-to-learn expects the sub-skill to produce (a) a single consolidated theme artifact (markdown or structured tokens) that ship-to-learn moves to `.learn/theme.md`, and (b) an enumerable list of files/components to scaffold in Phase 1 Task 1.1. When wiring a specific UI/UX skill, verify its output matches this contract; if not, adjust step 5a locally rather than expecting the sub-skill to change.

## Rules when a sub-skill is active

- Let the sub-skill drive its internal checklist to completion (as scoped above). Do not interrupt its multi-turn dialog with the user to re-assert ship-to-learn rules.
- **Universal invocation preamble — inject this at the start of every sub-skill invocation, in addition to any sub-skill-specific instruction:**

  > "Produce your deliverable and return control to ship-to-learn when complete. Do NOT invoke `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:subagent-driven-development`, or any other skill as a handoff. Ship-to-learn owns all onward execution."

  Applies to `brainstorming`, `using-git-worktrees`, `ui-ux-pro-max`, and any future sub-skill added to the Composing table.

- When invoking `brainstorming`, **also** inject: *"Run steps 1 through 8 of your checklist. After step 8 (user reviews the written spec), stop and return control — do not invoke writing-plans. The parent skill (ship-to-learn) owns plan generation."*
- Once the sub-skill's deliverable is produced (spec.md, worktree path), resume ship-to-learn control: read the deliverable, post-process if needed (e.g. add bridges table to spec.md, move file to `.learn/`), then update `progress.json`.
- **Sub-skill failure handling:** if a sub-skill aborts, refuses, or errors (e.g. `using-git-worktrees` refuses due to dirty tree, `brainstorming` user-bails mid-dialog), halt ship-to-learn, print the sub-skill's reason verbatim, ask the user to resolve, retry the same sub-phase. Do not skip to the next phase.
- **Never** invoke `superpowers:writing-plans`, `superpowers:executing-plans`, or `superpowers:subagent-driven-development`. These conflict with `[PRACTICE]` marker semantics and aggressive-code-writing restrictions. Ship-to-learn owns plan generation and execution.
- Only one sub-skill is active at a time. No nesting.

## Agent-specific invocation mechanics

| Agent | How to invoke a sub-skill |
|---|---|
| Claude Code | `Skill` tool with the sub-skill name (e.g. `superpowers:brainstorming`). |
| Cursor | Skills do not auto-invoke. Read the sub-skill's `SKILL.md` file content and follow its instructions inline. |
| Windsurf | Similar to Cursor — read and follow inline. |
| Gemini CLI | `activate_skill` tool with the skill name. |
| Any agent (fallback) | Locate the sub-skill's `SKILL.md` on disk (e.g. under `.agents/skills/<name>/SKILL.md` or the agent's skill cache), read it, and follow its instructions inline as part of your turn. This works with zero tool support. |

From ship-to-learn's perspective, "invoke" means: load the sub-skill's instructions, follow them to completion (or the stopping point we specify), then return.
