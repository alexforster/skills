# The Ousterhout review lens

This companion distills John Ousterhout's *A Philosophy of Software Design* into a self-contained review lens. All guidance needed to apply the skill is here. Do not locate or load the book or its source text. Chapter/section labels are attribution aids, not lookup instructions; reviews need no page citations. The workflow, examples, and severity scheme are this skill's operationalization of the principles, not a review procedure prescribed by the author.

These are diagnostic questions, not universal commandments. The book explicitly places reducing complexity above following any individual technique (Preface).

## 1. Complexity is the cost of understanding and changing

**Attribution:** Ch. 2, “The Nature of Complexity,” §§2.1–2.4. Ch. 3, “Working Code Isn't Enough,” §§3.2–3.3.

The causes are **dependencies** (pieces cannot be understood or modified independently) and **obscurity** (important information is not apparent). Look for their observable consequences:

- **Change amplification:** One conceptual change requires coordinated edits in multiple places.
- **Cognitive load:** Correct work requires learning or retaining too many facts, choices, states, or interactions.
- **Unknown unknowns:** A necessary dependency or obligation is not discoverable where a maintainer reasonably looks. Do not use this phrase just because the reviewer lacks context; identify the actual hidden relationship.

Ask what a maintainer doing a common task must know, not how sophisticated the software is. Complexity encountered on common paths matters more than complicated internals that callers rarely see. Encapsulation can be as valuable as elimination.

Working behavior and passing tests do not settle design quality. Small costs accumulate; extending a workaround can be worse than repairing the owning abstraction. But strategic investment is incremental, not permission to design an entire speculative future system. Do not convert the book's suggested investment percentages into a review requirement or measured productivity claim.

**Finding:** A new status requires editing a declaration and an unrelated manually synchronized message table, with no discoverable connection or enforced invariant. Show both sites and the maintenance consequence.

**Not enough:** “This is complicated,” “this adds 500 lines,” or “this uses many files.” A broad one-time migration can remove future change amplification rather than create it.

## 2. Judge depth by knowledge removed, not implementation size

**Attribution:** Ch. 4, “Modules Should Be Deep,” §§4.1–4.7. Ch. 21, “Decide What Matters,” §§21.1–21.4.

A module can be a function, class, subsystem, or service. Its interface is **everything a user must know**, not just public signatures: ordering, lifetimes, side effects, ownership, errors, units, concurrency obligations, and performance constraints may all count.

- What useful capability is hidden behind this interface?
- Can callers use the contract without studying the implementation?
- Does an apparently tiny interface conceal essential obligations, making it a false abstraction?
- Does the common operation force callers to learn rarely needed features or construct several cooperating objects?
- Do new methods or configuration choices remove work, or merely move the work into the caller's head?

Deep means substantial capability behind a simple usable contract. It does not mean “large class,” and depth is not a ratio of line counts. Small modules can be useful; large modules can mix unrelated knowledge. Essential details must remain visible. A persistence API that omits durability semantics is not admirably simple.

**Finding:** Callers must assemble a storage writer, a buffer, and a flush coordinator correctly for every ordinary write, even though the same ownership and lifecycle decisions recur in all callers.

**Non-finding:** A small interface implemented by several drivers hides distinct device-specific behavior. Its value is not measured by the length of any one forwarding method.

## 3. Organize around knowledge; contain decisions

**Attribution:** Ch. 5, “Information Hiding (and Leakage),” §§5.1–5.3, 5.6–5.9. Ch. 9, “Better Together Or Better Apart?,” introduction and §§9.1–9.2.

Identify the decision a module owns: format, representation, algorithm, protocol, policy, or invariant. Then look for other modules encoding that same decision.

- Would changing an internal choice require changes outside its owner?
- Do a reader and writer independently encode the same format?
- Is a returned data structure part of the domain contract, or the internal representation accidentally exposed?
- Do getters and setters disclose a decision despite private storage?
- Has a workflow been split into “read,” “process,” and “write” classes that still share the same knowledge?
- Does joining related operations remove intermediate state, repeated parsing, or caller coordination?

