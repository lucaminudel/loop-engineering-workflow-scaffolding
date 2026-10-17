---
name: coding-task-discovery
description: Explore the business-domain concepts and technical implementation details behind a coding task, and persist a Discovery findings report that feeds the Coding Task Inception. Use when the user says "Coding Task Discovery", or asks to explore or research a task before planning it.
---

# Coding Task Discovery

First of the four. It runs **before** `coding-task-inception` and produces its input.

The output is a **persisted Discovery findings report** — the business-domain concepts and the
technical implementation details the task depends on. Not a plan, not sub-tasks, not code. Planning
belongs to inception; if you find yourself sequencing work, you have left this skill.

Discovery is **read-only**. Read the codebase, fetch live external pages, search the web. Write
nothing into the repo tree except the report's own scratch notes, and no application code at all.



## Phase 1 — Frame the task
Ask for:

1. **The bigger picture** — which end-user feature or outcome does it serve?
2. **What it contributes to** — how does this task contribute to that? What is its part?
3. **A high-level description** of the task itself.
4. **What to explore** — which topics they want covered, which questions they want answered.


## Phase 2 — Agree the topic list

Turn what they gave you into an explicit, numbered list of topics and questions, and present it
before exploring. Separate:

- **Domain topics** — concepts, rules, actors, lifecycles, vocabulary. What the app means.
- **Technical topics** — structure, call chains, data sources, contracts, environments, testing.
  How the app is built.

Add topics and questions of your own that the framing implies but the user did not name, and mark
them as yours so they can drop them. Note the ones you judge highest-risk — the ones where being
wrong would cost the most later — and propose starting there.

Get agreement on the list before phase 3. The list is not frozen (see *Standing rules*), only
agreed.


## Phase 3 — Explore and discuss, one topic at a time

For each topic in turn: explore it, write up the findings, **present them, and wait** for the user
to react before moving to the next. They may accept it, correct it, ask for more depth, or open a
new topic on the spot. Do not batch the whole exploration and present it at the end.

A topic is done when the answers are **detailed enough to act on** — concrete values, exact paths,
named identifiers, real examples — not when they are merely plausible.


### Where to look

**What the user gave you** — first, and in full.

**The codebase** — read it rather than recalling it. Look at the implementation as well as the tests.

**Live external sources** — if the task touches an external site, `curl` the real
pages into the scratchpad and parse them.

**Your own knowledge and the web** — for anything outside the repo. Cite what you used.

**The repo's own documentation** — consult domain, technical, and verification docs as relevant.


## Phase 4 — The findings report

Maintained at `coding-tasks/<slug>/discovery.md`.
That location is readable and writable by every session and agent, and is where
`coding-task-inception` reads it from. Create the `<slug>/` directory if it does not exist.


The file carries:

- frontmatter with name, description, type
- **the framing** — the bigger picture, what it contributes toward, the high-level task description
- **the topic list**, numbered, domain and technical separated, each marked explored / open
- **per topic: the question, the finding, and the evidence** — file:line citations, URLs fetched
  with the date, selectors and values seen, commands run and their output. A finding without
  evidence is an opinion.
- **Domain concepts** — a glossary of the terms the task touches, in the app's own vocabulary, with
  the rules and the edge cases attached
- **Technical implementation details** — structure, call chains, data sources, contracts, config
  keys, environments, existing tests covering the area
- **Contradictions and gaps** — where docs disagree with code, where the external source disagrees
  with expectations, and what remains unknown
- **Open questions for inception** — decisions this discovery deliberately did not make
- a dated log of the discovery sessions, and instructions that any agent may update the file

Keep it factual. Anything shaped like "we should therefore..." is a plan and belongs to inception.


## Phase 5 — Exhaust and verify

Only when the topic list is exhausted. Before declaring that, **ask explicitly** whether there are
more topics, sub-topics, questions or follow-ups — the user often has one more once they see the
shape of what you found.

Then:

1. **Verify and double-check the findings.** Go back over the report and re-confirm each one against
   the real source, not against your notes. Cited file:line still says what you claimed; the URL
   still returns what you parsed; the config key still exists. Findings written early in a long
   session are the ones that rot. Correct what is wrong and say what you corrected.
2. **Add a summary at the top** — short: what was explored, and the headline findings. A reader
   should get the shape of it in under a minute without scrolling.
3. **Present the final document** and ask whether it is complete or whether there is more to
   explore. Wait. Expect rounds; fold each one in and re-present.


Do not hand off to `coding-task-inception` yourself. Tell the user the report is ready and where it
is; starting inception is their call.

## Phase 6 — Retrospect

Review how phases 1–5 actually went and fold anything durable back into this skill. Corrections the
user made are the highest-value input — especially topics you should have proposed and did not,
findings you asserted without evidence and had to walk back, and sources you should have checked
earlier. Ask the user if they want to add any other improvement to this skill.

**Where the improvement goes — check before editing anything.** Phase logic (the steps, their
order, what this phase produces) belongs in this file. 


## Standing rules

**The topic list is open, always.** At any point the user may add a topic, add a sub-topic, ask a
new question, or drill into an earlier answer. Take it, add it to the numbered list in the report,
and place it — either explore it now or say where it sits in the order. Never treat an addition as
scope creep, and never quietly drop one.

**Evidence over recall.** Every finding is traceable to something you actually read, fetched or ran
in this session. If you are working from memory of this codebase, go and check.

**Honesty over completeness.** "I could not determine X, and here is what I tried" is a valid
finding and belongs in the report. Filling a gap with a plausible guess is the one failure mode that
survives into the plan and then into the code.


## Anti-patterns

- Writing application code, or modifying the repo tree at all.
- Producing a plan, sub-tasks or an implementation sequence — that is inception's job.
- Describing an external data source without fetching it.
- Recording a finding without a file:line, URL or command backing it.
- Declaring a topic done at "plausible" rather than at "detailed enough to act on".
- Keeping the report only in the conversation, where the next session cannot read it.
