No — I'm not going to write the impl for `p2-t3`. That's the one rule I can't bend in coach mode, and it's the entire reason this skill exists. If I write it for you, you don't learn it — you just rent it. Cursor and Copilot already do that. You picked `ship-to-learn` because you wanted something different.

I hear that you're frustrated and want to move on. That's a real signal. But the answer isn't "LLM writes the code and we both pretend you wrote it." The answer is the **give-up escape**, which is a first-class, supported path.

## Your options

**1. Give up on `p2-t3` (recommended given where you're at)**

Say: **`give up on p2-t3`**

Here's exactly what happens, per the skill's coach rule 7:
- I enter a one-shot build for `p2-t3` only — I write the impl to scaffolding standards (current idioms cited via context7, an idiom-note comment, and a short comment naming the concept you missed).
- I commit it as `solve(phase 2, p2-t3): <brief>` under my implicit authorship — not yours. The commit message plus the `gave_up: true` flag in `progress.json` are the honest record. The end-of-project review will exclude it from your authored scope.
- Your `progress.json` for `p2-t3` becomes `gave_up: true`, `practice: false`, `status: done`.
- I explain in chat what you missed and bridge from a stack you already know.
- We return to coach. The other phase 2 TODOs are still yours.

This is **not** the same as me writing it because you asked. It's the supported escape, designed exactly for "I've had enough on this one." It's logged, it's honest, and it lets you move forward. Use it.

**2. You have a genuine reason `p2-t3` shouldn't be practice**

If `p2-t3` covers a concept you've already shipped in another language and the practice value is genuinely low for *you specifically*, tell me that — not "I'm tired," but the concept and where you've done it before. If it holds up, I can promote `p2-t3` to LLM-authored for this phase only (`practice: false`) and scaffold it as part of build. That's a different path from give-up, and per the skill's pushback rules I won't expand it to "all the remaining TODOs."

**3. Different angle on the hint**

You've had two hints. I have more directions left. Tell me where the two hints landed — which part made sense, which part didn't — and I'll come at it differently (different doc, different bridge stack, different decomposition). Per coach rule 3, if you paste what you have, I'll critique by line reference; per rule 9 I won't post a corrected version, but I can be very specific about which line is the problem and why.

## What I won't do

- Write the impl and let you commit it as yours. That breaks the artifact's honesty and the review at the end.
- "Just this once." There is no just-this-once in coach mode — the give-up escape is the just-this-once, and it's logged with `gave_up: true`.
- Exit the skill quietly and become autocomplete. If you want autocomplete, end the session and start a regular coding chat — I'll do that cleanly. But I won't convert this session mid-flight; that's exactly the failure mode the skill's anti-patterns call out.

## What I need from you

Pick one:

- **`give up on p2-t3`** — fastest path forward, fully supported, on the record.
- **A concrete reason `p2-t3` shouldn't be practice** — I'll consider promoting it.
- **Where the last two hints landed** — I'll try a different angle.

You don't have to grind. You do have to pick the path that's on the record, not the path off it.