Execution order belongs somewhere, but does not by itself justify module boundaries. Separately timed stages can be good boundaries when they hide different knowledge. Across independently deployed components, a shared protocol may be unavoidable; judge whether the contract is explicit and each side's internal choices stay hidden, rather than demanding one class across services.

A new wrapper does not fix leakage if it merely exposes all the leaked details through another API. Likewise, moving constants into one file does not necessarily encapsulate the algorithm or invariant that uses them.

**Finding:** A reader and writer separately specify field ordering and escaping; changing the serialization rules requires coordinated edits. Recommend one format owner that hides both operations, not a shared bag of format constants.

**Non-finding:** A caller consumes a documented domain record, independent of storage layout. Returning structured data is not inherently information leakage.

## 4. Different layers should earn their boundaries

**Attribution:** Ch. 7, “Different Layer, Different Abstraction,” §§7.1–7.6.

Follow an operation through layers and name the abstraction each supplies.

- Does a layer translate caller intent into a different capability, or just reproduce the next layer's API?
- Does a wrapper's interface need to change whenever the wrapped interface changes, without insulating callers from anything?
- Do intermediate methods carry parameters they neither interpret nor own?
- Does a context object actually localize system-wide state, or conceal a grab-bag of unrelated dependencies?

Dispatch and different implementations of the same contract provide real value even with identical signatures. Adapters and decorators can also earn their boundaries; show the particular benefit and its cost. Do not ban wrappers or dependency injection by name.

For pass-through variables, consider ownership, object lifetime, and existing configuration structure before proposing a context. A global or service locator may shorten signatures while making dependencies less obvious. The book itself notes context objects' nonobvious dependencies and thread-safety risks (§7.5).

**Finding:** Three new layers repeat the same operations and low-level configuration while adding no policy, translation, or isolation; an operation's signature change must propagate through all three.

**Non-finding:** A dispatcher selects the correct implementation, or an adapter protects domain callers from a vendor's data model. Similar method signatures alone prove nothing.

## 5. Make mechanisms somewhat general; keep specialization in its owner

**Attribution:** Ch. 6, “General-Purpose Modules are Deeper,” §§6.1–6.8. Ch. 8, “Pull Complexity Downwards,” §§8.1–8.3. Ch. 9, §9.4.

Functionality should meet current needs, while the interface avoids unnecessarily encoding one particular use. Generality can simplify today's design even without later reuse.

- Can a small set of natural operations replace a growing family of feature-specific methods?
- Is the generalized API still convenient for actual current consumers?
- Are common operations forced into loops, conversions, splitting, or assembly in callers?
- Does a general mechanism branch on a particular screen, tenant, request kind, or product feature that belongs elsewhere?
- Is an extra state or special case actually necessary, or can a normal representation cover it? An empty range can sometimes eliminate a separate “no range” state.

Specialization need not always move upward. Product policy often belongs above a generic mechanism; device-specific behavior can live below a stable driver interface. What matters is that the general mechanism does not have to know each specialized use. Specialization for a different mechanism can legitimately live in the module that owns it—for example, text-specific undo actions in a text module, separate from generic history management (§6.7).

Pull complexity down when it is part of the module's capability, simplifies callers, and improves its interface. Do not pull unrelated product policy down merely to shorten calling code. For configuration, ask who can make the better decision: the implementation may know a sensible default, but the caller may own a deadline or business priority that cannot be inferred locally.

**Finding:** A text store adds one public method per UI gesture, or returns lines that force multiple callers to implement cross-line editing. A range-oriented text API can hide storage details without importing gesture policy.

**Non-finding:** A storage module exposes a caller-owned durability requirement. Hiding it behind a guessed default would hide something essential.

**Avoid:** A universal plugin framework for imagined future consumers; one giant function controlled by modes; a primitive API so minimal that every caller rebuilds the missing mechanism. All three can be “general” while increasing complexity.

## 6. Join or split based on independence, not length

**Attribution:** Ch. 9, introduction and §§9.3, 9.5–9.9.

