---
name: ste-writing
description: Rewrite prose (docs, READMEs, PR descriptions, error messages, release notes, comments — never code) into ASD-STE100 Simplified Technical English to remove "AI slop". Use when asked to make writing not sound like AI, make docs clear or plain, enforce a controlled writing style, or write technical documentation that reads human. Two modes — strict (procedures/safety) and STE-flavored (general prose).
---

# ste-writing

Write prose in ASD-STE100 Simplified Technical English. This applies to documentation, READMEs, pull-request text, error messages, release notes, and comments. It does not apply to code, identifiers, or command syntax. It is not for marketing copy, essays, or anything that needs a voice — STE strips voice on purpose.

Rule numbers point to ASD-STE100 Issue 9 (January 2025), which has 53 rules in 9 sections, a dictionary of 875 approved words, and 1274 non-approved words that each carry an approved alternative. This file carries the rules that fire on software prose. `references/ste-rule-map.md` records which of the 53 this file covers, which it skips, and where it leaves the standard on purpose.

## Rules

WORDS
- Use one name for one thing (1.11). Do not call the same item by two different names.
- Give each word one meaning (1.3). "fall" means to move down, not to decrease. "close" means to shut, not near.
- Give each word one part of speech (1.2). If "test" is a noun in your text, do not also use it as a verb.
- Where English offers synonyms, STE keeps one and drops the rest (1.1). Use: start (not begin/commence/initiate), use (not utilize/leverage), help (not facilitate), make sure (not ensure), before (not prior to), after (not subsequent to), about (not regarding/concerning), get (not obtain/acquire), show (not demonstrate), also (not additionally/furthermore/moreover).
- A word outside the dictionary is still legal when it is a technical noun, or part of one (1.5, 1.6, 1.8). This is what makes STE usable here: worktree, mutex, webhook, and clipboard are all fine. Pick the short, plain technical noun (1.9).
- A verb outside the dictionary is legal when it fits a technical verb category (1.12): commit, merge, deploy, rebase, and cache are all fine. But when an approved verb plus a technical noun says the same thing, use those instead: "add a label to the issue", not "label the issue".
- Do not use a technical noun as a verb (1.7): "add the user", not "onboard the user". Do not use a technical verb as a noun (1.13): "deploy the service", not "do a deploy".
- Keep prepositions literal (9.2). "above" and "below" are for physical position, not for limits: "more than 20 psi", not "above 20 psi". The same trap catches "see" for "come to know", "go through" for a threshold, and "turn" for a change of color.
- No slang and no jargon (1.10): not "nuke the cache", not "the happy path", not "spin up a box".
- American spelling (1.14).
- House rule, not a numbered STE rule: no marketing adjectives — seamless, robust, powerful, cutting-edge, effortless, world-class, next-generation, revolutionary. The dictionary already excludes them, but slop is full of them, so check for them directly.

MULTI-WORD NOUNS
- Use a maximum of three nouns in a row (2.1). This rule fires more than any other on software prose.
- To break a long cluster, use a preposition: "the retention policy for clipboard history", not "clipboard history retention policy setting".
- When you must keep a long term, write it out once, then give it a short name or hyphenate it (2.2).

VERBS
- Use only these forms (3.2): infinitive, imperative, simple present, simple past, simple future, and past participle as an adjective. This removes "will have been processed", "is being written", and "should be able to be configured".
- Use a past participle as an adjective only before a noun, or after a form of "to be", "to become", or "to stay" (3.3): "the cached response", "the response is cached".
- Active voice (3.6). "the parser reads the file", not "the file is read by the parser". In descriptive prose, use the passive only when the actor is unknown.
- Use a verb for an action (3.7). "analyze the log", not "perform an analysis of the log".
- No stacked auxiliaries (3.4). Not "it is important to note that this may help to improve". Write "this improves X".
- Use the "-ing" form only inside a technical noun (3.5). "the landing page" is fine. "we are supporting Postgres" becomes "we support Postgres".
- Do not build phrasal verbs from two words (9.3): install (not set up), start (not spin up), do (not carry out), find (not figure out), remove (not take out). Phrasal verbs are the largest translation hazard in the standard. A few are approved with restricted meanings, for example "put on" and "come on".

