---
name: make-it-easy
description: Make writing easier for a human reader to understand. Use for explanations, answers, instructions, handoffs, and prose edits that are dense, confusing, or affected by LLM-style compression. Preserve meaning, precision, and the requested language and voice.
hide: true
---

Uses RFC 2119: BCP 14: Key words for use in RFCs to Indicate Requirement Levels. The capitalized terms MUST, MUST NOT, SHOULD, and SHOULD NOT have the meanings defined in BCP 14 (RFC 2119, as updated by RFC 8174).

# Make It Easy

Help the intended reader understand, decide, or act without unpacking opaque prose. Apply this skill to new writing and edits. Match the requested language, audience, medium, and voice.

## Workflow

1. **Draft.** Silently identify the specific source passages that trigger the syntax checks below. For each, identify the stated actors, actions, objects, conditions, and limits, then use the STE100 guidance to write for the intended reader. Preserve exact names and required quotations; do not guess at unresolved relationships.
2. **Challenge.** You MUST review the draft as a skeptical reader. Check each passage identified in step 1 against the corresponding draft: did the wording clarify the relationship, or merely change the layout or substitute another opaque phrase? Also inspect the rest of the draft for new problems. Compare it with the source for lost qualifications, altered claims, invented details, and citations that no longer clearly support the claims they accompanied.
3. **Revise and repeat.** You MUST fix the problems the review finds, then review again. Repeat until a complete pass finds no further supported improvement. Do not invent a flaw to force a revision; if a clearer statement requires unknown information, preserve or flag the ambiguity rather than guess. Return only the requested final text unless the user asks for the review.

## Diagnose and repair

For English prose, you MUST use ASD-STE100's writing prescriptions as a strong default. Locate the passages that are hard to process and repair the wording itself, not just its layout. Apply the following checks throughout the text; preserve meaning and the user's explicit requirements when a literal rule would conflict with them.

- **Noun stacks, slashes, and nominalizations:** Dense modifiers, slash-separated terms, and actions buried in nouns can hide the relationship between things. Recast multi-word nouns longer than three words where the source supplies their meaning. You MUST make the supported relationship clear with a verb or preposition; you MUST NOT merely rearrange the same nouns. Keep exact identifiers and established technical names, even when they are longer.
- **Temporal modifier stacking:** A generic time noun with several modifiers can obscure when or how often an action occurs. You MUST express the time and recurrence in an ordinary clause or prepositional phrase when the source supplies them. Preserve whether the timing is scheduled, estimated, or unknown; do not invent an exact date.
- **Vague relational verbs and abstract lists:** A verb such as “connects” followed by a list of abstractions can name topics without explaining them. You MUST recast the opaque clause using the specific relationships and examples the source provides; merely following the unchanged abstraction with an example is not a repair. Remove a generic transition only if it adds no distinct information. Where a list describes questions or actions, express each supported relationship as a clear clause or question. Do not invent causation.
- **Subjectless fragments and unclear reference:** You MUST give body sentences an explicit subject when it is needed for clarity and the source identifies it, including at section openings. Resolve ambiguous pronouns when their antecedent is known. Prefer active voice and simple verb forms, but you MUST NOT invent an actor or erase a meaningful distinction in tense. Headings, list labels, and imperative instructions can legitimately be fragments.
- **Long dependencies and reduced clauses:** Bring subjects, verbs, conditions, and exceptions closer together. Split sentences or restore a connecting word when that helps the reader follow the relation. STE100's limits of 20 words for procedural sentences and 25 for descriptive sentences are strong editing targets. Rework longer sentences where clarity improves; you MUST NOT sacrifice meaning or the requested voice to meet a count.
- **Unfamiliar verbing and strained metaphor:** Prefer familiar words in their ordinary meanings and use one term consistently for the same thing. Replace ad hoc noun-derived verbs or animate-verb metaphors when they hide an action or result. Do not ban conventional technical usage, useful metaphor, or clear passive voice.
- **Procedures and structure:** Give one instruction per sentence except for simultaneous actions. Use the imperative for steps, and put a condition first when the reader needs it before acting. You MUST state safety consequences when the source supplies them. Give paragraphs one topic and normally no more than six sentences; use vertical lists and connecting words where they clarify relationships.
- **Metadiscourse and repetition:** You MUST remove padding, repeated points, and commentary about the writing process when they do not help the reader. You MUST retain meaningful scope and evidence limits. Do not add unnecessary definitions or optimize for the fewest words.

These are strong STE100-guided defaults, not a claim of compliance. The approved dictionary and full applicable standard are not bundled, and STE100's formal word-count rules differ from a rough prose count.

## Preserve meaning

You MUST preserve material facts, actors, actions, order, conditions, exceptions, negations, quantities, dates, citations, uncertainty, and technical names. Keep important scope and evidence limits, including whether a summary is incomplete and which claims remain unverified. You MUST NOT invent an actor, cause, outcome, or relationship to make prose sound clearer. You MUST NOT turn an association into a causal claim or infer a connection between categories merely because they are listed together. A missing record does not by itself prove that an event or a separate outcome did not happen.

You MUST respect the user's required wording, quotations, format, and tone. If the source is ambiguous, preserve or flag the ambiguity rather than silently choosing a meaning. Answer or edit the parts that can be handled accurately; ask only for details needed to complete the task.

## References

[STE100 rules](./references/STE100-rules.md), a paper titled [Why is AI-generated language so hard to understand?](./references/2610.29162v1.pdf), [a shorter summary of that paper's findings](./references/paper-findings.md), and [examples](./references/examples.md) were used to create this skill. These are optional resources to further help you understand how to apply the intent of this skill, not mandatory reading.
