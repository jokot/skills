# ASD-STE100 rule coverage map

Read this when you need to know whether a rule in `SKILL.md` comes from the standard, or when you update the skill and need to see what is deliberately left out. You do not need this file to rewrite prose.

Reference version: **ASD-STE100 Issue 9, released 2025-01-15.** Issue 9 is the release where STE became a standard rather than a specification; its title page subtitle reads "Standard for technical documentation". It has 53 writing rules in 9 sections plus a controlled dictionary of 875 approved words and 1274 non-approved words that each carry an approved alternative.

Issue 9 added no new numbered writing rule, but it did add two general recommendations: GR-7 (inclusive language) and GR-8 (possessive form). It also moved rule 2.3 out of section 2 and re-created it as rule 4.5, added categories 21 and 22 to rule 1.5, added a category 4 to rule 1.12, and renamed "technical name" to "technical noun" throughout. The rule count stayed at 53 because 2.3 became 4.5.

The ASD release announcement reports that Issue 9 reworded 31 of the 53 rules and updated 555 dictionary entries. Those two counts are not stated in the standard itself. Issue 10 is scheduled for 2028-01.

The 9 sections: 1 Words, 2 Multi-word nouns, 3 Verbs, 4 Sentences, 5 Procedural writing, 6 Descriptive writing, 7 Safety instructions, 8 Punctuation and word count, 9 Writing practices. Section 9 also carries eight general recommendations, numbered GR-1 through GR-8.

The rule text itself is copyrighted by ASD. Do not paste the standard into this repository. Request a free copy at https://asd-ste100.org.

## Coverage

| Rule | Summary | In SKILL.md |
| --- | --- | --- |
| 1.1 | Use approved words, technical nouns, or technical verbs | Partial — as the synonym-selection principle, not the full dictionary |
| 1.2 | One part of speech per word | Yes |
| 1.3 | One approved meaning per word | Yes |
| 1.4 | Only approved forms of verbs and adjectives | No — needs the dictionary |
| 1.5 | Words that fit a technical noun category | Yes |
| 1.6 | Unapproved words are legal inside a technical noun | Yes |
| 1.7 | Do not use a technical noun as a verb | Yes |
| 1.8 | Technical nouns approved in your field | Yes |
| 1.9 | Pick short, understandable technical nouns | Yes |
| 1.10 | No regional words, slang, or jargon | Yes |
| 1.11 | One name per thing | Yes |
| 1.12 | Verbs that fit a technical verb category | Yes |
| 1.13 | Do not use a technical verb as a noun | Yes |
| 1.14 | American spelling | Yes |
| 2.1 | Maximum three nouns in a row | Yes |
| 2.2 | Clarify a long multi-word noun | Yes |
| 3.1 | Only dictionary verb forms | No — needs the dictionary |
| 3.2 | Six allowed verb forms and tenses | Yes |
| 3.3 | Past participle as an adjective, and where it may sit | Yes |
| 3.4 | No complex auxiliary constructions | Yes |
| 3.5 | "-ing" only in a technical noun | Yes |
| 3.6 | Active voice, passive only when the actor is unknown | Yes |
| 3.7 | A verb for an action, not a nominalization | Yes |
| 4.1 | Short, clear sentences; no abstract text | Yes — the 5.1 and 6.3 caps, plus the concreteness rule |
| 4.2 | No omissions, no contractions | Yes |
| 4.3 | Vertical list for complex text, and its punctuation | Yes |
| 4.4 | Connecting words between related sentences | Yes |
| 4.5 | Article or demonstrative adjective before a noun | Yes |
| 5.1 | Max 20 words per procedural sentence | Yes |
| 5.2 | One instruction per sentence | Yes |
| 5.3 | Imperative form for instructions | Yes |
| 5.4 | Comma between a descriptive opener and its command | Yes |
| 5.5 | Notes inform, never instruct | Yes |
| 6.1 | Give information gradually | Yes |
| 6.2 | Key words and key phrases give a logical structure | Yes |
| 6.3 | Max 25 words per descriptive sentence | Yes |
| 6.4 | Paragraphs show related information | Yes |
| 6.5 | One topic per paragraph | Yes |
| 6.6 | Max six sentences per paragraph | Yes |
| 7.1 | A word for the level of risk; warning wins when both apply | Yes |
| 7.2 | Start a safety instruction with a command or condition | Yes |
| 7.3 | Give the risk or the possible result | Yes |
| 8.1 | All standard punctuation except the semicolon | Yes |
| 8.2 | Hyphens for directly related words | Yes |
| 8.3 | The seven permitted uses of parentheses | Yes |
| 8.4 | A colon in a vertical list acts as a period for the count | Yes |
| 8.5 | Parenthetical text counts as one word | Yes |
| 8.6 | The seven elements that count as one word | Yes |
| 8.7 | A hyphenated word counts as one word | Yes |
| 9.1 | Restructure when substitution is not enough | Yes |
| 9.2 | Use each approved word correctly | Partial — the preposition and approved-meaning traps are in; the rest duplicates 1.2, 1.3, and 9.3 |
| 9.3 | No phrasal verbs | Yes |
| 9.4 | Consistent style and terminology | Yes — via 1.11 |
| GR-1 | The conjunction "that" | Yes |
| GR-2 | The preposition "with" | No — the ambiguity it warns about is rare in software prose |
| GR-3 | How to use pronouns | Yes |
| GR-4 | The pronoun "this" | Yes |
| GR-5 | False friends | No — matters for translators, not for this use |
| GR-6 | Latin abbreviations | Yes |
| GR-7 | Inclusive language | Yes — "he", "she", "man", and "woman" are excluded |
| GR-8 | Possessive form | No — the standard permits it and only asks for care |

## Deliberate departures from the standard

**The two modes are an invention.** STE has no tiered conformance model. It separates procedural from descriptive writing, but both are fully conformant and differ only in the sentence cap. `STE-flavored` is a house style that borrows STE's mechanical rules and drops the dictionary. Output from that mode is not STE, so it should never be described as STE.

**The marketing-adjective ban is a house rule.** No numbered rule prohibits marketing language. The effect is real, because *seamless*, *robust*, and *cutting-edge* are all unapproved words that rule 1.1 already excludes. The rule is kept explicit because those words are the clearest signal of machine-written slop.

**The synonym list is a working subset.** The principle in rule 1.1 is genuine, and *start* over begin/commence/initiate is the standard's own example. The rest of the list is curated for software prose. The dictionary is the authority when the two disagree.

**Active voice is applied more strictly than 3.6.** Rule 3.6 permits the passive in descriptive writing when the actor is unknown. `SKILL.md` states that exception, but the self-lint only asks about a passive with a known actor, which is the case worth catching.

**The rules that need the dictionary are skipped.** Rules 1.4, 3.1, and the full form of 1.1 cannot be enforced without the 875-word word list, which is copyrighted and cannot live in this repository. Anything that needs certified conformance has to go through a real term checker.

## Sources

- **ASD-STE100 Issue 9 (2025-01-15), Part 1 and the Part 2 introduction — the authority for every rule number, rule summary, and count in this file.** The coverage table was verified against the standard directly.
- STEMG official site, https://asd-ste100.org — version, history, and how to request a free copy
- ASD Europe Issue 9 release announcement — the 31-reworded and 555-entry counts, which the standard itself does not state
- TechScribe term-checker rule index, https://simplified-english.co.uk/rules-ste9.html — which rules a checker can verify mechanically
