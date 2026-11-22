This gpt_brief..md file defines the content and verification contract for a Rust UML-style ownership map: a compact, source-grounded account of where types, traits, implementations, and capabilities belong in actual files and folders.

# Rust UML — Type Ownership and File Silo Brief

## Purpose

A Rust UML brief is an **implementation-ownership artifact**, not a generated call graph, a proposed class hierarchy, or a demand to refactor files. It answers three questions:

1. **What exists?** The important types, traits, and implementation capabilities, and their relationships.
2. **Where does it live?** The *actual* module path and physical source file that defines or extends each item.
3. **Does it belong there?** Whether the file/folder arrangement reflects one practical domain authority, or hides a real ownership mismatch.

The result is a concise, UML-like diagram and file map useful for code review, architectural QA, orientation, and a bounded file-siloing pass. It must be rebuildable from source without the conversation that prompted it.

Examples are illustrative Rust, not a prescribed project layout. The ownership rules apply beyond Rust; Rust syntax and module semantics define how this particular brief records the result.

**Relationship to companion files:** `gpt_brief.repo.md` owns the wider AI-facing in-repo implementation brief; `gpt_style.pseudocode.md` owns method-level algorithm outlines; `gpt_style.rust.md` owns Rust code/comment/region syntax. This brief owns the **type-to-file responsibility map and its QA contract**. Do not copy the other files' rules into this one.

---

## The Model: Three Distinct Layers

| Layer | Represent | Never conflate with |
|---|---|---|
| **Physical ownership** | Actual crates, directories, `.rs` files, `mod` declarations and re-exports | A desirable but nonexistent folder layout |
| **Type relationships** | Structs/enums, traits, `impl Trait for Type`, composition, type-specific capabilities | C#/Java inheritance, call order, or abstract classes invented for the picture |
| **Responsibility** | The concept or invariant each file/type owns, and the collaborators that consume it | A second source of truth just because another module uses the concept |

The **filesystem is the UML package hierarchy**. The **type is the UML class-like box**. A trait is a contract, not an automatically separate `interfaces/` layer. An `impl` in another file is a capability of the *same* type, not a derived class and not a second owner.

A conceptual cluster may clarify a large flat directory, but must be explicitly labelled `conceptual — not a real folder`. Never draw an invented `owner/`, `pages/`, or `interfaces/` package as if it already exists.

### Relationship notation

Use a short legend; choose labels a reader can verify in Rust:

| Notation | Meaning | Rust evidence |
|---|---|---|
| `Concrete : Trait` | Concrete type implements trait | `impl Trait for Concrete` |
| `Owner + capability` | Same type extended by inherent `impl Owner` in another module | `impl Owner` and its defining type/file |
| `Owner has Component` | Composition / stored dependency | Struct fields, owned handles, or documented lifecycle responsibility |
| `Consumer → Contract` | Uses a capability provided by another owner | Imports, type signatures, or relevant API calls |
| `A → B` labelled *produces/reads/coordinates* | Specific responsibility relation, if needed | Source-backed data or authority transfer |

In diagram syntax, a trait implementation may use UML realization (`..|>`). **Do not** use an inheritance arrow (`--|>`) to imply a Rust struct derives from another struct. Likewise, `Deref`, `From`, a generic bound, a shared field, and a wrapper are **not** inheritance. Describe each by its actual role.

Where `:` could ambiguously mean either trait implementation or an inherent extension, spell out `implements` versus `impl` in the box or legend. One notation is not a licence to misstate semantics.

---

## Source Reconstruction

Before drawing, inspect the actual target revision, not only existing architecture docs or a previous diagram.

1. **Find the roots.** Resolve relevant crate/package roots, `lib.rs` / `main.rs`, `mod` declarations, inline modules, `#[path]`, and any material re-exports. Record a branch/commit only if it can be verified.
2. **Inventory the owners.** For each relevant file, identify meaningful `struct`, `enum`, `trait`, `type`, `impl Type`, and `impl Trait for Type` declarations. Recognise test-only and feature-gated types as such.
3. **Follow split implementations.** A type may be declared in one file and gain inherent `impl` blocks in child/sibling modules. Keep one conceptual class box, with the extension files listed as capabilities.
4. **Establish the true authority.** Identify who owns state, who decides policy, who performs side effects, and who only adapts, observes, renders, or tests it. Dependencies are not evidence of shared ownership.
5. **Compare placement with responsibility.** Determine whether the current file contains cohesive collaborators or unrelated concerns deposited there for convenience.
6. **Reconcile docs against source.** Mark a stale architecture name or historical path as drift rather than treating it as live. Identify unknown or macro-generated relationships as unverified when necessary.

