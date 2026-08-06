This gpt_brief..md file defines the structure and content expectations for a Task Brief — a durable, restart-safe work packet for one bounded repository change.

# Task Brief

## Purpose
A Task Brief is an execution artifact, not a stable feature description and not a chat summary.

It captures **what one branch must change, what evidence the agent must inspect, what boundaries must hold, and what proves completion**.

It is designed so a coding agent can begin or resume work from repository state without the original planning conversation.

Inactive ready briefs live in:

```text
.tasks/<task-id>-<slug>.md
```

When activated, the brief is carried onto its dedicated branch and kept current through implementation, review, blocking, and handoff.

A Task Brief is temporary relative to Feature Briefs and In-Repo Docs. It may be archived or removed after the work is merged and all durable knowledge has been reconciled into the correct standing artifacts.

---

## Relationship to Other Briefs

- **Project Summary** — whole-project navigation, status, and relative size
- **Feature Brief** — stable account of what the feature is and does
- **In-Repo Doc** — stable implementation map of the current feature
- **Task Brief** — bounded change contract for one queued or active piece of work

Do not duplicate standing documentation into a Task Brief. Link or name it under `Read First`, then record only task-specific context, deltas, constraints, and findings.

---

## Structure

```md
# Task Brief — <Task Name>

**ID:** TASK-###
**Status:** Draft | Ready | In Progress | Blocked | Review | Complete | Superseded
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

---

## Header Rules

### ID
Use a repository-unique task identifier when the project maintains a queue.

Recommended form:

```text
TASK-014
```

The ID remains stable across filename, branch, review, and follow-up references.

### Status
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
Use `Not activated` while queued in `.tasks/`.

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
- writes and refines queued briefs in `.tasks/`
- assigns IDs, dependencies, boundaries, and acceptance criteria
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
1. reconcile durable behaviour and implementation knowledge into Feature Briefs, In-Repo Docs, or the Project Summary as appropriate
2. preserve follow-up work as separate `.tasks/` briefs
3. archive or remove the completed Task Brief according to repository policy

Do not leave durable system rules stranded only inside a completed Task Brief.

---

## What Good Looks Like

- readable quickly but complete enough to execute
- objective describes a result rather than an activity
- repository evidence is clearly separated from assumptions
- boundaries stop adjacent redesign
- acceptance criteria prove both new and preserved behaviour
- another agent can resume entirely from branch state
- Handoff says exactly what happened and what remains

## What Bad Looks Like

- a transcript or broad feature brainstorm renamed as a Task Brief
- `Ready` status with unresolved scope-changing questions
- exact code prescriptions based on unverified assumptions
- no out-of-scope or preserve rules for a risky change
- acceptance criteria that cannot be tested or inspected
- progress reported only in chat
- permanent feature rules left behind in a temporary task artifact
