This gpt_env..md file defines the repository workflow for preparing, assigning, resuming, and reviewing work across planning chats and coding agents.

# Agent Task Workflow Environment

## Purpose
Use repository artifacts to move work between AI sessions without reconstructing prior conversations.

The workflow assumes:
- planning chats are used to develop and decompose work
- coding agents are scarce execution capacity
- repository state, not chat history, is durable context
- the task queue holds prepared work not yet assigned
- branches isolate active work

---

## Artifact Chain

```text
Project Summary
  -> Feature Brief — what the feature is and does
  -> In-Repo Doc — how the current feature is built
  -> Task Brief — what one bounded change must accomplish
  -> Branch — implementation, proof, and handoff
```

Do not use stable Feature Briefs or In-Repo Docs as progress logs. Temporary execution state belongs in the Task Brief.

### Milestones and pipelines

Two optional tiers slot into that chain when a project is large enough to need them. Adopt either
only when it is carrying weight; a project with one work stream needs neither.

**Milestone** — a bounded outcome that several Task Briefs add up to. It sits between the Feature
Brief and the Task Brief, and it owns two things nothing else can:

```text
Project Summary
  -> Pipeline        — one independent work stream, with its own milestone sequence
     -> Milestone    — one bounded outcome; owns the task ORDER and the acceptance gate
        -> Task Brief
           -> Branch
```

- **The task order.** Sequence belongs to the milestone, not to filenames and not to an ID. A brief
  is identified by its slug; the milestone's task table is what puts it in order. Resequencing then
  never renames a file or breaks an inbound link.
- **The honest scope record.** A milestone lists what shipped *without* a brief as well as what
  shipped with one. Most projects adopt the brief-per-change convention partway through; pretending
  otherwise leaves the early work invisible. Record those arcs, name where they *are* documented,
  and mark the list as history rather than a backlog of briefs to write retroactively.

**Pipeline** — an independent work stream (renderer / simulation / UI / game; or service / client /
infrastructure) with its own milestone sequence, task queue and standing design. Milestone numbers
are pipeline-local, so always cite them qualified — `Renderer / M2`, never a bare `M2`.

Adopt pipelines when work streams genuinely progress independently. The cost is real: every
milestone reference must be qualified, and one canonical owner per design doc has to be enforced by
hand.

**A milestone is not a Feature Brief.** The Feature Brief says what a thing is; the milestone says
which bounded outcome a set of changes is chasing, and when it is done.

### Documentation kinds

Standing design docs benefit from a **kind prefix** in the filename once a project has more than a
handful. The prefix is a contract, not a label: it sets what the header must carry and how a reader
should treat the doc.

| Prefix | Means | Header must carry |
|---|---|---|
| `feature.` | one bounded subsystem | the **rule** that must hold |
| `layer.` | cross-cutting design decomposing onto several subsystems | **which** features it lands on |
| `research.` | uncommitted direction | a **graduation or expiry** condition |
| `concept.` | setting, premise, canon | that it is **not scope** |

`research.` and `concept.` are separate because their failure modes are opposite: research risks
never being built, concept risks being mistaken for committed work.

A kind change is a rename. That is acceptable precisely because a kind change is rare and worth
being loud about — unlike task order, which changes routinely and therefore lives in the milestone
rather than in a filename.

### Where implementation lives

A standing design doc never describes how the current code works. Three tiers, and the boundary
between them is what stops the same content existing in three places and drifting:

```text
in-language file header   why THIS FILE is shaped this way   per file, checked by the toolchain
In-Repo Doc               how the live subsystem works NOW   per subsystem, beside the code
Feature Brief / design    what it is and what it should do   per concept, tracks intent
```

**Dead ends split the same way.** An implementation dead end — a formula that failed, a cache key
that thrashed, an API that could not carry the shape — follows the code. A design dead end — an
architecture that was built and removed — stays in the design doc. Both must be written down; only
their address differs.

Where the language already provides the first tier well enough to serve AI readers too, a separate
AI-facing implementation doc is redundant. See `gpt_brief.repo.md`.

---

## Work Surfaces

### Planning chat
Use for design, decomposition, scope boundaries, acceptance criteria, Task Brief authoring, and review of agent output.

Planning is complete only when another agent can act without the original conversation.

### Heavy coding agent
Prefer for broad repository discovery, cross-system changes, architectural refactors, and long diagnose-build-test loops.

### Light coding agent
Prefer for bounded patches, isolated fixes, targeted tests, documentation reconciliation, and parallel work with a clear seam.

Routing is advisory. Task shape decides the agent.

---

## Task Queue

Ready but inactive Task Briefs live in the repository, in one declared location. Two layouts work;
pick one per project and say which in the project's own documentation.

**Flat queue** — the default, and correct for a project with one work stream:

```text
.tasks/<slug>.md
.tasks/renderer-width-perspective.md
```

**Pipeline-local queue** — when the project uses pipelines, file each brief with the work stream
that owns the change:

```text
docs/<NN>-<pipeline>/tasks/<slug>.md
docs/01-renderer/tasks/entity-representation-lod.md
```

The queue is project-owned repository content. Do not place it under IDE metadata folders such as
`.idea/`.

**The filename is the task's identity.** A slug is already unique within its folder and is what
every inbound link uses; a parallel ID sequence adds an allocator to maintain, a second thing to
renumber, and a collision to discover later. Use IDs only where an external system already imposes
them — a tracker, a ticketing tool — and then treat that ID as a foreign key, not as the order.

Do not create a branch merely to hold a queued brief. Create the branch when dispatching the task
unless it already contains useful isolated state such as exploratory commits, scaffolding,
task-specific documentation, or a deliberately frozen base.

