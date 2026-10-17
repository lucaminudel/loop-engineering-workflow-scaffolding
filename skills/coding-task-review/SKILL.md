---
name: coding-task-review
description: Adversarially review the uncommitted work produced by the Coding Task Execution skill, fix the high-priority findings, and hand the user a PR-review guide and commit message for the pending changes. Use when the user says "Coding Task Review", or asks to review the work done before committing.
---

# Coding Task Review

Fourth and last of the four. Inputs are the plan, work summary, and optionally the discovery report
that the earlier skills persisted:

- `coding-tasks/<slug>/plan.md`
- `coding-tasks/<slug>/work-summary.md`
- `coding-tasks/<slug>/discovery.md` (when it exists)

List the `coding-tasks/` directory to find the slug rather than guessing it. If neither plan nor
work summary exists, stop — there is nothing to review against. If only one exists, say which is
missing and proceed.

The output is **a reviewed, green, committable working tree and a guide for reviewing it** — not a
commit. You never commit.


## Phase 0 — Start from green

Ask the user to confirm the state of the code to be reviewed is green: all tests pass and the full
verification pipeline passes.


## Phase 1 — Read the diff and reconstruct intent

Scope is **the changes the plan and work summary account for**. Everything else pending in the tree
is listed, not reviewed.

Run `git status --short` and `git diff HEAD` over the plan-related paths. Untracked files the plan
created will not appear in `git diff HEAD` — read them in full directly.

Partition, and say so before reviewing:

| Bucket | Treatment |
|---|---|
| Files the plan/work summary account for | reviewed, line by line |
| Files pending in the tree that they do not | named, flagged as riding along, not reviewed |

Reconstruct intent: for each reviewed file, map it to the work summary claim and the plan sub-task
that owns it. A diff you cannot map to a sub-task is itself a finding.

Read the file's *surroundings*, not just its diff. Most consistency findings come from neighbouring
code that did not change.


## Phase 2 — Review, adversarially

**The judgement is yours, inline, in this session.** Do not delegate verdicts to a subagent — it
carries the same blind spot and will confirm the same mistakes with more words.

The difficulty is that you likely wrote this code, and your default posture is to confirm it. Invert
it deliberately: for each change, the question is not "is this reasonable?" but **"what input,
ordering, or environment makes this wrong?"** Try to produce a concrete failing case. A finding
without one is a smell, not a finding.

Re-read the relevant codebase standards and domain docs before judging against them.

### The lenses

**Code quality, simplicity, consistency.** Does it look like the code around it? Is there a simpler
shape? Abstractions introduced for one caller are suspect. New patterns that duplicate existing ones
under a different name are suspect.

**Dependencies and direction.** Does anything new point the wrong way — inner layer reaching
outward, domain type importing infrastructure, shared module depending on a caller? Look for cycles.

**Claims made in comments are findings to verify.** Any comment asserting something is
impossible, unreachable, or always/never true is a factual claim — check it against the code. A
false one is a finding even when the behaviour is right.

**Bugs.** For what this change actually touches: decorators or wrappers that silently drop methods;
cache key collisions; parsing of external content (encoding, relative vs absolute URLs, null paths);
error paths and what the user sees when things fail.

**Test quality and coverage.** For each behaviour added, is there a test that fails if you break
it? Delete a line mentally and ask which test goes red. Check that teardown actually cleans up, and
that no mutable fixture is shared between spec files.

**Security.** External or user-supplied strings reaching the DOM; personal data in committed test
fixtures (check the whole fixture, not just the block the tests read); secrets or tokens in config
files; what gets written to persistent storage.

### Cross-check with review tools

Run these **after** the lenses above, never before — running first anchors you to the tool's output
and defeats the adversarial posture this phase requires.

> Add project-specific review commands here.

Four rules for folding the output in: scope it to what the plan accounts for; verify every finding
against the real files before believing it; deduplicate with what the lenses already caught; do not
paste the reports into the chat.


## Phase 3 — Triage, then stop

Rank every finding. **High-priority** means it would break behaviour, lose or expose data, hide a
regression, or make a reviewer reject the PR. Everything else is not high-priority.

Verify every finding against the real files before reporting it. Drop anything you cannot reproduce.

**The exact message sequence:**

**Message 1 — the overview, with NO ask.** Two lists:
1. High-priority findings, each with: file:line, what is wrong, the concrete failing case, and the
   proposed fix. Followed by a fix plan — ordered, with the verification each fix needs.
2. Noted, not actioned — everything lower. Named, one line each, no fix plan.

End it by saying nothing is being changed yet, and **stop**. No question attached.

**Message 2 — finding 1, and only finding 1.** Where it came from, why it is wrong, the failing
case, the fix's size and blast radius. One ask: what to do about this finding. Then wait.

