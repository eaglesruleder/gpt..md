# Structural Reassessment Metrics

## Purpose

Use this rubric to reassess a code domain after implementation or refactoring.

The score is a compact summary only. The evidence and resulting action list matter more than the number.

---

## Scoring scale

Score each category from 1–10:

```text
10  no concrete structural issue found
8–9 small bounded debt; architecture remains clear
6–7 meaningful local structural issue(s)
4–5 duplicated ownership / hidden policy / abstraction debt increasing change cost
1–3 multiple competing truths or unclear ownership
```

Do not award or remove points mechanically for LOC, file count, comment count, method size, or number of abstractions.

---

# 1 — Single Source of Truth

### Question

For every long-lived piece of mutable state:

```text
is this authoritative state
or
is it a cached / duplicated copy of another fact?
```

### Look for

```text
parallel collections describing the same entities
assignment/state copied into multiple owners
cached aggregates derived cheaply from live state
duplicated status flags
snapshot skill/stat values beside live state
controller state mirroring domain state
```

Strong smell:

```text
A changes, therefore B must also be manually updated
```

### Measure

Count:

```text
derived mutable fields
parallel mutable collections
same domain fact stored in multiple owners
manual synchronisation sites
```

Preferred target:

```text
0 unjustified duplicated mutable facts
```

---

# 2 — Durable Identity

### Question

Does a domain object remember what something **is**, or merely where it currently happens to live?

### Look for

```text
stored Vec indices
array slots
collection offsets
runtime ordering used as identity
reconstructed IDs from multiple fields
spawn-order assumptions
```

Strong smell:

```text
reordering a collection changes semantic identity
```

### Measure

Count persistent fields behaving like:

```text
*_index
*_slot
*_offset
position_in_*
```

Classify each as:

```text
transient lookup
persistent identity
```

Preferred target:

```text
0 persistent incidental identities
```

---

# 3 — Abstractions Must Pay Rent

### Question

Does each abstraction protect a real boundary, invariant, reuse case, or behavioural distinction?

### Look for

```text
trait with one implementation
enum with one variant
wrapper around one field
forwarding class
public helper with one production caller
cache read only by tests
closed executable enum duplicating an existing Task/Action path
```

### Measure

For each candidate, count:

```text
production implementations
production callers
distinct behaviours represented
cross-module / cross-crate ownership boundary
```

Heuristic:

```text
0 production consumers
    delete candidate

1 production consumer
    inline/localise candidate unless it protects a real boundary

1 implementation trait
    challenge it
    but keep if it is the actual dependency-inversion seam

2+ real consumers / behaviours
    abstraction begins to prove itself
```

Important exception:

A single-implementation trait is justified when it expresses a real ownership inversion, e.g.:

```text
RPG defines lifecycle
Starfall supplies concrete policy
```

---

# 4 — Domain Ownership / Leakage

### Question

Does each domain know only concepts it actually owns?

Expected hierarchy:

```text
A4K
    reusable engine substrate

RPG
    genre-neutral gameplay framework

Starfall
    authored / game-specific actualisation

Bevy
    runtime composition
```

### Look for

```text
Farm concepts in RPG
Starfall constants in generic scheduling
game inventory assumptions inside Worksite
renderer-specific concepts inside world substrate
generic code interpreting authored string keys
```

### Measure

Count:

```text
forbidden higher-domain type references
higher-domain imports
higher-domain constants/vocabulary in generic code
```

Preferred target:

```text
0 unexplained downward domain leaks
```

---

# 5 — Invariants Enforced in Code

### Question

Is a rule merely documented, or can the implementation actually prevent/reject invalid state?

Look for prose such as:

```text
must
never
exactly one
at most one
always
cannot
```

Then find the enforcement.

### Enforcement strength

Best to weakest:

```text
type system
signature / visibility
construction API
always-on assertion
runtime validation
test
documentation only
```

Strong smells:

```text
comment says impossible, code permits it
debug_assert! protects release-critical correctness
silent overwrite/fallback path hides invalid authored state
```

### Measure

Count:

```text
prose-only invariants
debug-only invariants affecting release correctness
silent overwrite / fallback paths
```

Preferred target:

```text
0 important prose-only invariants
```

---

# 6 — Hidden Policy / Accidental Semantics

### Question

Is behaviour being decided by something with no domain meaning?

### Look for

```text
Vec order as priority
string/ID lexical order as scheduling policy
map iteration order
spawn order
test order
insertion order
stable-sort preservation treated as gameplay
```

Strong smell:

```text
changing unrelated ordering changes gameplay outcome
```

### Measure

Count:

```text
tie-breakers with no semantic justification
order-dependent tests
implicit ordering assumptions
```

Preferred target:

```text
0 accidental policy sources
```

If two outcomes are genuinely equivalent, let them remain equivalent rather than inventing a tie-break policy.

---

# 7 — Context / Parameter Cohesion

### Question

Are the same groups of values repeatedly passed together because they already form a real concept?

### Look for repeated bundles

```text
entities + map + game
stats + progression
world + actor + rng
```

### Measure

Count:

```text
too_many_arguments suppressions
same 3+ parameters repeated across adjacent layers
immediate re-wrapping into an existing context type
```

