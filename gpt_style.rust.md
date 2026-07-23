This gpt_style..md file describes the expected response style, shaping how the assistant writes, structures, and presents Rust output without changing the task itself.

# Rust Style

## Purpose
Write Rust that reads like the applied-pseudocode philosophy in `gpt_style.pseudocode.md`, in the surface conventions Rust actually affords. This file owns the Rust **surface**: what rustfmt takes away, what the type system gives back, the three comment forms, the doc-header shape, and where the step outline lives.

Examples are Rust; the rules are Rust-general. Nothing here is Bevy- or project-specific — engine structure, crate layout, and tooling belong in the loaded `gpt_env..md`.

---

## Relationship to gpt_style.pseudocode.md

`gpt_style.pseudocode.md` owns the language-general layer and stays the master. It applies here unchanged except where noted below:

- check → decide → do → return
- methods as sentence fragments; names carry meaning
- unit/range suffix grammar
- vertical `||` / `&&` chains, one condition per line
- collapsed-code region labels (§11) — the label discipline ports verbatim
- `// Intent:` / `// Objective:` on load-bearing comments; untagged restating comments deleted
- no magic numbers in logic
- a region wraps 5+ meaningful lines

Two sections are **superseded in Rust** and one is **extended**. Those are the whole point of this file.

---

## Superseded: layout rules rustfmt owns

`rustfmt` is the ecosystem default, runs in CI, and cannot be configured to produce the pseudocode file's vertical layout. It wins. The following §7 rules **do not apply to Rust**:

- comma-led argument lists and initializers with the closing delimiter on its own line
- aligned multi-line ternaries led by `?` and `:`
- hand-aligned operand columns

Write it plainly and let `cargo fmt` decide:

```rust
// rustfmt's shape — do not fight it
let tint = Vec4::new(
    (red * brightness).clamp(0.0, 1.0),
    (green * brightness).clamp(0.0, 1.0),
    (blue * brightness).clamp(0.0, 1.0),
    1.0,
);
```

**What survives intact:** vertical boolean chains (rustfmt preserves one-condition-per-line breaks), the `|` vs `||` accumulate rule with its `// Intent:` tag, naming, suffix grammar, and every semantic rule in the file. Only the visual layout is conceded.

**Naming case is likewise a surface concession.** `snake_case` functions, `CamelCase` types, `SCREAMING_CASE` consts. The rule — methods read as sentence fragments — is unchanged: `try_add_from_held_slot`, `can_accept_from`, `should_discard_leftovers`.

---

## Extended: the suffix grammar wants to be types

The pseudocode file encodes units and ranges in name suffixes because most languages offer nothing better. Rust does. A newtype costs one line, compiles to nothing, and makes the compiler enforce what the suffix could only suggest:

```rust
pub struct Moisture01(f32);   // 0..=1 by construction
pub struct BrownsQty(u32);
```

Reach for a newtype when a value crosses an API boundary, when two same-typed values could be swapped at a call site, or when the range is an invariant worth protecting. Keep the plain primitive with a suffixed name for locals and private fields — a newtype per local is noise.

The suffix grammar still governs the *name* either way: `01`, `Qty`, `C`, `PerHour` / `PerDay` / `Days`, `Threshold` / `Tolerance`.

---

## Guards are the native idiom

"check → decide → do → return" is more natural in Rust than in most languages. Prefer the guard forms over nesting:

```rust
let Some(circumference) = self.circumference_x else {
    return to - from;
};
```

`?` is a guard. `let … else` is a guard. `match` replaces a boolean chain when the branches are over a value's shape rather than over independent reasons — a vertical `||` chain is still correct when each line is a distinct reason to bail.

---

## The three comment forms

Rust has three, and only one of them is a comment in the `gpt_style.pseudocode.md` §8 sense. Applying §8 to all three strips the documentation.

| Form | What it is | Governed by |
|---|---|---|
| `//` | ordinary comment | §8 unchanged — sparing, `// Intent:` tagged, restating comments deleted |
| `///` | documents the **next** item; rendered by `cargo doc` | API contract. Says what it is and how to use it |
| `//!` | documents the **enclosing** item; written at the top of a file | The module's rationale. Says why it exists and why it is shaped this way |

`//!` is an inner attribute and must precede every item in the file — a `use` above it is a compile error, not a lint.

Intra-doc links (`[`RingSpace`]`) resolve against real paths and warn when broken. Treat a rotted link as a doc that has drifted from the code.

---

## The module header