**Then: settle it, fix it, verify it — and only then move to finding 2**, in its own message, same
shape.

Change nothing until the user approves a fix. Each finding gets its own message; never bundle them.


## Phase 4 — Fix

Execute the approved plan only. Nothing from list 2 gets touched, and no cleanup gets folded in —
the diff must not grow during its own review.

Verify using the same tier discipline as the execution skill. Read it from disk before relying on
it: `skills/coding-task-execution/SKILL.md`. Do not reconstruct tiers or commands from memory.

A fix that does not go green is not done. If a fix turns out wider than it looked, stop and say so.

Re-running the full pipeline after the fixes is the user's call — ask, state why it is needed, and
wait. That request is its own message.


## Phase 5 — The PR-review guide

Write it for a human about to read the diff cold, with no prior knowledge of this task. The
reviewer should understand what the change is for and what to look for *before* opening a single
file.

Persist it to `coding-tasks/<slug>/review.md` — a new file in the
same `<slug>` directory, not appended to `work-summary.md`. Post a condensed version in the chat
and end with the link to the document. That message carries no ask.

Draw intent from the persisted files, not from memory:

| Source | What to draw from it |
|---|---|
| `discovery.md` | bigger picture, domain vocabulary, live-source facts, contradictions found |
| `plan.md` | goal, out-of-scope, decision table with whys, research findings, assumptions |
| `work-summary.md` | per file why it changed, verification evidence, items needing human review |

The document carries:

- **What this change does** — a short paragraph in the terms of the app, not the code.
- **Decisions the reviewer would otherwise question** — the plan's decision table condensed to what
  is visible in the diff, each with its *why* and the alternative rejected. Deferred scope goes here
  so the reviewer does not flag what was deliberately not built.
- **Verification status** — the final pipeline run and outcome, dated.
- **Findings** — high-priority ones with their resolution; noted-not-actioned ones as a list,
  including why they were dismissed.
- **What to review, in what order** — a numbered reading order that makes the change
  comprehensible. Each step carries the tests that cover it; tests are not a step of their own
  unless no single implementation file owns them.
- **Walkthrough in reading order** — per step, per file: what it is, what changed and why, what to
  look for. Each implementation step includes its related tests directly beneath it.
- **What to check** — things a human should verify that tests do not: live source values, config
  per environment, judgement calls, items flagged in the work summary.
- **Riding along** — pending files from Phase 1 that were out of scope.


## Phase 6 — Suggest the commit message

Propose the text; **do not commit and do not stage**. Cover the whole set of pending changes
including the ones that rode along. Body explains *why*; the diff already shows *what*.


## Phase 7 — Retrospect

Review how phases 0–6 actually went and fold anything durable back into this skill. Corrections the
user made are the highest-value input — especially findings you missed that they caught, and
findings you raised that turned out to be noise. Ask the user if they want to add any other
improvement to this skill.

**Where the improvement goes — check before editing anything.** Phase logic belongs in this file.
Interaction protocol rules belong in the shared interaction-protocol file.


## Phase 8 — Close

The working tree is reviewed, green and committable, and the commit message is proposed. You do not
commit — staging and committing are the user's.



## Standing rules
- **Git** Stay on the current branch. No commits, no staging, no stash, no revert, no clean. The
  working tree stays exactly as the user left it, plus the approved fixes.
- **Scope** The plan defines what is reviewed; the approved fix plan defines what is changed.
  Review is not the moment to improve unrelated code.
- **Honesty over reassurance.** A review that finds nothing is a review that was not performed. If
  a change genuinely is clean, say what you checked and why it holds.
- **Commands against live environments** Read-only commands are permitted. Mutating commands are
  not permitted during review.


## Anti-patterns

- Reviewing on top of a red or unverified build.
- Reporting findings not verified against the real files.
- Confirming your own earlier work instead of attacking it.
- Fixing lower-priority findings, or folding in cleanups, during the fix phase.
- Starting fixes before the user approves the fix plan.
- Committing or staging anything.
- Ending the two-list overview with a question.
- Moving to the next finding before the current one is decided, fixed and verified.
- Bundling permission for a pipeline run with any other decision.
- Writing a PR guide that restates the diff instead of directing attention within it.
- Writing a PR guide that explains *what* changed but not *why*, when the persisted files hold that
  intent and were not read for it.
- Parking every spec in a final "the tests" step instead of presenting each spec under the
  implementation step it covers.
- Appending the PR guide to the work summary instead of giving it its own file.
- Pasting the whole guide into the chat, or ending the final message with anything other than the
  link to it.
- Spawning subagents; this protocol is done inline unless the user asks otherwise.
