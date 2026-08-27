This gpt_brief..md file defines the structure and content expectations for a Task Brief — a durable, restart-safe work packet for one bounded repository change.

# Task Brief

## Purpose
A Task Brief is an execution artifact, not a stable feature description and not a chat summary.

It captures **what one branch must change, what evidence the agent must inspect, what boundaries must hold, and what proves completion**.

It is designed so a coding agent can begin or resume work from repository state without the original planning conversation.

Inactive ready briefs live in the project's declared task queue — `.tasks/<slug>.md` by default, or
a pipeline-local `tasks/` folder where the project uses pipelines. `gpt_env.agent_workflow.md` owns
that choice.

When activated, the brief is carried onto its dedicated branch and kept current through implementation, review, blocking, and handoff.

A Task Brief is temporary relative to Feature Briefs and In-Repo Docs. It may be archived or removed after the work is merged and all durable knowledge has been reconciled into the correct standing artifacts.

---

## Relationship to Other Briefs

- **Project Summary** — whole-project navigation, status, and relative size
- **Feature Brief** — stable account of what the feature is and does
- **In-Repo Doc** — stable implementation map of the current feature
- **Milestone** — one bounded outcome several tasks add up to; owns the task order and the acceptance gate (optional; see `gpt_env.agent_workflow.md`)
- **Task Brief** — bounded change contract for one queued or active piece of work

Do not duplicate standing documentation into a Task Brief. Link or name it under `Read First`, then record only task-specific context, deltas, constraints, and findings.

---

## Structure

```md
# Task Brief — <Task Name>

**Milestone:** <owning milestone and phase, or omit>
**Status:** <Draft | Ready | In Progress | Blocked | Review | Complete | Superseded> — <one-line note>
**Suggested agent:** Heavy | Light | Either
**Branch:** <branch or Not activated>
**Base:** <base branch or commit>
**Depends on:** None | <task, branch, PR, or commit>

---

## Objective
One or two sentences describing the concrete repository result.

## Context
Only the background needed to understand why the change exists and how it fits the feature.

## Read First
- <Feature Brief>
- <In-Repo Doc>
- <source path or supporting reference>

## Current State
- **Confirmed:** ...
- **Likely:** ...
- **Assumed:** ...
- **Open question:** ...

## Required Changes
- ...

## Boundaries
### In scope
- ...

### Out of scope
- ...

### Preserve
- invariant, behaviour, or compatibility rule

## Investigation Required
- question the executing agent must resolve from repository evidence

## Hazards
- known trap that will cost the executing agent time — a mirrored file, a build invocation, a
  concurrently active branch in the same seam

## Landing Order
- slice, where the acceptance list is too wide for one pass

## Likely Touchpoints
- `path/` — expected reason

## Runtime / Data Flow
1. ...
2. ...

## Acceptance Criteria
- [ ] observable behaviour
- [ ] test, build, or proof condition
- [ ] standing docs reconciled when affected

## Validation
- Command / check: Not run
- Result: Not run

## Landing record
| Date | Slice | Landed in |
|---|---|---|
| — | — | — |

## Handoff
**Implementation summary:** Not started.

**Changed areas:** None.

**Docs:** Not assessed.

**Blockers / risks:** None recorded.

**Next action:** Activate or begin implementation.

## Follow-up Tasks
- None
```

Omit optional sections that add no execution value. Do not omit the Objective, Read First, Required Changes, Boundaries, Acceptance Criteria, Validation, or Handoff sections from a ready task.

`Hazards`, `Landing Order` and `Landing record` are optional and earn their place only on a task
large enough to need them — a multi-slice change, or one that will land across several merges. A
`Design Summary` section is likewise worth adding where the change turns on a distinction the
Objective cannot carry alone.

---

## Header Rules

### Identity
**The filename is the task's identity.** A slug is already unique within its queue folder, is what
every inbound link and branch name uses, and needs no allocator.

Do not maintain a parallel `TASK-###` sequence. An identifier a human allocates from memory is
eventually allocated twice, and the collision surfaces later as two documents claiming the same
reference — by which point sibling briefs, commit messages and in-repo docs all point somewhere
ambiguous. Where a brief once carried such a number, keep it as a `**Legacy ID:**` line so old
references still resolve, and do not mint new ones.

Use an ID only where an external system already imposes one — a tracker, a ticketing tool. Then it
is a foreign key recorded in the header, not the task's identity and never its order.

### Milestone
Name the owning milestone, and the phase within it, where the project uses milestones.

