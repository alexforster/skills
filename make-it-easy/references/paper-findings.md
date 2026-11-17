# What the paper does—and what it does not show

This note summarizes the evidence behind the review heuristics in [worked examples](worked-examples.md). The paper reports group-level patterns in generated English and a small training intervention. It does not test whether a particular rewrite helps a reader understand a passage. Use its findings to notice possible costs for readers. Do not use them as a readability score or a mandate to rewrite every marked construction.

## Source and study design

Martin, Clark, and Mahowald, “Why is AI-generated language so hard to understand?”, arXiv:2610.29162v1 (21 October 2026), [bundled source PDF](2610.29162v1.pdf). Page numbers below refer to this bundled copy.

Experiment 1 (Sections 2–3, pp. 2–7) compares 14 model variants: nine frontier or near-frontier systems and five smaller variants (two Llama and three Olmo). It examines three settings: chat, coding-agent sessions, and essays. Not every model appears in every setting. 12 models contribute chat, nine coding, and all 14 essays. The collection uses 10 chat topics, five public-code-repository tasks, and seven essay prompts. This is a varied but still small sample of models, prompts, and domains.

The paper measures language features that prior psycholinguistic research associates with processing difficulty:

- Long syntactic dependencies
- Object relative clauses, especially those without optional *that*
- Stacked noun compounds of three or more nouns
- Less-conventional noun-derived verbs
- Animate-preferring active verbs with inanimate subjects
- Rare word sequences (unattested bigrams and trigram-model perplexity).

The authors standardize dependency length for sentence length. They analyze text with spaCy dependency parses. An external classifier judges candidate noun-derived verbs and inanimate-subject constructions (Section 3.1, pp. 3–4; Appendix B.1, p. 16).

Keep three kinds of claim separate:

- **Observed feature distributions:** The authors count constructions and compare their rates. Relative to round-trip paraphrases, most frontier models show more rare n-grams, noun-derived verbs, inanimate subjects, and object relatives without *that*. Many also show longer dependencies and more stacked nouns in chat and coding. In the available human comparisons for essays, all nine frontier/near-frontier models show more inanimate subjects with animate verbs and more low-frequency n-grams. These measures lack separate human chat/coding baselines (Appendix B.4). Essay effects under round-trip translation are often smaller. Dependency length is unchanged or slightly longer after round trip, and stacked-noun differences are significant for only two models. Smaller Llama and Olmo base models are often closer to reference text, while Olmo Think resembles frontier models (Section 3.5, pp. 5–7). These are distributional results, not a finding that every instance is hard to read.
- **Established psycholinguistic associations:** The paper draws on prior literature that links several features to processing costs (Section 3.1, pp. 3–4; Section 2, pp. 2–3). These features include low predictability, long dependencies, object relatives, some noun-stack ambiguities, unusual noun/verb shifts, and animacy mismatches. Those findings motivate which signals to inspect. They do not establish that every occurrence causes difficulty, or that removing one always helps in context.
- **Proposed training mechanism:** The authors propose a possible mechanism. They hypothesize that task optimization for information density can favor compact forms without a matching pressure for successful human communication. Experiment 2 is a proof of concept, not evidence that this mechanism caused the observed habits in large deployed models. The authors fine-tune Qwen2.5-1.5B-Instruct on structured-event-to-English tasks with a length penalty. They compare settings with and without KL regularization. The penalty shifts outputs toward shorter trained alternatives and shows consistent near transfer to untrained *that* omissions. Far transfer appears only for the genitive alternation without KL, one of four far-transfer alternations tested (Sections 4.1–4.3, pp. 7–9; Appendix C, pp. 18–20).

## Comparisons and interpretation

Human/reference comparisons are approximate genre matches: OpenSubtitles for chat, StackExchange for coding, PERSUADE student essays for essays, plus Wikipedia (Section 3.3, p. 4; Appendix B.2, p. 16). OpenSubtitles, StackExchange, and Wikipedia baselines use samples of about 500,000 words each. These are not universal norms. The paper also has incomplete human baselines. Chat and coding do not have human n-gram baselines separate from their n-gram reference corpora. The authors did not score denominal verbs and inanimate subjects on human chat/coding corpora (Appendix B.4, p. 18).

To compare wording with an approximation of shared meaning, the authors translate each English sentence into Japanese and back to English with NLLB-200 (3.3B). They then compare the original and returned texts (Section 3.4, p. 5). Treat this round trip as a paraphrase-based proxy, not guaranteed meaning preservation or a recommended production pipeline. Translation can alter meaning as well as normalize wording. It cannot isolate style perfectly.

## Limits that matter for reviewers

The study is English-only, covers a subset of models, and uses few chat/coding seeds. Model and domain patterns vary (Section 7, p. 9). The paper reports no new direct human-comprehension experiment and validates no readability threshold. Feature associations and group-level counts therefore cannot predict the comprehension of a particular reader or passage. Automated parsing and classifier judgments can misclassify examples. The paired tests in Appendix B.4 are not adjusted for multiple comparisons.

The paper's inanimate-subject measure concerns active verbs that usually take animate subjects (Section 3.1, p. 4). It is not a measure of passive voice, and it does not establish that every figurative use of “expose” or “survive” belongs to that category. The skill's preference for stating the actual finding or outcome is a reader-focused editorial choice, not a result that the paper directly tested.

The intervention uses one small model, a narrow structured-generation task, and three seeds. It supports a plausible mechanism under those conditions, not a general causal account of frontier-model writing.

## Practical use

Treat a marked feature as a question: *Could a reader have to hold a relationship in memory, resolve a temporary ambiguity, or infer a rare combination here?* If so, consider stating the same relationship more explicitly. Then check that the rewrite preserves the original actor, action, timing, negation, scope, uncertainty, and exact technical names.

Keep conventional or deliberate constructions when they are clear. Do not optimize for short sentences, ban passive voice or metaphor, insert *that* mechanically, or claim an experimentally demonstrated improvement. The [worked examples](worked-examples.md) show how to make those checks and leave uncertain relations visible.
