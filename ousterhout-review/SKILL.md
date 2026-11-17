---
name: ousterhout-review
description: Review code, diffs, pull requests, refactors, and module boundaries through John Ousterhout's A Philosophy of Software Design. Use when asked for an Ousterhout review or Ousterhout test, or a focused review of complexity, deep modules, information hiding, and long-term ease of change. Not a general correctness or style review.
hide: true
---

# Ousterhout review

Answer one question: **Does this design make the system easier to understand and modify, or does it accumulate dependencies and obscurity?**

The unit of judgment is the system experienced by its developers and callers, not the size or apparent neatness of an individual function. More code inside a module can mean less complexity outside it. A small diff can introduce a large conceptual burden.

Read [the review lens](references/review-lens.md) bundled with this skill before evaluating the code. It contains the necessary principles, questions, counterexamples, and red flags; do not locate or load the book or its source text. Apply the guidance as investigative prompts, not automatic violations. Reducing total complexity takes precedence over any individual principle.

## Scope and authority

- Review only through this lens. Discuss correctness, performance, tests, or operational behavior when they establish a design dependency, hidden contract, or justified tradeoff; do not turn this into a general bug hunt, security audit, style review, or test-coverage audit.
- Review is read-only unless the user separately requests implementation. Do not change code, add dependencies, or refactor as part of the review.
- Establish the target and baseline from the user's request and available repository context: supplied diff, commit range, current changes, named files, or a module. Do not silently choose a different target. Ask only if a consequential ambiguity cannot be resolved from the available context.
- For changes, judge the complexity delta against the baseline. Distinguish introduced, worsened, and pre-existing problems. Report a pre-existing issue only when the change makes it consequential or the user requested review of that existing design; label it explicitly.
- Preserve the actual requirements and constraints. Do not recommend removing required functionality to make a design look simpler.

## Review method

### 1. Build a working model, not a diff-only opinion

Identify each changed responsibility, its entry points, and the modules that own the relevant knowledge. Read the surrounding implementation, actual callers, contracts, and relevant tests. Inspect unchanged code when it is needed to judge a changed boundary. Use tests as evidence of intended semantics, not as proof of good design.

Trace representative operations from caller intent through the changed logic to the external boundary or stored result. Include error paths, state transitions, and lifecycle ordering where the design depends on them. For each important boundary, establish:

- what capability it promises;
- what decisions it genuinely hides;
- what a caller must know to use it correctly, including informal obligations;
- which other modules must change if those decisions change.

Account for every changed semantic unit. In a large review, organize the work by responsibilities and track coverage; do not substitute a few sampled files for the requested scope. Expand dependency tracing when it can resolve a candidate finding. Stop expanding when the contract, consumers, and relevant constraints are understood—not after an arbitrary file count and not after reading the whole repository.

### 2. Examine who must know what

Use the reference to investigate these connected questions:

- **Depth and information hiding:** Does a boundary remove more knowledge than it introduces? Is a representation, algorithm, format, or policy understood in multiple places?
- **Decomposition and layers:** Can the pieces be understood independently? Does each layer supply a distinct abstraction, or merely forward calls, state, and configuration? Is the split based on knowledge or just execution order?
- **Generality and ownership:** Is the mechanism easy to use for current needs without encoding a particular caller's feature? Are special cases and caller-side bookkeeping necessary? Does complexity belong inside this module, or would moving it there leak unrelated policy?
- **Errors and contracts:** Can the normal contract cover an edge case naturally? Can recovery or handling be owned in one place? Does the abstraction expose failures that callers genuinely need to distinguish?
- **Obviousness:** Do names, comments, conventions, and state representations make the important information discoverable? Does a reader need to reconstruct a contract from implementation details or remember an invisible dependency?

Do not emit one finding per question. These are ways to uncover root causes, often the same root cause.

### 3. Test candidates against a maintenance task

For each candidate, choose a concrete task supported by the requirements or current usage: changing an encoded format, replacing an internal representation, adding another operation in an existing family, changing a policy, or using an API from an existing caller. Label a future scenario as hypothetical; do not invent promised features.

Trace what a maintainer would have to find, understand, remember, and modify. Establish this causal chain:

