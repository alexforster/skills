# Rust choices at the call site

## Show the module map; keep the implementation behind it

Avoid a `lib.rs` that contains the wire parser, the service loop, and their public types. A caller who imports one type shouldn't have to learn that layout. The following is an example file map; the small module contains one externally useful type.

```text
src/lib.rs              mod journal; pub use journal::Journal;
src/journal/mod.rs      mod journal_file; mod record; pub use journal_file::Journal;
src/journal/journal_file.rs    defines Journal and its impl
src/journal/record.rs          defines the private record representation
```

If `journal_file` deliberately exports only the intended API, `pub use journal_file::*;` is also fine. Give peer modules only the access they need, for example `pub(super)` on an internal record. If `journal` itself is useful vocabulary for consumers, a public module can be clearer than flattening everything into the crate root.

## Put the operation on its owner

Compare `utils::validate_name(&archive, name)` with `archive.accepts_name(name)`. The latter tells the reader whose policy decides. If the policy only depends on the name, `Archive::accepts_name(name)` needs no receiver. Don't borrow the whole archive to suggest a connection that the implementation doesn't need.

Use a private associated function when it hides a decision that belongs to the type. In this example, an archive stores named entries. Callers ask about membership without learning how the archive compares names:

```rust
use std::collections;

#[derive(Default)]
pub struct Archive {
    entries: collections::BTreeSet<String>,
}

impl Archive {
    /// Records a name. ASCII letter case does not distinguish entries.
    pub fn insert(&mut self, name: &str) {
        self.entries.insert(Self::comparison_name(name));
    }

    /// Reports whether this name has been recorded, ignoring ASCII letter case.
    pub fn contains(&self, name: &str) -> bool {
        self.entries.contains(&Self::comparison_name(name))
    }

    fn comparison_name(name: &str) -> String {
        name.to_ascii_lowercase()
    }
}

let mut archive = Archive::default();
archive.insert("March");
assert!(archive.contains("MARCH"));
assert!(!archive.contains("April"));
```

The helper is not valuable because it shortens either method. It keeps insertion and lookup from acquiring different comparison rules. If the representation changes to avoid allocating a lookup key, neither caller changes.

## Preserve provenance; name the job

Compare `fn open(p: PathBuf) -> Result<File, Error>` with `fn open(configuration_path: &path::Path) -> io::Result<fs::File>`. The second tells the caller which file to supply, borrows a path rather than demanding ownership, and identifies the failure family.

Here is the import block for a module that also uses a Tokio channel and a project-local catalog. Keep one path per statement, even when several paths share a prefix. Alphabetize within each group; omit groups with no imports.

```rust,ignore
use std::fs;
use std::io;
use std::path;

use tokio::sync::watch;

use crate::catalog;
```

Prefer `configuration_sender.send(configuration)` over `tx.send(cfg)`. By contrast, `peer.send(message)` needs no expansion if there is only one meaning of peer here. When a private type alias replaces a long channel type, its name should tell the reader something useful about the role, not just hide the spelling.

## Choose the contract, then the abstraction

- **Construction:** Prefer `Archive::open(path)` to a builder that collects one required path. If three adjacent arguments are all durations with different meanings, compare a named policy value against positional arguments at a real call site.
- **Polymorphism:** A codec that accepts `impl io::Read` can read a file or an in-memory byte slice. That capability has value without defining an application-specific trait. Decide separately whether to store a concrete reader, a generic reader, or a boxed reader. Don't box an input the operation can consume immediately.
- **Extension versus isolation:** If a handle means “receiver plus a domain convenience,” `Deref` can be the honest interface. If it promises “deliver the latest configuration,” exposing the receiver also commits the API to its channel semantics. Decide deliberately. Do not expose mutable access if replacing or mutating the backing value could violate an invariant.

## Failure is part of the contract

Compare `parse_limit(value).ok()` with a result that distinguishes an unset limit from an invalid one. The former makes a malformed setting silently indistinguishable from a missing setting. The latter lets the caller supply a default **only** when the setting is absent:

```rust
use std::num;

#[derive(Debug, PartialEq)]
struct Limit(u32);

impl Limit {
    /// An absent setting means no limit was specified. A present value must be an unsigned integer.
    fn parse(setting: Option<&str>) -> Result<Option<Self>, num::ParseIntError> {
        setting.map(|value| value.parse().map(Self)).transpose()
    }
}

assert_eq!(Limit::parse(None), Ok(None));
assert_eq!(Limit::parse(Some("0")), Ok(Some(Limit(0))));
assert!(Limit::parse(Some("off")).is_err());
```

The standard parse error is enough here. Add a domain error when the extra distinction changes what a caller can do. At a process boundary, attach the setting name or path where that context is known. Don't discard the underlying cause just to create a friendlier sentence.

An impossible outcome and an invalid input are different. If private state records that initialization completed, but the required resource is missing, an `expect` message should name the invariant, not say “failed to get resource.” If a client supplied a malformed value, return or handle that error. If the owner can no longer safely operate, do not continue as though the failure were local.

## Pick the test boundary that can fail meaningfully

| Decision under test | Evidence worth keeping |
|---|---|
| A decoder retains an incomplete record | Feed a split inside the header, then the remainder. Assert that exactly one complete record appears with the expected contents. |
| An ordered index removes an entry with two children | Assert the surviving contents and order, not which internal rotation happened. |
| A service applies a configuration change | Update it through the supported external boundary and observe the changed behavior. A mock assertion that a publisher called `send` does not prove the consumer changed behavior. |
| A diagnostic reports where parsing stopped | Pin the offset if the API promises it. Don't pin the derived `Debug` spelling merely because it is easy to snapshot. |

The assertions in the small Rust samples illustrate contracts. They are not a request to add permanent unit tests for every straightforward helper.

## Choosing a dependency

1. Define the needed behavior, including awkward inputs and supported platforms. Check `std` and the dependencies the project already uses. A transitive dependency is not automatically an appropriate public API or direct dependency.
2. When the need is nontrivial, search crates.io before building a substitute. Compare credible candidates with the local implementation you would otherwise own.
3. Inspect the actual published version's implementation on the relevant path and its tests. Check its public contract, feature flags, transitive cost, supported Rust versions, license, and known advisories. A crate with a tiny call site may bring a substantial dependency tree.
4. Check adoption and maintenance independently. Who uses it? Are relevant issues addressed? Does its release history make sense for its scope? Downloads can suggest adoption; neither downloads nor a recent release prove code quality. A mature, finished crate may change infrequently.
5. Prefer local code for a small contract whose correctness and maintenance are easy to establish. Prefer a vetted crate over reimplementing a deceptively hard mechanism. A few lines that delegate cryptography are not equivalent to a few lines implementing it.
6. If adoption and local implementation are equally tasteful but materially different, show the user the tradeoff. If registry or source access is unavailable, state which check you couldn't perform. Do not report a crate as vetted without evidence.

Once you choose a dependency, write its entry as an inline table. Keep additional fields inside the braces and the SPDX comment after the closing brace. If the crate offers a choice of licenses, preserve the full expression rather than picking one identifier for the comment.

```toml
[dependencies]
combine = { version = "^4" } # MIT
futures-util = { version = "^0.3", features = ["sink"] } # MIT OR Apache-2.0
```

The same spelling applies in build, dev, target-specific, and workspace dependency tables. For a path dependency or an inherited workspace dependency, keep the actual `path` or `workspace` field; don't invent a version. Check the dependency's package metadata for its license. If it doesn't establish an SPDX expression, ask rather than guessing.