Joining helps when pieces share knowledge, are normally used together in both directions, or can expose a simpler combined capability. Splitting helps when it creates independently understandable abstractions or separates unrelated responsibilities.

- Can a reader understand the extracted helper without the parent's implementation, and the parent without the helper's implementation?
- Has the split made callers pass intermediate state back and forth, or added lifecycle coordination?
- Does removing repetition centralize a genuine shared decision behind a simple interface?
- Would the proposed extraction need many locals, mode flags, callbacks, or exceptions that reproduce the original complexity?

Repeated policy is a strong signal; superficial textual similarity is not. A two-line repeated expression may cost less than a new interface. If two policies evolve independently, coupling them through a configurable helper can be worse than duplication.

Do not merge unrelated functionality into a god module. One-way use is insufficient: a block cache uses a hash table, but hash tables serve many other purposes and need not become part of that cache's abstraction. Likewise, do not split a coherent long function solely to satisfy a length threshold.

**Finding:** An extracted helper and its caller depend on the same mutable intermediate state and must be read together to understand the phase invariant. Join them or find a boundary that actually encapsulates the subtask.

**Non-finding:** A long, coherent routine has a simple contract and readable internal stages. Conversely, a short helper that encapsulates meaningful parsing rules can be valuable even with one caller.

## 7. Reduce exceptional obligations by changing the contract

**Attribution:** Ch. 10, “Define Errors Out Of Existence,” §§10.1–10.8, 10.10. Ch. 6, §6.8, covers eliminating special cases.

“Exception” includes uncommon control-flow conditions, not only language-level throws. Each exposed condition can create a handler obligation and another state callers must reason about.

- Does an API report failure even though the caller's desired postcondition already holds? “Ensure absent” may naturally succeed when already absent; “delete exactly one existing record” may not.
- Can the ordinary representation or contract handle an edge case without a separate branch in every caller?
- Can recovery be owned below the boundary, preserving the promised semantics? Can related failures be handled at one appropriate level?
- Are handlers spread out because the low-level API exposes distinctions its users cannot meaningfully act on?
- Are genuine failures swallowed or converted into empty results so that callers cannot tell whether the operation succeeded?

Defining an error away is a deliberate, valid API contract—not catching and ignoring it. Masking requires actually fulfilling the abstraction's promise, not merely returning a success-shaped value. Preserve information needed for recovery, diagnosis, or domain semantics. Aggregation must not erase distinctions the caller needs.

The book considers terminating on unrecoverable failures, but recoverability depends on the application's purpose. Do not recommend crashing a service, adding retries, clamping inputs, or hiding data loss simply to remove handlers. Respect the actual contract and deployment constraints.

**Finding:** Every caller handles an “already absent” error by treating it as success under an explicitly ensure-absent contract. Centralize that semantic rule at the owner.

**Finding:** A transport catches all communication failures and returns success, hiding delivery failure needed by the application—a false abstraction, not successful complexity reduction.

**Non-finding:** A domain operation reports a rejected transaction because its caller must choose whether to compensate. The error carries essential information.

## 8. Make the important information obvious

**Attribution:** Chs. 12–13, “Why Write Comments?” and “Comments Should Describe Things that Aren't Obvious from the Code.” Ch. 14, “Choosing Names.” Ch. 15, “Write The Comments First.” Ch. 17, “Consistency.” Ch. 18, “Code Should be Obvious.”

Names and comments are part of an abstraction, not decorative cleanup.

- Can a name create a precise, consistent image of the concept, including useful units, scope, or role?
- Does the contract document information not apparent from signatures: boundaries, ownership, side effects, invariants, error semantics, and required ordering?
- Are interface comments about the capability and obligations, separate from irrelevant implementation detail?
- Do implementation comments explain purpose, rationale, or invariants rather than narrating syntax?
- Does familiar-looking code behave as the established convention predicts?
- Does an event, callback, or mutable state dependency require knowledge that is difficult to discover at its point of use?

Do not replace missing contracts with “self-documenting code,” and do not prescribe a comment on every statement. Show which essential fact is missing and why the code or type system does not communicate it. A precise type can carry some information; prose can capture behavior and rationale that the type does not.

