---
name: coding-task-inception
description: Turn a coding task brief into a researched, questioned, confirmed and persisted implementation plan before any code is written. Use when the user says "Coding Task Inception".
---

# Coding Task Inception

A nine-phase protocol. The output is a **confirmed, persisted plan**, not code. Write no application code during phases 1–8.

## Phase 1 — Receive the general information

The user supplies background and references. Read every file they name, in full. Hold it; do not
act on it yet.

Read also the Discovery findings report persisted by the previous step, `coding-task-discovery`, at
`coding-tasks/<slug>/discovery.md`.
List the `coding-tasks/` directory to find the slug rather than guessing it: one match, use it;
several, ask the user which; none, say so. It carries
the framing, the domain concepts, the technical implementation details, the contradictions found,
and the open questions it deliberately left for this phase. If it does not exist, say so and ask
whether to run discovery first or proceed without it.

Always read the relevant architecture, domain, and verification docs for the codebase.

If the user asks a question mid-brief, answer it, but do not start planning.


## Phase 2 — Receive the specification

What to implement, constraints, quality criteria, example files, how to verify. Do not push back
yet; collect everything, then note conflicts for phase 5.

A gap you find between the user's brief and the Discovery findings report is **itself a topic**. Raise it alone.


## Phase 3 — Research

The most valuable phase. Budget real effort here — every plan defect in practice traced back to
research skipped, not to poor planning.

**Verify external data sources live.** If the task scrapes or calls an external site, fetch the
actual pages with `curl` into the scratchpad and parse them before planning. Confirm the user's
expected test data against reality.

**Trace every consumer of every file you will change.** Confirm behaviour by reading the code, then
still verify by running tests — never assert "this won't break it" from reasoning alone.

**Follow the call chain to the real consumer.** Trace from the caller to the concrete class through
every layer before believing a method is reachable.

**Check whether the design is already written down.** Read the architecture and design docs before
proposing structure; you may be re-deriving a decision already made.


## Phase 4 — Build the plan

Create sub-tasks with `TaskCreate`, then wire dependencies with `TaskUpdate` `addBlockedBy`. Each
task's description must be self-contained: exact file paths, verified selectors, expected values,
and *why* — a future agent reads the description, not this conversation.

Identify the tasks with no dependencies and say which they are; they can start immediately.

**Check the order against real-world sequencing, not only code dependencies.** A graph correct about
what compiles can still be wrong about what must *happen* first. Those constraints usually live in
docs rather than code; re-read what you cited in research before presenting.

Sequence verification as **two tiers**, never one:
- a fast inner loop on the dev environment
- one authoritative final run on the test environment

Offer **review checkpoints** — explicit stop-and-hand-back tasks where the user inspects the shape
before the next layer goes on. Place each one at a state that compiles and runs; if the dependency
order prevents that, say so and propose the split that fixes it.


## Phase 5 — Question the gaps

Use `AskUserQuestion` with a recommendation as the first option and a concrete `preview` snippet
showing what each choice looks like in code. Every question must change what you build — if a
sensible default exists, take it and state it as an assumption instead.

**Treat the spec's implementation directives as provisional.** The brief may name the interface and
class to modify. If research says otherwise, present the trade-off with costs, flag the divergence
from what they wrote, and get explicit confirmation. Do not silently comply, and do not silently
deviate.


## Phase 6 — Review the plan for completeness, prevent most execution interruptions
Double-ckeck the plan decisions and sub-tasks making sure they are sufficiently detailed and have all the information required for the agent to implement the plan-subtasks with no interruptions such as additional asks or reviews that need to be presented to the user.

Where you can anticipate a gap and a missing information or a decision or a confirmation required, bridge the gap now with the user so the plan execution will have much much less interruptions

Walk again the sub-task list and list every point where the executing agent would have to stop: a
permission gate, an unverified precondition, a detail left to judgement. Ask those questions now and include the required info to avoind the need for an interrupton during the execition.

## Phase 7 — Present, amend, confirm, persist

Present the full plan as a table: number, sub-task, blocked-by. Add files created vs modified,
decisions locked in, assumptions, and anything you would flag as accepted risk.

Invite amendment and **wait**. Expect several rounds; fold each amendment into the tasks before
re-presenting.

**"Invite amendment" is the whole ask, and it is the only one in that message.** 

Persist only after explicit confirmation, to
`coding-tasks/<slug>/plan.md` — the same
`<slug>` directory the discovery report came from. That location is readable and writable by every
session and agent, which the in-session task list is not. Task artifacts are found by path, not by recall. The file must carry:

- frontmatter with name, description, type
- goal and explicit out-of-scope
- the decision table, with the *why* for each
- research findings, with file:line citations
- the sub-task table with a **Status** column (`to be done` / `in progress` / `done`)
- per-sub-task detail: paths, commands, expected values
- standing assumptions and accepted risks
- a dated progress log, and instructions that any agent may update it


## Phase 8 — Retrospect

Once the plan is persisted, review how phases 1–7 actually went and fold anything durable back into
this skill. Corrections the user made are the highest-value input. Ask the user if they want to add any other improvement to this skill.


---

## Anti-patterns

- Writing application code before phase 6 confirmation.
- Planning against an external data source without fetching it first.
- Dropping or retracting a point to get a message down to one ask, instead of moving it to the next
  message.
- Inferring process rules from repo guideline docs instead of asking.
- Declaring a change safe for another stack without running that stack's tests.
- Persisting the plan only to the in-session task list, which the next session cannot read.
- Spawning subagents; this protocol is done inline unless the user asks otherwise.


## Standing rules

- 
- 