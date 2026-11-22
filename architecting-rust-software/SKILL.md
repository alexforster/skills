---
name: architecting-rust-software
description: Use when designing or implementing Rust software. Apply the author's preferences for module layout, impl blocks, type paths, naming, APIs, Tokio services, failure handling, tests, and crate selection. Use for new code and changes to existing Rust projects, not as a general code-review checklist.
---

# Architecting Rust software

Write Rust that tells the reader who owns each decision. At a call site, the reader should understand the operation without inspecting its implementation. Inside the implementation, the reader should recognize the origin of a type and the role of a value without chasing imports.

The capitalized terms MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY carry their RFC 2119 meanings, as clarified by RFC 8174. A SHOULD describes the normal choice, not an invitation to substitute the agent's usual style. A red flag is a question about the design, not a prohibition.

## Before choosing a design

1. Read the applicable project requirements and the code that will call the new API. Separate an explicit requirement from a pattern that happens to exist nearby.
2. Name the operation in the caller's words. Identify the type that should own its representation, invariants, and failure policy. Sketch the common call site and one important boundary case.
3. For a consequential decision, sketch a genuinely different design. Compare what callers must know, what a future change would touch, and what work the implementation must do. Don't implement both to make the comparison.
4. If a project requirement conflicts with these preferences, you MUST ask the user before choosing. You MUST also present an alternative when it is more tasteful than the requested or proposed design, or when equally tasteful options differ meaningfully. Show the competing signatures or call sites, explain the consequence, recommend one, and wait. Do not ask about routine choices these rules already settle. Do not broaden a focused change into an unsolicited migration.

## Give behavior a home

- `lib.rs` and `mod.rs` SHOULD be maps, not implementation files. Use them for module declarations, deliberate exports, documentation, and applicable attributes. For a directory of children, prefer `name/mod.rs`. A leaf can simply be `name.rs`.
- Keep implementation modules private unless their namespace helps the caller. Export only what a consumer needs. A glob re-export is fine when the leaf deliberately exposes exactly those names. Do not flatten a crate merely to shorten paths.
- Behavior SHOULD live in the owning type's `impl`. Use a method if the operation needs an instance. Use a private associated function if the operation belongs to the type but needs no receiver. Lack of `self` does not imply lack of an owner.
- If an operation has no obvious owner, reconsider the boundary. Entry points, tests, and rare independent leaf algorithms may stand alone. Do not collect unrelated functions in `utils.rs` or invent an empty `Utils` type to satisfy the spelling of the rule.
- Keep knowledge that changes together under one owner. Reading and writing a format may use different files without giving two modules independent definitions of that format. Do not extract a helper because a function crossed a line count. Extract when the helper's contract lets a reader skip its body.

## Make Rust read as prose

- Group imports in this order: standard library, third-party crates, then project-local imports (including sibling workspace crates). Separate adjacent groups with one blank line. Sort each group alphabetically by import path. Each `use` statement MUST name one path; never combine imports with braces.
- Import the namespace and qualify the type: `use std::path;` then `path::PathBuf`, or `tokio::task::JoinHandle`. Keep the shortest path that still tells the reader where the type comes from. Do not fill the top of a file with bare type imports by habit.
- Use ordinary prelude names directly. Import a trait when method syntax needs it. A trusted local facade such as `super::*` can supply the vocabulary within its own subtree. A function-local DSL glob can make a grammar readable without polluting the file. These are deliberate exceptions, not a general preference for external globs.
- Read the expression as a sentence. `configuration_sender.send(configuration)` names the actor, action, and object. `tx.send(cfg)` makes the reader translate. A short word like `peer` needs no expansion. A vague name SHOULD be exceptional even in a short scope. Name booleans so the true state reads as a fact.
- Use a guard or `let … else` to finish a terminating case. Use `match` when the alternatives tell the story. Choose a loop or an iterator chain for ease of reading, not to minimize lines. Order declarations so related behavior is easy to find; there is no universal public-before-private order.
- When choosing a new project's formatting, prefer `max_width = 120` and `use_small_heuristics = "Max"`. Don't change a project's formatter, edition, or minimum Rust version to imitate an older codebase.

## Design the contract before the machinery

Start with a concrete type and a direct constructor. Introduce a trait for a meaningful capability, not merely to mock a collaborator. Use a builder or options type when it improves actual call sites. Don't manufacture defaults for required dependencies.

An input can accept `impl Trait` without making the owner generic. Name a type parameter when it relates two types or appears in several positions. Choose storage independently: weigh allocation, dispatch, and lifetime constraints against the burden of passing a generic parameter through other types.

A domain handle can hide a channel. An **extension wrapper** can intentionally expose its backing type through `Deref`. Decide which promise you mean. Add `DerefMut` only if the underlying mutations preserve the wrapper's contract. Don't generate pass-through methods just to avoid dereferencing.

Write the caller-facing rustdoc while choosing a nontrivial signature. State what the caller cannot infer: units, range endpoints, ownership, ordering, failure conditions, and what is true after success, as applicable. Keep implementation mechanics out of that contract. Inside the implementation, explain the non-obvious reason or invariant rather than translating the next line into English.

## Services, failures, tests, and dependencies

- **Services:** For a named, long-running Tokio service, construct and wire it first. Acquire any client handles before calling `into_task(self) -> tokio::task::JoinHandle<Self>`. A private `async fn task(mut self) -> Self` owns the loop. The composition point keeps the join handle. See [the service template](references/services.md) for shutdown, child work, and graceful degradation.
- **Failure:** Reuse a standard error type when it describes the contract: `io::Result` for a purely I/O failure. Define error variants when the caller can meaningfully act differently. Preserve absence separately from invalid input when they have different meanings. See [error semantics and test boundaries](references/rust-choices.md#failure-is-part-of-the-contract).
- **Panic:** A panic means the program is in an unrecoverable state and cannot safely continue. Use `unwrap()` for a fact the nearby code proves. Use `expect("…")` to identify something believed impossible and explain the invariant. Use `unreachable!()` for an impossible branch. Never reclassify a routine external failure as impossible to avoid deciding how to handle it.
- **Tests:** Unit-test mechanisms with a high risk of subtle bugs: parsers, algorithms, nontrivial data structures, and tricky state transitions. Exercise plumbing through integration or black-box behavior. Don't introduce a trait and mocks to verify that one component forwards a value to another. Pin exact bytes or diagnostics only when the contract promises them.
- **Crates:** Check `std` and existing dependencies first. For a recognizable, nontrivial need, search crates.io early. Finding a candidate is not permission to adopt it. Inspect evidence, including the relevant source, before choosing. See [choosing a dependency](references/rust-choices.md#choosing-a-dependency).
- **Cargo.toml:** Every dependency MUST use an inline table, even when `version` is its only field. End each declaration with a same-line comment giving the dependency's SPDX license identifier or full license expression: `combine = { version = "^4" } # MIT`. Verify the license for the version used; do not guess.

## References

- [Rust choices at the call site](references/rust-choices.md): paired examples of layout, ownership, imports, errors, tests, and dependency decisions.
- [A long-running Tokio service](references/services.md): a complete example and the limits of the pattern.
- [Ousterhout in Rust](references/design.md): all 15 design principles and 14 red flags from the 2018 summaries, expressed as Rust decisions. Use this when comparing boundaries or when a name or contract is hard to explain.