**Order does not live here, and does not live in the filename.** The milestone's own task table
declares the sequence, so resequencing never renames a file. This field points at the owner; the
owner points back with the position.

### Status
State the enum, then a one-line note. The enum is what a dispatcher reads; the note is what a human
needs and will otherwise write in its place:

```text
**Status:** In Progress — far people and props shipped; hysteresis and the visual gate remain
```

A status that is only prose cannot be filtered or dispatched against. A status that is only an enum
loses the detail someone will add anyway, badly.

- `Draft` — still being designed; not dispatchable
- `Ready` — restart-safe and available for activation
- `In Progress` — implementation underway
- `Blocked` — cannot proceed without a named dependency or decision
- `Review` — implementation claims completion and awaits review
- `Complete` — accepted or otherwise finished
- `Superseded` — replaced by another task or design

Do not mark a task `Ready` when material scope or acceptance questions remain unresolved.

### Suggested agent
Use task shape, not prestige:
- `Heavy` — broad discovery, cross-system implementation, architecture, long test/fix loop
- `Light` — bounded patch, isolated fix, tests, docs, or targeted review
- `Either` — no meaningful execution advantage

This field is routing advice, not an execution constraint.

### Branch
Use `Not activated` while the brief sits in the queue.

When dispatched, record the actual branch. Do not pre-create a forest of empty branches solely to store queued briefs.

### Base and dependencies
Name the expected base branch or commit and any task, PR, branch, migration, or decision that must land first.

A dependency must be concrete. Avoid vague phrases such as "after renderer work" when a branch, task ID, or acceptance condition can be named.

---

## Section Rules

### Objective
State the end result, not the activity.

Good:
> Introduce a shared monster model source and migrate the base and OotA monster-card forms to consume it without changing rendered card behaviour.

Bad:
> Investigate models and clean up the monster form.

### Context
Keep this short. Include design intent or history only where it changes implementation decisions.

Do not paste the planning conversation or repeat full feature documentation.

### Read First
Name the minimum standing docs and source areas required to orient correctly.

A file belongs here when failing to read it would create a serious risk of implementing the wrong abstraction, duplicating an existing system, or breaking a preserved rule.

### Current State
Separate certainty explicitly:
- **Confirmed:** directly observed in repository state or settled design
- **Likely:** strong inference requiring verification
- **Assumed:** temporary premise used to make the task coherent
- **Open question:** unresolved detail that can materially change scope or implementation

The executing agent must verify likely and assumed items before depending on them.

### Required Changes
Use behaviour and ownership language. State what must be created, changed, migrated, removed, or reconciled.

Do not prescribe exact implementation tokens unless they are settled constraints. Let repository inspection determine local mechanics.

### Boundaries
Define three things where relevant:
- what is included
- what is excluded
- what must remain true

This section prevents an implementation agent from turning a bounded task into adjacent redesign.

### Investigation Required
List repository questions the executing agent must answer before or during implementation.

Investigation is valid inside a ready task when:
- the objective and boundaries remain stable regardless of the answer
- the repository is the source of truth
- acceptance criteria can still be evaluated

If the answer could redefine the objective, the task is still `Draft` or is a dedicated research task.

### Hazards
Traps that will cost the executing agent time and are not discoverable from the code in the moment:
a file hand-mirrored by a test, a build invocation that fails obscurely from the wrong directory, a
concurrently active branch in the same seam, a cold-build failure mode that looks like a code error.

This is not a risk register. Every entry must be actionable in the first hour.

### Landing Order
Where the acceptance list is too wide for one pass, name the slices and their order. This is the
plan; `Landing record` below is what actually happened.

### Landing record
For a task that lands across several merges. One row per slice: date, what shipped, and where.

This is the exception to "no chronological log", and it exists because the alternative is worse —
without it a long-lived brief grows a `## Landed (date)` section per merge, each carrying prose that
belongs in a standing doc. Keep the record to the table.

**The architecture a slice produced does not belong here.** Reconcile it out into the design doc or
In-Repo Doc that owns it, and link. If the Landing record starts explaining *how* something works,
that content has outgrown the brief.

### Likely Touchpoints
List expected areas as orientation, not authority. The agent may discover different ownership.

Do not invent files merely to make the brief look complete.

### Runtime / Data Flow
Include only when sequencing, state transfer, persistence, or ownership flow is material to correct implementation.

Keep it behavioural. This is not a replacement for the In-Repo Doc.

### Acceptance Criteria
Every criterion must be checkable through behaviour, tests, build output, inspection, or documentation reconciliation.