Hard-to-name or hard-to-describe concepts can reveal mixed responsibilities. Try describing the contract simply and completely. If a short explanation works, missing documentation may be the right fix; if explanation requires exposing a web of internals, reconsider the boundary. Do not write a lengthy comment to camouflage poor decomposition.

Consistency matters when similar concepts behave similarly. Forcing dissimilar concepts into the same convention creates misleading expectations. Event-driven or asynchronous code is not inherently a finding; expose its real dependencies and important invariants.

**Finding:** A new `timeout` crosses layers without indicating whether it is a duration or deadline, and its units/clock are only recoverable by reading downstream arithmetic. Identify the plausible misuse and make the contract precise.

**Non-finding:** A conventional short loop index or a new domain-specific name accurately identifies a distinct concept. Longer names and more comments are not inherently better.

## 9. Review evolution and justified tradeoffs

**Attribution:** Ch. 11, “Design it Twice.” Ch. 16, “Modifying Existing Code.” Ch. 19, “Software Trends.” Ch. 20, “Designing for Performance.” Ch. 21, “Decide What Matters.”

Review the result and its effect on future work, not an imagined development history.

- Does a change repair the owning abstraction, or add another exception around a boundary that no longer fits?
- Are contracts and design explanations updated where future maintainers will see them? Is crucial rationale stranded only in a commit message?
- Is a cross-module decision documented at one discoverable home, with references where needed, rather than copied inconsistently?
- Does a pattern, inheritance hierarchy, getter/setter API, or test seam provide demonstrated leverage against complexity, rather than being justified by fashion?
- For performance-motivated complexity, is there evidence that the constraint matters and the added mechanism helps? Could a simpler critical path serve both speed and clarity?

Clean design and performance are often compatible. Avoid gratuitous expensive operations during ordinary design; require evidence for more intrusive optimization tradeoffs. Missing measurements in the available review material do not prove that none exist. Do not make unsupported speed claims or reflexively demand removal of a required optimization.

Consider alternative designs yourself. Evaluate caller ease, generality, simplicity, and performance; do not demand proof that the author tried two designs or wrote comments first. Tests can make redesign safer, but passing tests do not establish a deep abstraction. Process ideology is outside this lens.

## Red flags: search prompts, never an automatic score

These are the named red flags in the supplied text. Multiple flags can identify one underlying defect; report the defect once.

| Red flag                                            | Investigate                                                     | Source       |
| --------------------------------------------------- | --------------------------------------------------------------- | ------------ |
| Shallow Module                                      | Interface obligations nearly equal the capability hidden        | Ch. 4 §4.5   |
| Information Leakage                                 | One design decision encoded in multiple modules                 | Ch. 5 §5.2   |
| Temporal Decomposition                              | Execution phases split knowledge that needs one owner           | Ch. 5 §5.3   |
| Overexposure                                        | Common use requires learning uncommon features                  | Ch. 5 §5.7   |
| Pass-Through Method                                 | Added boundary supplies no distinct capability                  | Ch. 7 §7.1   |
| Repetition                                          | Repeated code represents a shared decision without an owner     | Ch. 9 §9.3   |
| Special-General Mixture                             | A mechanism knows a particular use of that mechanism            | Ch. 9 §9.4   |
| Conjoined Methods                                   | Separated implementations must be read together                 | Ch. 9 §9.7   |
| Comment Repeats Code                                | Prose adds no missing information                               | Ch. 13 §13.2 |
| Implementation Documentation Contaminates Interface | Caller documentation exposes irrelevant internals               | Ch. 13 §13.5 |
| Vague Name                                          | A name permits materially different interpretations             | Ch. 14 §14.3 |
| Hard to Pick Name                                   | The concept may lack a coherent responsibility                  | Ch. 14 §14.3 |
| Hard to Describe                                    | A simple, complete contract cannot be stated                    | Ch. 15 §15.3 |
| Nonobvious Code                                     | Important information cannot be found from a reasonable reading | Ch. 18 §18.2 |
