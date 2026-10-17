---
name: coding-task-execution
description: Execute a plan previously produced and persisted by the Coding Task Inception skill, tracking per-sub-task status and building a work summary report for human code review. Use when the user says "Coding Task Execution", or asks to start, resume or continue work on a stored plan.
---

# Coding Task Execution

Input is a confirmed plan at
`coding-tasks/<slug>/plan.md`.
That file is the source of truth across sessions and agents, not the in-session task list. List the
`coding-tasks/` directory to find the slug rather than guessing it: one match, use it; several, ask
the user which; none, there is no plan.

If no such plan exists, stop and say so — run `coding-task-inception` first. Do not improvise one.



## Phase 1 — Load and reconcile

Read the plan in full: decision table, research findings, sub-task table, standing assumptions,
progress log.

Reconcile it against reality before doing anything:

- Sub-tasks marked `done` are **not** redone. Verify the claim cheaply (does the file exist, does
  the test run) rather than trusting the label blindly, and say what you checked.

- Sub-tasks marked `in progress` from an earlier session are the first thing to resolve. Establish
  what actually landed, finish or restart that one task, and do not start anything new until it is
  settled.

- The plan's research findings were true when written. Anything it cites — a selector, a file:line, a config key — is verified again the moment you rely on it.

Mirror the plan into the in-session task list if it helps you work, but the persisted file is what you update.


**Establish the baseline before changing any code.** 
Confirm that all test suites are currently passing. Without a confirmed baseline you cannot tell
your own regressions from failures that were already there.




## Phase 2 — Order the work

Derive the execution order from the plan's *Blocked by* column. Never start a sub-task whose
blockers are not all `done`. Name the tasks that are unblocked right now before starting.

**Parallelism is for writing code, never for verifying it.** Two agents running builds or tests at once race on shared state and produce failures that look like regressions. So:
- Subagents may draft code for sub-tasks that are unblocked *and* whose file sets are disjoint.
- Every lint, build and test run happens serially, on the main agent, in the main working tree.
- Subagents work **directly in the main tree**, not in worktrees. With no commits being made (see
  Standing rules), a worktree has no clean way to hand changes back.
  - Partition by file. Two agents must never hold the same file open for writing.

Where the graph is a chain, say so and run it serially rather than manufacturing parallelism.


## Phase 3 — Bridge the gaps before you start to limit the interruptions

Look ahead to the whole plan and identify any gap, clarification, decision, missing info or
permission you need from the user. Ask now instead of interrupting later.

Update the plan and sub-task descriptions with the additional info that emerges. Record each
question and answer, and use them during execution to avoid unnecessary interruptions.
The aim is to avoid preventable interruptions during the execition, and limit the number of times you will have to stop and hand over to the user. Therefore the plan execution will stop to hand over to the user for things that cannot be unticipated now.
If the user has already approved the step that follows a checkpoint, the checkpoint is a report,
not a stop: post it, log that the earlier approval covered it, and continue.


## Phase 4 — Execute

For each sub-task, in order:

1. Set its Status to `in progress` in the persisted plan **before** starting.
2. Do the work as the sub-task describes.
3. Verify it (Phase 6).
4. Set Status to `done` and append a dated line to the plan's progress log.
5. **Replace the comments you just wrote with self-explaining code (Phase 7), then re-verify.**
6. Write its entry in the work summary report (Phase 8) and post that entry in the chat — as a
   report, with **no question appended**. Moving on to the next unblocked sub-task needs no
   permission; a checkpoint (Phase 5) is its own message.

A sub-task that cannot be finished stays `in progress`. Never mark `done` on partial work, failing
tests, or anything you worked around rather than solved — report it and stop.



## Phase 5 — Checkpoints

When the plan contains a review checkpoint, **stop completely** and hand back. Show what the
checkpoint task says to show and wait for explicit permission to continue. Do not start the next
sub-task, do not "get a head start" on unrelated ones.

**The checkpoint's ask is "may I continue?", and nothing else rides with it.**


## Phase 6 — Verify before marking done

Organise verification into tiers ordered by cost (build time, live service usage). Run the cheapest
tier that covers the change; climb only as far as the change requires.