`//!` is the home for content that has nowhere else to live: the rule a later reader must not break, and the reasoning that stops a settled decision being re-litigated. Five beats, each omitted when there is nothing to say:

```rust
//! <What this is> — <milestone / epic anchor>
//!
//! <Why it exists — the problem, only when non-obvious>
//!
//! **The rule: <the invariant a later reader must not break>.**
//!
//! # Why it is shaped this way
//! <decisions, with dead ends named so they are not re-tried>
//!
//! <known limits, stated rather than left to be discovered>
```

A module whose purpose is obvious gets one line. The header earns length only from rules and dead ends — the two things a reader cannot recover from the code itself.

**Name dead ends explicitly.** "X was tried and failed because Y" is the highest-value sentence in a header, because without it the next session re-tries X.

**State limits rather than leaving them to be discovered.** A known-and-accepted limitation reads as engineering; the same limitation discovered later reads as a bug.

---

## The step outline

The fold map is the pseudocode layer (`gpt_style.pseudocode.md` §13), and Rust splits it across two mechanisms.

**File-level grouping is `mod`, not a comment.** Where another language reaches for a `#region Persistence`, Rust has a real construct: it folds, it appears in `cargo doc` as a navigable section, it enforces visibility, and it is greppable as a path. Prefer a module over a region marker for concern grouping.

**Method-local steps stay comment markers**, since no construct groups statements inside a function:

```rust
// region: batches = min(add_qty, stack/input, room/output); if !batches return
...
// endregion
```

The §11 label discipline applies unchanged: collapsed-code labels, coined meaning-names (`never_processed`), slash-grouped same-meaning siblings, pseudocode truthiness. Labels are loose on purpose; precise edge cases go in `// Intent:`.

**A block expression is often better than a labelled region** when a step produces one value — it folds natively and makes the step's output explicit:

```rust
let accepted_quantity = {
    ...
};
```

**Extraction is cheaper in Rust than the pseudocode file assumes.** A small private function inlines to nothing, so §3's bar for "does the name add value" is the only cost — there is no runtime argument against splitting.

---

## Doc skeletons

A module whose shape is not obvious from its signatures carries a step outline in its `//!` header, fenced as `text` so rustdoc does not try to run it:

````rust
//! ```text
//! blend_dirty_tiles()
//!   region: drain dirty set, dedupe to unique coords
//!   region: resolve each 3x3 neighbourhood
//!   region: index = adjacency_bits(neighbourhood) -> patch tilemap
//!
//! adjacency_bits()
//!   — marching-squares bits; friendly-terrain equivalence, not identity
//! ```
````

Two shapes, deliberately distinct: `region:` for a step outline, `—` for a one-line note on a function that needs no outline.

**The 5-line floor applies to docs as well as code.** A module of short, self-naming functions gets prose and no fence — a skeleton that says less than the signatures is worse than nothing. Do not skeleton trivial functions; a trailing prose line listing a family of small helpers beats five more entries.

---

## Tests

**Unit tests colocate** in the same file, at the bottom:

```rust
#[cfg(test)]
mod tests {
    use super::*;
```

**Test names are sentences** stating the claim, not the method under test: `delta_is_correct_across_the_seam`, `reset_clears_the_window`, `zero_circumference_is_rejected`. The failure output should read as the broken claim.

**Test the invariant, not the implementation.** A test named for the rule survives a refactor; one named for the function does not.

**Guard against vacuous passes.** A test that can silently check nothing — because a collection was empty or a parse failed — needs a companion asserting the thing under test was actually present. A vacuous invariant still reads green, which is worse than no invariant.

---

## Doctests

A fenced block in `///` is compiled and executed by `cargo test`. Documentation that lies fails the build — the only place in the doc stack where drift is caught mechanically rather than by review.

**Doctests are examples first, tests second.** They render into public documentation, so a thorough test that is a confusing example is the wrong trade.

- **doctest** — the canonical usage a newcomer should copy, plus the one case that is surprising. Verified as a side effect.
- **unit test** — seam cases, panics, boundary conditions. Noise in rendered docs.

**A function earns a doctest when its contract is non-obvious.** Not by default — each one compiles, and a wall of trivial examples costs build time and reader attention both.

Useful fences: `no_run` (compile, do not execute), `should_panic`, and `compile_fail` — which proves a type-safety claim, so a newtype's invariant and its proof live in one artifact.

---

## What this file does not own

Crate layout, workspace structure, dependency boundaries, engine and ECS structure, tooling commands, and platform failure modes are environment concerns, owned by the loaded `gpt_env..md`.
