# Ousterhout in Rust

This is a practical interpretation of the two summaries in John Ousterhout's *A Philosophy of Software Design* (2018). Numbers in parentheses are the book's **printed** pages. Use the principles to design; use the red flags to discover where a design may be making work for someone else.

## Fifteen principles

1. **Small costs accumulate** (11). Count the extra fact a caller must remember today. A handle that must be reset after every read looks cheap once. Every new caller inherits that obligation.
2. **Passing tests is not the whole job** (14). Before shipping a working branch, ask whether the next feature will need another copy of it. Fix the boundary that caused the branch when the fix belongs in the current change.
3. **Invest a little as you go** (15). Compare two signatures before publishing one. Clarify a misleading name when you change its owner. Don't wait for a rewrite or redesign unrelated subsystems.
4. **Seek depth** (22). A deep Rust module provides substantial behavior behind a contract that takes little effort to learn. Count the facts and steps a caller can forget, not the number of `pub fn` declarations.
5. **Make ordinary use ordinary** (27). If almost every reader needs buffering, provide it by default. Give the unusual caller a way to choose a different policy without requiring every caller to configure one.
6. **Prefer a simple contract to an easy implementation** (55, 71). If the module can enforce the required ordering, don't give clients three methods and a warning to call them in sequence. More private code can mean less work across the program.
7. **Generalize the operation, not the roadmap** (39). A byte buffer can remove a range. It need not know why the caller chose those bytes. Support today's functionality without implementing a framework for hypothetical consumers.
8. **Separate the mechanism from its particular use** (62). Let the editor decide what is selected; let the buffer remove the range. A future change to selection policy should not require editing the buffer.
9. **Make each layer change the abstraction** (45). A transport reads bytes. A decoder produces records. A service applies a record. A layer that just renames `read_bytes` has not bought the reader that change in vocabulary.
10. **Handle complexity where the knowledge lives** (55). If a decoder can retain an incomplete record, keep the fragments there. Don't force every connection loop to carry them. Conversely, don't put an application-specific retry policy in a general decoder.
11. **Choose semantics that remove false errors** (79). For `ensure_absent`, an already-absent entry meets the postcondition. An inability to inspect or remove the entry does not. Changing the contract is different from ignoring `Err`.
12. **Compare real alternatives** (91). Sketch two materially different APIs, such as borrowing a slice versus transferring an owned record. Trace the likely caller and the ownership cost. Don't compare two names for the same design and call it an alternative.
13. **Document missing knowledge** (101). If `offset` counts bytes from the start of a record rather than characters from the start of a stream, say so. A comment that says “set offset” communicates neither fact.
14. **Spend the writer's effort to save the reader's** (149). A named output with a field such as `bytes_consumed` can be clearer than a positional tuple containing several integers. Use the tuple when its meaning really is obvious.
15. **Grow the system one useful abstraction at a time** (154). Let a feature demonstrate the need. Once it does, design the whole contract that feature needs instead of scattering feature-specific branches among existing owners. This does not authorize speculative functionality.

## Fourteen red flags

Treat each question as a diagnostic. Identify the actual burden before proposing a change.

| Signal (printed page)                  | Question to ask about the Rust design                                                                                                                                                                |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Shallow module (25, 110)               | Does a caller have to learn almost as much to call the API as to implement its behavior? Would removing or combining it simplify the call site?                                                      |
| Information leakage (31)               | If the encoding changes, which other modules must change? Private fields do not help when consumers must reconstruct the encoding from accessors.                                                    |
| Temporal decomposition (32)            | Did we create `read`, `process`, and `write` modules because those events occur in order, even though two modules now need the same format knowledge?                                                |
| Overexposure (36)                      | Does a caller using the normal case have to choose buffer sizes, retry intervals, or low-level flags that the owner could safely choose?                                                             |
| Pass-through method (46)               | Does a forwarding method translate an abstraction, enforce a contract, or protect a boundary? If not, what does it buy? A deliberate extension wrapper is a different promise.                       |
| Repetition (62)                        | Are several sites making the same nontrivial decision? Move the decision to its owner. Don't extract identical punctuation or a trivial expression into a helper with five arguments.                |
| Special-general mixture (65)           | Would changing one application's policy require editing a supposedly reusable mechanism? Move the policy, not necessarily the underlying operation.                                                  |
| Conjoined methods (72)                 | Must a reader alternate between two private bodies to understand either one? Join them or give one a contract the reader can trust independently.                                                    |
| Comment repeats code (104)             | If the comment disappeared, what fact would the reader lose? Supply the missing unit, invariant, purpose, or reason instead of narrating the statement.                                              |
| Implementation in interface docs (114) | Does the caller need to know the channel or lock type to use the service correctly? Describe the delivery or blocking guarantee instead, unless the underlying type is deliberately part of the API. |
| Vague name (123)                       | Read `state.update(data)` aloud. Which state changes, what does the argument represent, and what happens? Name those things where the context doesn't already do so.                                 |
| Hard to name (125)                     | Are you choosing between vague names because a value represents two unrelated concepts? Reconsider the representation before reaching for a thesaurus.                                               |
| Hard to describe (131)                 | Can you write short, complete rustdoc without listing implementation branches? If not, simplify the contract rather than deleting necessary qualifications.                                          |
| Nonobvious code (148)                  | Could a reader predict what triggers this handler or what a returned boolean means? Improve the type, name, or flow first; document any remaining non-obvious fact.                                  |