Do not declare an exhaustive type census unless the whole relevant scope was inspected. A bounded map of *important* entities is preferable to a dense diagram of trivial markers, getters, tests, and generics that answer no ownership question.

### Ownership questions worth tracing

- Which module would be responsible for changing this invariant?
- Does the type's defining file house its natural implementation and contract?
- If an `impl` is elsewhere, is that because it adds a coherent capability for the same type?
- Does the trait sit with the domain contract, while adapters implement it at their own boundary?
- Is a second cache merely a derived read model, or a competing decision authority?
- Are resource, entity hierarchy, installation, scheduler, and diagnostic duties separated by what they own?
- Are public exports and `pub(crate)` / `pub(super)` boundaries consistent with those responsibilities?

---

## Required Output

### 1. Scope and legend

Name the subsystem, source revision (or `not verified`), and whether the result is an **inventory**, a **siloing QA**, or an **intended move plan**. Define the diagram's relation notation once. An inventory is not permission to reorganise code.

### 2. Real-path ownership tree

Show actual folders and files, with the important definition or extension beside each path. Make it clear where the type is defined versus where an `impl` extends it.

Illustrative only:

```text
src/
  domain/
    world.rs         WorldRead (trait): terrain/version facts the domain requires
    owner.rs         Owner<S> (definition): sources, units, reconciliation
    owner/
      live.rs        impl Owner<S> (+ incremental tick)
      census.rs      impl Owner<S> (+ diagnostic census)
    scheduler.rs     Scheduler: queue, urgency, capacity
  adapters/
    snapshot.rs      SnapshotWorld : WorldRead (implements)
```

Do not fabricate the directory for a concise diagram. If the actual code is flat, show flat paths first; list optional **conceptual groups** separately. One file can house several strongly related types, and one type can have several `impl` files without becoming several owners.

### 3. UML-style relationships

Provide a compact type/capability view that corresponds to the path tree. Emphasise relations that explain ownership or placement rather than every call or import.

```text
Owner<S> [domain/owner.rs]
  + incremental tick [domain/owner/live.rs: impl Owner<S>]
  + census           [domain/owner/census.rs: impl Owner<S>]
  has SourceSlot<S> / UnitSlot

WorldRead [domain/world.rs: trait]
  ^ implements: SnapshotWorld [adapters/snapshot.rs]

Scheduler [domain/scheduler.rs]
  consumes Owner's queued subjects; does not decide demand
```

If a rendered UML diagram is useful, group nodes by **real folder paths**. Use interface-realization arrows for `impl Trait for Type`, composition for genuinely owned parts, and labelled capability edges for split inherent `impl`s. Any proposed group must be visually distinguished from real physical packages. Avoid inheritance arrows between Rust structs, fictitious base classes, and arrows for ordinary method calls.

### 4. Responsibility and siloing verdict

When auditing, produce a short matrix:

| Actual file / type | Owns | Verdict | Evidence / action |
|---|---|---|---|
| `path/file.rs` / `Type` | One practical responsibility | **Keep** | Why its methods and collaborators belong together |
| `path/other.rs` / helpers | A responsibility owned by another existing file | **Move candidate** | Exact declarations, actual callers, simpler destination |
| `path/large.rs` | Several tightly coupled capabilities | **Keep, improve navigation** | Why splitting offers no real boundary |
| `path/flat/` | Several distinguishable domains as siblings | **Defer** | Real paths remain flat; proposed grouping is conceptual |

Use **Keep / Small move candidate / Navigation only / Investigate / Defer**. Reserve **Confirmed mis-silo** for code whose existing file demonstrably obscures or violates its practical owner. Do not deduct quality or propose moves merely for line count or the existence of multiple `impl` blocks.

### 5. Bounded move map, only when requested or justified