**design decision → dependency or obscurity → change amplification, cognitive load, or an undiscoverable obligation → concrete maintenance consequence.**

A red-flag label alone is not a finding. Neither are a line-count threshold, an unfamiliar idiom, a pattern name, or an aesthetic preference. Explain exactly what knowledge is duplicated or missing and who pays for it. Small recurring costs still count; do not dismiss them just because today's patch is small.

### 4. Design it twice; challenge your recommendation

For a consequential finding, compare the current design with a materially different alternative. Consider removing an unnecessary boundary, joining shared knowledge, extracting a genuinely deep mechanism, moving specialization, strengthening a contract, or choosing a representation that eliminates an exceptional state. A rename or added comment is enough only when the problem really is missing precision, not misplaced responsibility.

Evaluate the alternative across the affected system: caller knowledge, interface obligations, ownership of decisions, change sites, and implementation complexity. State the main cost or constraint. Prefer the smallest coherent redesign that removes the cause; do not automatically prefer the smallest textual patch or a new abstraction.

Before keeping the finding, ask:

- Does the existing boundary provide real dispatch, adaptation, isolation, or other functionality that I overlooked?
- Is the information essential to the caller, rather than a detail that can safely be hidden?
- Are the apparently repeated pieces actually the same decision, or independent policies that happen to look alike?
- Would my alternative make current use harder, introduce a grab-bag context, or entangle unrelated responsibilities?
- Is added complexity justified by an established constraint or demonstrated performance benefit?

Do not infer the author's mindset or process from the code. Practice design-it-twice yourself; do not claim that the author failed to do it. Evaluate comments present in the result, not whether they were written first.

### 5. Keep only evidence-backed findings

A finding needs specific code evidence, a complexity mechanism, a credible consequence, and an actionable direction. Cite the primary site and the callers or companion sites needed to demonstrate it, with real file/line references. For a diff review, anchor to changed code where possible and explain the introduced or worsened behavior. Name the relevant principle; a chapter/section attribution from the bundled reference is optional. Never look up source text or page numbers for the review. Use code—not the author's authority—as proof.

Resolve counterevidence before reporting. Separate an observed fact from a projected consequence. If a critical caller, requirement, or boundary is unavailable, state the missing evidence rather than manufacturing a defect. Deduplicate by root cause and prioritize by spread, frequency of exposure, and difficulty of safe change, not by how many red flags apply.

Finish when the requested scope is covered and candidates are substantiated or rejected. Do not force a finding quota.

## Deliver the review

Honor a user-specified review format. Otherwise use:

1. **Verdict:** `Pass for reviewed scope`, `Needs revision`, or `Inconclusive`, followed by a one-sentence reason. A pass means no supported actionable design findings in the reviewed scope—not proof of correctness or a universal design certification. If supported findings coexist with missing coverage, report them and make the verdict inconclusive.
2. **Coverage:** Target/baseline, responsibilities and paths actually examined, and consequential gaps. Keep it compact; distinguish inspected code from assumptions. For a whole-module review, state that there is no change baseline.
3. **Findings, highest impact first:**
   - **[High / Medium / Low] Imperative title — `file:lines`**
   - **Evidence:** The boundary/decision and the other sites that establish the issue; identify whether it is introduced, worsened, or in-scope existing debt.
   - **Complexity cost:** The causal mechanism and concrete maintenance task, in ordinary language. Name the relevant Ousterhout principle briefly; book citations are optional.
   - **Better design:** The recommended change, why it reduces total complexity, and its main tradeoff. Include a short interface sketch only if it clarifies the boundary.
4. **Unresolved evidence**, only when it materially limits the judgment: what is missing and what inspection would resolve it. An unanswered question is not a confirmed finding.

High: a widely exposed boundary or hidden invariant makes ordinary changes unsafe or requires broad redesign. Medium: meaningful recurring knowledge or coordination costs in an affected subsystem. Low: a localized but concrete ambiguity or unnecessary obligation. Tie the severity to evidence; do not promote every principle violation to a blocker.

If there are no findings, say so. Briefly identify a demonstrated strength when it explains the pass; do not manufacture praise or append a generic checklist. Keep the report useful to a reader who has not read the diff or the book.