SENTENCES
- One instruction per sentence (5.2), unless two actions happen at the same time.
- Max 20 words in an instruction (5.1), max 25 in descriptive prose (6.3), max 25 in a note (5.1).
- Count words this way (8.4 thru 8.7). Each of these is one word: text in parentheses, a hyphenated word, a number, a number with its unit, an abbreviation, an alphanumeric identifier, quoted text, a title or heading or label, and a proper noun of a person, group, organization, or country (8.6). A colon in a vertical list ends the count the way a period does (8.4).
- Be concrete (4.1). Do not name an effect without its values: not "different settings change the retention time", but "the retention time is 30 days at the default setting". Do not describe a state where an action belongs: not "no orphaned locks are permitted", but "make sure that there are no orphaned locks".
- Write instructions in the imperative (5.3).
- Put the condition before its command, and separate them with a comma (5.4).
- No contractions, and do not drop words to hit the cap (4.2). Use articles: a, an, the.
- Use an article or a demonstrative adjective before a noun (4.5).
- Use only the pronouns that the dictionary approves, and only when the noun is unambiguous (GR-3). "he" and "she" are not approved, and neither are "man" and "woman" outside a context that needs them (GR-7). Put a noun after "this" (GR-4): "this setting", never a bare "this".
- Use connecting words to join sentences on related topics (4.4).
- Keep the conjunction "that" (GR-1). "make sure that the port is free", not "make sure the port is free".

STRUCTURE
- Give information gradually (6.1). Put one idea in a sentence, then build on it. Do not front-load a paragraph with every fact at once.
- Repeat the key words that carry the topic, and do not vary them (6.2). If the thing is a "retention policy" in the first sentence, it is a "retention policy" in the fourth — not a "cleanup rule".
- Use a paragraph to hold related information (6.4). One topic per paragraph, max six sentences (6.5, 6.6).
- For steps, use a vertical list, one action per item, imperative form (4.3). Mark items with a dash, a bullet, a letter, or a number. Put a colon before the first item. Start each item with an uppercase letter. Use an article before the subject noun. Put a period on an item that is a full sentence and on the last item, none on a fragment, and never a comma or a semicolon.
- Notes give information only, never an instruction (5.5).

SAFETY AND ERROR TEXT
- Start with a word that gives the level of risk (7.1): WARNING for injury, CAUTION for damage. When both risks apply together, use WARNING. A project can define its own equivalents.
- Then give the command or the condition (7.2). Then give the risk or the result (7.3).
- Example: "CAUTION: Do not stop the service during a migration. The database can keep a partial schema."

PUNCTUATION
- No semicolons (8.1). Write two sentences. The em dash is not banned by STE, only the semicolon is — add "no em dash" yourself if you want it gone.
- Use hyphens to connect words that are directly related (8.2). Use parentheses only for these seven purposes (8.3): a reference to an illustration or text, an identifying letter or number, a work-step identifier, an abbreviation, a singular and plural form together, an explanation, or an alternative.
- No Latin abbreviations (GR-6). Write "for example", "that is", and "compare".

METHOD
- When a word-for-word swap does not produce a clean sentence, throw out the structure and write a new sentence (9.1). Most slop needs this, not substitution.

Write only the requested text. No preamble, no summary, no closing remarks.

## Modes

- **strict** — procedures, runbooks, safety text, error messages: apply every rule, both length caps, and the dictionary discipline. Aim at output that conforms to the standard.
- **STE-flavored** — general prose (READMEs, PR descriptions, docs): apply the sentence, paragraph, verb-form, active-voice, pronoun, and phrasal-verb discipline. Relax the 875-word dictionary lockdown so the text keeps enough range to read naturally. This output is not conformant STE, so do not label it as STE.

## Self-lint (run before returning text)

1. Any instruction over 20 words, or any descriptive sentence over 25? Split it. Count a parenthetical, a hyphenated word, a number with its unit, an abbreviation, an identifier, quoted text, and a heading as one word each.
2. More than three nouns in a row? Rewrite with a preposition.
3. Any semicolon? Replace with a period.
4. Any contraction? Expand it.
5. Any passive voice with a known actor? Make it active.
6. Any verb form outside the six allowed? Rewrite the sentence.
7. Any "-ing" main verb, nominalization ("perform an analysis"), or phrasal verb ("spin up")? Replace with a plain verb.
8. Any bare "this", "it", or "they" without a clear noun? Name the noun.
9. Any Latin abbreviation? Expand it.
10. Same thing named two ways? Pick one name.
11. Any sentence that names an effect without its values, or describes a state where an action belongs? Give the numbers, or give the command.
12. Any "above" or "below" used for a limit? Write "more than" or "less than".

The mechanical rules above are lintable and are what removes slop. Full STE also needs human judgment (the right technical noun, whether a sentence "makes good sense") — a checker cannot certify that, and slop is not about that. This skill fixes the FORM of slop. It cannot make a hollow paragraph true.

Free official standard (do not paste it in full; it is copyrighted): https://asd-ste100.org