For each proposed move, name *declaration(s)*, **from → to**, current callers, visibility/re-export adjustments, and the expected gain. State whether the change is mechanical or alters authority/semantics. Leave untouched items explicitly marked **accepted** or **deferred**.

Keep source moves separate from policy changes. An audit does not silently create new `mod` trees, traits, or wrappers. If a large refolder is worth doing, treat it as its own change with dependency, compile, and runtime validation; do not bury it inside a readability pass.

---

## Siloing Decision Rules

**Keep together when:** definitions and helpers share one invariant; a type's methods form one practical domain capability; a `Plugin` / `Material` implementation belongs with its owning type; colocated tests exercise that contract; or splitting would require more imports, visibility widening, cross-module coupling, or explanation than it removes.

**Separate when:** a helper naturally belongs beside the resource or entity hierarchy it manages; a file mixes unrelated ownership (for example, *work scheduling* and *page entity creation*); a trait is stranded with one adapter instead of the domain that needs the contract; or independent policy and side-effect authorities are accidentally competing.

**Accept cross-file `impl` when:** the defining type retains one owner and the second file adds a coherent role such as incremental updates, diagnostics, publication stages, or a read-only view. Record `Type + capability` rather than inventing a subclass.

**Accept large files when:** methods are domain-cohesive and the executable flow is legible. Improve the in-method pseudocode outline or the real module/header map before recommending extraction. A file does not fail a silo audit by line count.

**Don't manufacture abstraction.** Traits need a real substitution/test/boundary role; a one-variant enum can remain if removing it buys little; private/internal types need not become public to make a diagram convenient. A Rust module, not a comment-only `region`, is the genuine file-level ownership construct.

**Respect visibility and testing cost.** Check `pub`, `pub(crate)`, `pub(super)`, imports, macros, module cycles, feature gates, and colocated tests before accepting a move as mechanical. A pleasing folder tree does not outweigh churn across a proven runtime or public API.

---

## Pseudocode Relationship

The UML brief identifies **which class-like type owns a behaviour and in which file its implementation lives**. A separate implementation pseudocode map explains **how the behaviour flows**.

For large orchestration methods, a useful row is:

```text
Owner::reconcile()       [domain/owner.rs]       decide wanted/owed/retained
Owner::tick()            [domain/owner/live.rs]  process changed state incrementally
Presenter::present()     [ui/presenter.rs]       validate → prepare → activate → commit
```

That is the ownership outline, not a nested call tree or a transcript of every helper. Use `gpt_style.pseudocode.md` / `gpt_style.rust.md` if the task also requests step-level fold labels. A method whose calls already read as the algorithm needs no ceremonial regions; the type/file diagram does not prescribe refactoring it.

---

## Proof and Maintenance

An acceptable Rust UML brief passes these checks:

- **Path truth:** every path claimed *actual* exists at the inspected revision; proposed paths are unambiguously labelled.
- **Definition truth:** every type/trait/`impl` relationship is supported by a declaration or an explicit qualified exception (e.g., macro-generated or conditional).
- **Single authority:** split `impl`s, adapters, projections, read models, and diagnostics do not appear as competing owners of one decision.
- **Scope truth:** coverage is described as complete only for the scanned scope; unchanged legacy names are not assumed dead.
- **Actionability:** each alleged mis-silo has an exact owner, destination, caller surface, and why the move pays for itself.
- **Concision:** the diagram communicates hierarchy and responsibility at a glance; it is not a full call graph or a dump of all symbols.
- **Drift:** when a type moves, a trait changes ownership, a module is renamed, or a real folder is restructured, update the file tree and associated relationship diagram together. Pure method-body changes normally do not require a UML rewrite.

When evidence supports the existing placement, explicitly say **Keep**. A correct review can recommend **no file moves**.

---

## What This Brief Does Not Own

- The feature's user-facing loop or long-lived gameplay rules (`gpt_brief.feature.md`).
- The method-by-method applied pseudocode document (`gpt_brief.repo.md` and `gpt_style.pseudocode.md`).
- Rust formatting, region marker spelling, or test naming (`gpt_style.rust.md`).
- Local workspace layout, build commands, or project-specific paths (environment files).
- Branch execution instructions, CI outcomes, task status, or one-off architecture decisions (task briefs).

It owns the durable contract for **representing and judging real Rust type, trait, `impl`, and file/folder relationships**.