---

## Task Lifecycle

```text
Draft -> Ready in queue -> Activated on branch -> In Progress -> Review -> Complete
                                                       \-> Blocked / Returned
```

`gpt_brief.task.md` owns the status vocabulary and the `Ready` gate; this file does not restate
them. The one workflow consequence: **do not dispatch a brief that is not `Ready`.** An unready
brief consumes execution capacity to rediscover what planning should have settled.

---

## Branch Activation

When dispatching a ready task:
1. confirm the correct base branch and dependencies
2. create a dedicated branch
3. keep the active Task Brief on that branch
4. record branch, base, status, and suggested agent
5. execute from repository evidence

Recommended branch name:

```text
agent/<slug>
```

Follow an established repository convention where one exists.

Do not activate parallel branches that mutate the same ownership seam unless dependency and merge order are explicit.

---

## Session Start Contract

An execution agent must:
1. confirm repository and branch
2. read the active Task Brief
3. read every item under `Read First`
4. inspect named source areas before accepting implementation assumptions
5. compare branch state with the recorded handoff
6. continue from repository evidence, not presumed chat history

A prepared task should support a minimal dispatch prompt:

> Check out `<branch>`, read the Task Brief, and execute it.

If that is insufficient, the work packet is not restart-safe.

---

## Execution Rules

### Repository evidence overrides assumptions
When code contradicts the Task Brief, record the discovered state and update the brief. Preserve the objective unless the discovery materially invalidates its scope or intended behaviour.

Do not silently implement against a known-false premise.

### Preserve boundaries
Do not use a bounded task as permission for adjacent redesign. Record worthwhile adjacent work as a new queued brief unless required by the current acceptance criteria.

### Keep the active brief current
Update it for material dependencies, invalidated assumptions, blockers, changed acceptance tests, and follow-up work that must survive the session. Do not turn it into a chronological diary.

### Reconcile standing docs
- update the Feature Brief when behaviour or the feature loop changes
- update the In-Repo Doc when ownership, state, rules, or implementation shape changes
- update the Project Summary only when navigation, status, size, or priority changes
- update the Milestone when a task lands, when its scope or acceptance moves, or when an arc ships
  without a brief

Two reconciliations are routinely missed and are worth naming:

- **A `Status:` line that has gone stale is a defect.** A design doc saying "unbuilt" about something
  that shipped is worse than no status at all: it is read as current and believed. Check the status
  of every doc a change touches, not only its body.
- **Durable knowledge must not be left in a Task Brief.** A brief that accumulated architecture — a
  layer split, a version-lane rule, a dead end — has that content reconciled out into the design doc
  or In-Repo Doc that owns it before the task is considered done. The brief keeps the execution
  record; the standing doc keeps the rule.

---

## Completion Handoff

Before stopping, record in the Task Brief:
- current status
- implementation summary
- files or systems changed
- validation run and result
- acceptance criteria state
- standing docs updated or intentionally unchanged
- blockers, risks, and follow-up tasks
- exact next action when incomplete

A chat summary may accompany this, but it is not the durable handoff.

---

## Review Contract

Review against:
1. Task Brief objective and boundaries
2. acceptance criteria
3. tests and build output
4. Feature Brief behaviour
5. In-Repo Doc rules and shape
6. conventions in the touched area

Classify misses as:
- implementation defect
- Task Brief defect
- standing-doc drift
- scope breach
- environment or tooling failure

Persist actionable findings in the branch Task Brief or a new queued brief, not only in review chat.

---

## Failure Modes

### Conversation summary masquerading as a Task Brief
Long rationale, weak required changes, no proof of completion.

### Empty branch backlog
Many stale branches exist only to hold notes. Keep inactive work in the queue and branch at dispatch.

### Agent-specific task design
The brief depends on one model's habits. Define repository evidence, behaviour, constraints, and proof instead.

### Stable docs used as session logs
Temporary findings pollute Feature Briefs or In-Repo Docs. Move them to the Task Brief.

### Completion reported only in chat
Later sessions cannot determine what passed or remains. Update the branch artifact before stopping.

### An ID sequence with no allocator
Two briefs get the same number, or a brief is filed `Unassigned`, and cross-references silently
point at the wrong task. The sequence is the problem, not the discipline: an identifier a human must
allocate by remembering will eventually be allocated twice. Let the filename carry identity.

### Milestone numbers that outlive their milestone
Commit and PR titles carry the numbering that was current when they were written. After any
resequencing or split, that history no longer resolves against the docs. Keep a small old-to-new
map in the pipeline or project index; without it every historical PR title becomes unreadable.

### A design doc describing the live code
The design doc and the In-Repo Doc drift into the same subject, then disagree, and a reader has no
way to tell which is current. Split on the tier boundary: intent above, implementation beside the
code.

### An unrunnable acceptance criterion
"Manual visual pass", "verified by inspection", "looks correct" — named as a gate, never defined, so
the work can never be closed, only asserted. Either write down what to look at and what failure
looks like, or drop the criterion. A checklist is usually enough; a procedure is usually ceremony.

---

## What Good Looks Like

- planning capacity continuously produces bounded work packets
- coding capacity begins with context and proof criteria already prepared
- agent or context resets require no reconstruction from old chats
- the task queue remains a dispatch queue rather than a speculative branch forest
- every active branch has one objective and one completion contract
- implementation findings survive in repository artifacts
- every milestone can state what shipped under it, including the parts that shipped without a brief
- a resequence renames nothing
- standing docs and their `Status:` lines still describe the code