Preferred response:

Reuse existing context types where ownership/mutability match:

```text
WorldView
ActionWorld
ActionContext
Agent
```

Do not create a `MegaContext`, `ExecutionContext`, `WorkerContext`, etc. solely to reduce argument count.

Large orchestration signatures may be legitimate.

---

# 8 — Public Surface Discipline

### Question

Is every public item public because production code outside its owner needs it?

### Look for

```text
test-only getters
public structs used only internally
historical re-exports
public helpers with no external caller
```

### Measure

For each public item classify:

```text
cross-crate production caller
same-crate caller only
test-only caller
no caller
```

Preferred target:

```text
public      cross-boundary production use
pub(crate)  same-crate collaboration
private     local implementation detail
```

Strong smell:

```text
public because tests need access
```

---

# 9 — Wrapper / Forwarder Value

### Question

Does a wrapper add semantics, identity, invariants, ownership, or an API boundary?

Worth keeping when it does something like:

```text
JobScope
    participates in assignment identity

FailedAssignment
    exact assignment + failure reason form one concept

WorksiteWorker
    unifies membership + held assignment

WorksiteView
    read-only behaviour boundary

TaskTick
    outcome + budget accounting travel together
```

Weak wrapper pattern:

```text
one field
one getter
one forwarder
no invariant
no reuse
no ownership boundary
```

### Measure

Count wrappers with:

```text
1 field
1 production caller
0 independent invariants
```

These are review candidates, not automatic deletions.

---

# 10 — Tests vs Architecture Policy

### Question

Are tests protecting intentional behaviour, or accidentally freezing implementation detail?

### Look for

```text
tests expecting an exact tie winner with no semantic basis
tests requiring a public getter used nowhere else
tests locking collection order
tests forcing obsolete wrapper types to remain
```

Strong smell:

```text
"we need this behaviour because the test expects it"
```

instead of:

```text
"the test expects it because the domain requires it"
```

### Measure

Count:

```text
test-only production APIs
order-sensitive tests without a domain reason
tests asserting implementation shape rather than observable behaviour
```

---

# Reassessment Summary Format

Use:

```text
Domain: <path>

Single source of truth
    score:
    candidates:
    resolved:
    justified:
    remaining:

Durable identity
    score:
    candidates:
    resolved:
    justified:
    remaining:

Abstractions pay rent
    score:
    candidates:
    resolved:
    justified:
    remaining:

Domain ownership
    score:
    candidates:
    resolved:
    justified:
    remaining:

Invariant enforcement
    score:
    candidates:
    resolved:
    justified:
    remaining:

Hidden policy
    score:
    candidates:
    resolved:
    justified:
    remaining:

Context / parameter cohesion
    score:
    candidates:
    resolved:
    justified:
    remaining:

Public surface
    score:
    candidates:
    resolved:
    justified:
    remaining:

Wrapper / forwarder value
    score:
    candidates:
    resolved:
    justified:
    remaining:

Tests vs architecture policy
    score:
    candidates:
    resolved:
    justified:
    remaining:
```

Then report:

```text
Confirmed structural debt remaining: N
Investigation-only candidates remaining: N
```

---

# Overall Score

If an overall number is useful:

```text
overall = mean(category scores)
```

Keep one decimal place.

Do not introduce weighting until repeated reassessments show that some categories should matter more.

The score is a trend indicator only.

The action list is the real output.

---

# Review Yield

Track whether the rubric itself is useful.

For each reassessment:

```text
Candidates surfaced
Confirmed real issues
Justified exceptions
False positives
```

Calculate:

```text
review yield =
confirmed real issues / candidates surfaced
```

Example:

```text
10 candidates
6 real issues
3 justified
1 false positive

review yield = 60%
```

A useful rubric should surface bounded real issues while correctly allowing justified exceptions.

---

# Anti-Metrics

Do not use these as standalone quality scores:

```text
lines of code
number of comments
number of files
file length
method length
number of traits
number of structs
number of generics
test count
comment percentage
cyclomatic complexity alone
```

They may locate investigation targets, but do not prove poor architecture.

---

# Preferred Improvement Signal

The strongest positive signal is:

```text
fewer competing sources of truth
fewer incidental identities
fewer unjustified abstractions
stronger enforced invariants
smaller necessary public surface
same or stronger behaviour
same or better validation
```

Especially valuable:

```text
production implementation shrinks
while
enforced guarantees increase
```

---

# Reassessment Procedure

1. Read the whole target domain before scoring.
2. Usage-scan suspicious public items and abstractions.
3. Identify candidates using the metrics above.
4. Separate each into:
   - confirmed issue
   - justified exception
   - needs investigation
5. Score the current state.
6. Produce a bounded action list.
7. Apply changes.
8. Re-run the same scorecard.
9. Compare:
   - score movement
   - confirmed debt count
   - review yield
   - production LOC direction
   - validation status
10. Stop when remaining findings are justified rather than forcing a perfect score.

---

# Stop Condition

A domain is considered structurally reviewed when:

```text
confirmed structural debt = 0
```

and remaining candidates are explicitly justified.

A `10/10` is not required.

Do not manufacture work solely to improve the score.
