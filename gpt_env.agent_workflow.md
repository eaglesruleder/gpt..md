This gpt_env..md file defines the repository workflow for preparing, assigning, resuming, and reviewing work across planning chats and coding agents.

# Agent Task Workflow Environment

## Purpose
Use repository artifacts to move work between AI sessions without reconstructing prior conversations.

The workflow assumes:
- planning chats are used to develop and decompose work
- coding agents are scarce execution capacity
- repository state, not chat history, is durable context
- `.tasks/` holds prepared work not yet assigned
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

## `.tasks/` Queue

Store ready but inactive Task Briefs at:

```text
.tasks/<task-id>-<slug>.md
```

Example:

```text
.tasks/TASK-014-renderer-width-perspective.md
```

`.tasks/` is project-owned repository content. Do not place it under IDE metadata folders such as `.idea/`.

Do not create a branch merely to hold a queued brief. Create the branch when dispatching the task unless it already contains useful isolated state such as exploratory commits, scaffolding, task-specific documentation, or a deliberately frozen base.

---

## Task Lifecycle

```text
Draft -> Ready in .tasks/ -> Activated on branch -> In Progress -> Review -> Complete
                                                       \-> Blocked / Returned
```

A Task Brief is `Ready` only when:
- the objective and boundaries are explicit
- required standing docs and source areas are named
- confirmed facts are separated from assumptions and investigation
- acceptance criteria can prove success

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
agent/<task-id>-<slug>
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
Do not use a bounded task as permission for adjacent redesign. Record worthwhile adjacent work as a new `.tasks/` item unless required by the current acceptance criteria.

### Keep the active brief current
Update it for material dependencies, invalidated assumptions, blockers, changed acceptance tests, and follow-up work that must survive the session. Do not turn it into a chronological diary.

### Reconcile standing docs
- update the Feature Brief when behaviour or the feature loop changes
- update the In-Repo Doc when ownership, state, rules, or implementation shape changes
- update the Project Summary only when navigation, status, size, or priority changes

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

Persist actionable findings in the branch Task Brief or a new `.tasks/` item, not only in review chat.

---

## Failure Modes

### Conversation summary masquerading as a Task Brief
Long rationale, weak required changes, no proof of completion.

### Empty branch backlog
Many stale branches exist only to hold notes. Keep inactive work in `.tasks/` and branch at dispatch.

### Agent-specific task design
The brief depends on one model's habits. Define repository evidence, behaviour, constraints, and proof instead.

### Stable docs used as session logs
Temporary findings pollute Feature Briefs or In-Repo Docs. Move them to the Task Brief.

### Completion reported only in chat
Later sessions cannot determine what passed or remains. Update the branch artifact before stopping.

---

## What Good Looks Like

- planning capacity continuously produces bounded work packets
- coding capacity begins with context and proof criteria already prepared
- agent or context resets require no reconstruction from old chats
- `.tasks/` remains a dispatch queue rather than a speculative branch forest
- every active branch has one objective and one completion contract
- implementation findings survive in repository artifacts
