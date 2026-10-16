This gpt_env..md file defines Git and change-integration defaults for active repository development: one integration lineage, one implementation pipeline, and new functionality live by default.

# Git & Change Integration Environment

## Purpose

Branches are temporary isolation for changes, not competing long-term versions of a product. The goal is to continuously advance the ordinary running application while keeping related implementation work easy to review, validate, and integrate.

This file owns Git/change integration and feature-activation defaults. The companion gpt_env.agent_workflow.md owns task queues, agent assignment, branch activation, and handoffs; task briefs own scope and acceptance. Follow explicit user instructions and established repository requirements when they deliberately differ from these defaults.

---

## 1. One Integration Lineage

**Default: work on a single branch for one closely related body of work, then return it to main.**

- Continue related fixes, stages, experiments, and agent handoffs on the existing active branch. A new session, agent, or internal stage is not a reason to create another branch.
- Do not grow a tree of branches based on other unfinished feature branches. The deeper the branch ancestry, the more assumptions and integration conflicts the next branch inherits.
- Use main as the shared integration authority, not an old branch that has become an alternate project baseline.
- Bring working, validated slices into main promptly rather than accumulating weeks of changes off-main. Subsequent related work should start from updated main once the previous slice has landed.
- Keep commits reviewable inside the branch; do not translate every commit, stage, view, or subsystem adjustment into a new branch.

**When a single branch is genuinely infeasible:** separate only independently owned work. Integrate the earliest viable slice into main; base subsequent work on the newly integrated state rather than allowing several branches to evolve overlapping versions. If the work is not ready for main, describe the actual blocker instead of treating branch proliferation as a substitute for resolving it.

Parallel branches are valid for genuinely independent ownership or when explicitly requested. Where branches touch the same implementation seam, decide their dependency and integration order before they diverge. Avoid parallel changes to the same files and algorithms without a defined reconciliation owner.

---

## 2. One Code Pipeline for Related Behaviour

**Default: closely related cases share one implementation, parameterised by their real differences.**

- Different LOD tiers, view angles, output densities, platforms, and related cases should use the same underlying algorithm where their correctness contract is the same.
- Express genuine variation through typed inputs, values, policies, or configuration **within one code pipeline**. Do not create separate code branches or Git branches just to represent variants.
- Keep special cases at the narrow seam where behaviour actually differs rather than cloning the whole call chain.
- A separate production algorithm requires a demonstrable difference in responsibility or semantics, not just a convenient way to develop or test it independently.
- Once the replacement is accepted, remove superseded algorithms, compatibility routing, and temporary comparator paths unless they have an ongoing product purpose.

Example: Full N=1 and Far N=16 consuming one projected terrain operation is preferable to separate competing renderers whose projection rules must later be reconciled.

**Configuration expresses variation; it must not conceal an unintegrated replacement.** A resolution input, debug camera position, or user-facing quality setting is legitimate. A switch that chooses between the old application and a new otherwise dormant implementation is not a default integration strategy.

---

## 3. New Work Is Live by Default

**Unless explicitly instructed otherwise, all implementation on the working branch must participate in the ordinary runtime path.**

- Do not add an opt-in CLI argument, environment variable, feature flag, config switch, debug option, or unused API entry point solely to activate new work.
- Launching the application normally, without a special enable-new-feature argument, must exercise the branch's intended implementation.
- Test the ordinary default behaviour, not only a manually enabled experimental path. A passing test suite for code never reached by a normal run does not establish runtime completion.
- If the new path is not safe to run by default, the work is incomplete: fix it, split out a genuinely viable integration slice, or explicitly report the blocker.
- Do not silently fall back to old behaviour in a way that hides failures or incomplete new-code paths.

An activation switch has recurring costs: implementation, routing, invocation, test combinations, documentation, review, merge reconciliation, and eventual removal. It also risks hiding visual or functional regressions because ordinary runs do not exercise the new code.

**Exceptions are deliberate, not assumed.** A user may request a research spike, an A/B comparison, a migration gate, or a real product option. Record which behaviour is default, how each path is exercised, what validates the cutover, and the retirement condition of a temporary switch.

Diagnostic controls remain useful: camera placement, logging, deterministic scenarios, capture settings, and forced test inputs help reproduce the live implementation. They should not decide whether the new implementation exists.

---

## 4. Integration Validation

Before claiming that a branch is complete:

1. Confirm the intended branch, main base, dependencies, and scope of the diff.
2. Check for accidentally duplicated pipelines, unneeded activation flags, dormant new code, and legacy fallbacks.
3. Run relevant automated checks, then run the ordinary application entry point. For visual or interactive work, inspect the live path in representative conditions; synthetic or software-only proofs must not be represented as real-hardware visual acceptance.
4. Validate integration seams between related changes. Two independently green branches are not proof of a working merged product.
5. Integrate validated work into main through the agreed repository workflow and verify the combined result.
6. Retire approved temporary experiments and comparison paths when their purpose ends.

Report these states distinctly: **implemented**, **live on the working branch**, **validated on the default path**, and **integrated into main**. One state does not imply the next.

---

## Recognisable Failure Patterns

| Failure | Correction |
|---|---|
| Each agent session or implementation stage creates a new branch | Continue the active branch and keep reviewable commits |
| Related features grow overlapping branches for weeks | Converge through one lineage or validated slices into main |
| Full/Far or high/low paths duplicate the same algorithm | Use one shared operation with explicit differences as inputs |
| New code only runs with an enable-new argument | Make the new path the branch default unless gating was expressly approved |
| Tests pass but ordinary launches still exercise legacy code | Validate the actual default launch and integration seams |
| The previous renderer survives indefinitely after cutover | Remove temporary alternatives and their routing glue |
| Several branches work alone but fail when combined | Integrate early and validate the combined result |

## Scope Boundary

This file does not prescribe commit-message wording, branch-name spelling, task-document structure, source-code style, or domain-specific architecture. Those belong to existing task, workflow, and style references.

The governing principle is **one integrated codebase, one live default behaviour, and branches that converge instead of multiplying**.