Cover:
- intended behaviour
- important preserved behaviour
- validation or test expectations
- standing-document updates when the feature or implementation shape changes

Avoid criteria such as "code is clean" or "implementation is robust" without a concrete proof condition.

### Validation
Update during execution with exact commands, checks, test targets, manual scenarios, and results.

Do not claim completion from file existence or tool success alone.

### Handoff
This is the durable continuation state.

Record:
- what was implemented
- what areas changed
- what validation passed or failed
- whether Feature Briefs, In-Repo Docs, or the Project Summary changed
- blockers and known risks
- the exact next action when incomplete

Do not use the Handoff as a chronological work log.

### Follow-up Tasks
Record adjacent worthwhile work that is not required by current acceptance criteria.

Prefer creating a new queued Task Brief when the follow-up is concrete enough. Do not silently absorb it into the current branch.

---

## Production

Task Briefs are produced in planning chat after the work has a stable objective and boundary.

Create one when:
- a coding session should be able to begin without the planning conversation
- work needs to wait for execution capacity
- work may move between models or sessions
- a branch needs an explicit completion contract

Keep it `Draft` when:
- the idea is still a concept or broad Epic
- an unresolved decision could change the objective or ownership
- acceptance cannot yet be stated

Mark it `Ready` only after a restart-safety check:

> Could an unfamiliar agent read the repository and this brief, then begin without asking what the task actually means?

If not, tighten the brief or return to planning.

---

## Activation and Consumption

### Planning
- writes and refines queued briefs in the task queue
- assigns the milestone, dependencies, boundaries, and acceptance criteria
- activates a ready brief onto a branch when execution capacity is available

### Code
- confirms branch and base
- reads the Task Brief and every `Read First` item
- verifies assumptions against repository evidence
- implements within boundaries
- updates Validation and Handoff before stopping

### QA / Review
- reviews branch output against the Task Brief and standing docs
- distinguishes implementation defects from Task Brief defects or documentation drift
- returns the task to `In Progress`, marks it `Blocked`, or accepts it as `Complete`

---

## Retirement

After merge or abandonment:
1. reconcile durable behaviour and implementation knowledge into Feature Briefs, In-Repo Docs, the
   owning Milestone, or the Project Summary as appropriate
2. preserve follow-up work as separate queued briefs
3. **demote the brief; do not delete it**

Do not leave durable system rules stranded only inside a completed Task Brief.

### Why completed briefs are kept

A Task Brief is temporary *relative to* standing docs, which means it stops being authoritative —
not that it stops being useful. Its Handoff and Investigation sections record the things a design
doc has no room for and a commit message never carries: what was tried, what failed, what the code
turned out to be rather than what the brief assumed.

Deleting that on merge destroys the only record of why an approach was not taken, and the next
session re-tries it.

So: reconcile the **rules** out, and keep the brief where it is with a `Complete` status. The
milestone or pipeline index should say plainly that completed briefs are retained for their
decisions and dead ends, not as live work — otherwise a reader mistakes a finished task for a queued
one.

Archive or remove only when a brief has no durable content at all, or when it is `Superseded` and
its successor already carries everything worth keeping.

### Not every change has a brief

Most projects adopt the brief-per-change convention partway through. Work that predates it does not
get a brief written retroactively — a reconstructed brief is a worse artifact than an honest gap,
because it looks like a contemporaneous record and is not.

Record those arcs in the owning milestone instead: what shipped, where it landed, and which standing
doc documents it. Mark that list as history, not as a backlog.

---

## What Good Looks Like

- readable quickly but complete enough to execute
- objective describes a result rather than an activity
- repository evidence is clearly separated from assumptions
- boundaries stop adjacent redesign
- acceptance criteria prove both new and preserved behaviour
- another agent can resume entirely from branch state
- Handoff says exactly what happened and what remains
- the status enum can be filtered on, and the note beside it says the useful thing
- the milestone can place the task without opening it
- durable rules the work produced have already moved to the doc that owns them

## What Bad Looks Like

- a transcript or broad feature brainstorm renamed as a Task Brief
- `Ready` status with unresolved scope-changing questions
- exact code prescriptions based on unverified assumptions
- no out-of-scope or preserve rules for a risky change
- acceptance criteria that cannot be tested or inspected
- progress reported only in chat
- permanent feature rules left behind in a temporary task artifact
- a status that is a paragraph, so nothing can dispatch against it
- a `## Landed (date)` section per merge, each explaining architecture that belongs in a standing doc
- a numeric ID nobody allocates, so two briefs share one
- a completed brief deleted on merge, taking its dead ends with it