| Tier | Scope | Gate |
|---|---|---|
| **Fast** | build, lint, unit tests — no live external services | run freely, after every change |
| **Scoped** | individual tests relevant to the changed files, including live-service tests | counts against the budget |
| **Full local** | whole suite locally with live services — only if the local environment is confirmed clean | ask first |
| **Full pipeline** | clean-room build, whole suite on the test environment | ask first, end of execution only |

**The fast tier is the loop you live in.** Get it green before climbing any higher. Never substitute
a higher tier for the fast loop, and never let the fast tier stand in for the full pipeline.

**Sequence verification tiers, never run them in parallel.**

**Prove which environment a run actually reached — never infer it from the flag you passed.**

**Prove the test fails, not just that it passes.** For each behaviour the sub-task exists to
protect, break the production code deliberately, watch that specific test fail, then revert and
confirm green. Apply mutation testing to that tehavour and related tests.

**Triage before blaming the change.** A stale local server may cause false failure, for example —
check whether the server predates the last build and restart it if so.


## Phase 7 — Replace the comments with self-explaining code

Runs **after** the sub-task is marked `done` and green, on **only the comments this task added or
modified** — never on comments the task did not touch.

The goal: a comment earns its place only when what it says **cannot** be said by the code.

**Every change in this phase is a semantics-preserving refactoring.** Nothing about behaviour
changes, and the verification that was green before the phase must be green after it, unchanged.

The three rules:

1. **Do not remove a comment whose information is not expressed in the code.**
2. **Do not leave a comment whose information can be deduced from the code.**
3. **Do not make a refactoring that changes the semantic behaviour of the code.**

Express intent through names (classes, methods, parameters, variables, constants), extracted
private helpers, and types that make invalid states unrepresentable — in both implementation and
test code.

Re-run the tier that covered the sub-task and expect an **identical** result. A changed test count
or a newly red test means rule 3 was broken: revert and say so.


## Phase 8 — Work summary report

Maintained at `coding-tasks/<slug>/work-summary.md` — the same
`<slug>` directory the plan came from — appended as each sub-task completes and **posted in the
chat at the same time**.

One section per completed sub-task:

- **Sub-task** — number and title.
- **Files** — each path created or modified, with one line on *why* that file changed.
- **Verification** — what was run and the result, quoted rather than characterised.
- **Needs human review** — judgement calls, deviations from the plan, assumptions baked into code,
  brittleness knowingly accepted. Write "nothing" only when it is true.

Keep entries short. A reviewer should be able to read the whole report before opening the diff.


## Phase 9 — Retrospect

Review how phases 1–8 actually went and fold anything durable back into this skill. Corrections the
user made are the highest-value input. Ask the user if they want to add any other improvement to
this skill.

**Where the improvement goes — check before editing anything.** Phase logic belongs in this file.
Interaction protocol rules belong in the shared interaction-protocol file.


## Standing rules

- **Git** Stay on the current branch. Make no commits. Leave the working tree for the user to stage
  themselves. Never stash, revert, or clean their pre-existing uncommitted changes.
- **Plan drift** If anything contradicts the plan, **stop and ask, always**. Report the finding and
  the options; do not silently redesign, and do not silently comply with a plan you now know to be
  wrong. Record the resolution in the plan's progress log before resuming.
- **Scope** The plan defines the work. Do not widen it, do not fold in unrelated cleanups, and do
  not narrow it because a step looks awkward.



## Anti-patterns

- Redoing a sub-task already marked `done`, or starting one whose blockers are not.
- Marking `done` with failing or unrun verification.
- Running builds or tests in parallel against shared local services.
- Continuing past a review checkpoint without explicit permission.
- Committing, branching, or tidying the working tree.
- Batching the work summary to the end instead of reporting per sub-task.
- Adapting the plan quietly when it turns out to be wrong.
- Handing back a question, blocker or checkpoint without the context, cause and options the user
  needs to answer it.
- Appending a question to a work-summary entry, a status report or a verification result.
- Skipping Phase 7, or doing it *before* the sub-task is green.
- Touching comments the task did not add or modify during Phase 7.
- Letting Phase 7 change behaviour: a different test count or a newly red test means the
  refactoring was not semantics-preserving. Revert it and say so.
- Starting work without a confirmed green baseline, or establishing one by running the suites
  yourself rather than asking the user.
