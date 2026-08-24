---
name: harness-engineering
description: Condensed reference on harness engineering — how to structure a repo's instructions, state, scope control, verification, observability, and multi-session/multi-agent loops so agent sessions stay effective over time. Load when designing or auditing a CLAUDE.md/AGENTS.md entry file, setting up a new project's agent harness, deciding whether something belongs in the entry file vs. a topic doc vs. a lint rule, designing PROGRESS.md/DECISIONS.md or feature-list tracking, deciding whether a repeated agent mistake needs a rule or an automated check, or considering whether a task justifies a loop (/loop, scheduled agent) or a multi-agent graph (Workflow tool) over a single session.
---

# Harness Engineering

Condensed reference from Walking Labs' "Learn Harness Engineering" lecture series (14 lectures). This is a synthesis to apply judgment against, not a spec to follow verbatim — treat every rule below as a default with a stated reason, not a mandate. When a project's own CLAUDE.md or skills already implement a piece of this (e.g. this project's TDD/gates skill already is the "three-layer termination check" from §5), defer to the project's existing practice instead of re-recommending the generic version — don't contradict working local conventions.

Full lecture index is in [lectures.md](lectures.md) if a section below needs more depth than the condensed form gives.

## 1. Instructions — entry file + topic docs

**Fresh-session test**: open a brand-new session with only the repo. Can it answer — what is this, how do I run it, how do I verify it, what's the current progress, what are the hard constraints? Any blank answer is a hole in the map.

- Entry file (`AGENTS.md`/`CLAUDE.md`) stays **50–200 lines**: project overview, quick-start commands, ≤15 non-negotiable hard constraints, links to topic docs. Nothing else belongs here.
- **Why short matters**: "lost in the middle" — LLMs under-attend to the middle of long context, so a rule at line 300 of 600 gets ignored regardless of importance. A bloated file also eats context budget before the agent opens a single source file.
- **The trap**: "agent fails → add a rule to the entry file → repeat" is the exact vicious cycle that produces 600-line files with declining success rates. Before adding a rule, ask if it belongs in a topic doc, a lint rule, or a test instead. Give every instruction a source, an applicability condition, and an expiry condition — audit and prune like dependencies.
- Topic docs: 50–150 lines each, placed next to the code they govern (e.g. `src/api/ARCHITECTURE.md`), read only when the agent is actually touching that area.

Template: [templates/entry-file.md](templates/entry-file.md).

## 2. Environment

The subsystem most instruction-only harnesses skip. Make the runtime itself self-describing: locked dependencies (`package.json`/`pyproject.toml`), pinned runtime version (`.nvmrc`/`.python-version`), reproducible environment (Docker/devcontainer).

**Test**: can a fresh session run the setup command with zero manual fixes? If not, sessions burn time on environment archaeology before ever reaching the docs.

## 3. State & initialization

Init is its own session, not folded into feature 1. Session 1 produces zero business code — only a runnable environment, one passing example test, a startup-readiness checklist, and a task breakdown. The time this costs is recovered within 3–4 later sessions.

- `PROGRESS.md` and `DECISIONS.md` exist because the *why* behind a choice is what compaction/reset loses — code survives, reasoning doesn't, unless it's written down.
- Git commits are free checkpoints — commit after each atomic unit, message says what and why.
- **Context anxiety is model-specific**: e.g. Sonnet 4.5 needed hard resets to avoid rushed-finish behavior near context limits; Opus 4.5 tolerated compaction fine. Don't assume one strategy transfers across models — re-check against whatever model is actually running.
- **Session exit checklist**, all five required: build passes, tests pass, progress file updated, no stale debug artifacts, standard startup path still works.

Template: [templates/progress.md](templates/progress.md).

## 4. Scope control — WIP=1 and feature lists

- **WIP=1 as default**: one task active at a time. Don't let the agent "also refactor while it's in there" — the single biggest driver of overreach.
- **Overreach and under-finish are the same failure, not two**: lines of code touched correlates weakly negatively with features actually completed.
- Every feature-list entry needs the **triple**: behavior description + executable verification command + state (`not_started`/`active`/`blocked`/`passing`). Missing any one and it's a memo again, not a harness primitive.
- The agent shouldn't be able to flip a feature to `passing` itself — only a passing verification command can (**pass-state gating**).

Template: [templates/feature-list.md](templates/feature-list.md).

## 5. Verification & completion

- Agents are systematically overconfident about their own completion (measured calibration bias) — "looks done" is a bias to correct for, not evidence.
- **Three-layer termination check, no skipping**: syntax/static → runtime/behavior → system/end-to-end. Passing layer 1 doesn't authorize claiming layer 3.
- Unit tests are structurally blind to interface mismatches, state-propagation errors, and resource leaks — mocking is exactly what hides these. Only end-to-end exercises the real wiring.
- **Completion priority constraint**: no refactoring or style polish until core functionality is verified.
- Error messages for the agent should say what's wrong, why, and specifically how to fix it — not just "test failed."

## 6. Observability

Two distinct layers, both needed:
- **Runtime observability** (logs/traces/health) — what did the system do.
- **Process observability** (sprint contracts, rubrics) — why should this be accepted.

- **Sprint contract**: scope + verification standard + explicit exclusions, negotiated *before* the generator starts — prevents building something the evaluator was always going to reject.
- **Evaluator rubric**: turn "good or not" into dimension-by-dimension scoring so different evaluator runs converge instead of vibes.
- Missing observability burns 30–50% of session time re-diagnosing state that was never recorded.

## 7. Harness upkeep

- **Quality document**: a living per-module score, not a one-off audit — new sessions read it and know where to prioritize first.
- **Harness simplification, monthly**: pick one component, disable it, run a benchmark task. If nothing degrades, remove it for good — a constraint load-bearing for one model generation can be dead weight for the next.
- "Add a rule" isn't the only response to a recurring mistake — turning a repeated review comment into an automated lint/test check is usually the better fix, and it's what keeps the entry file from growing back.

**The test that actually matters**: if you stepped away for 8 hours mid-task, would it still be making progress, or just sitting there waiting on your next message? First = loop. Second = you're still the loop, however good your multi-agent split is.

## 8. Loop engineering

- Multi-agent review (implementer + independent reviewer) is necessary but not sufficient to make something a "loop" — that split is just the *Sub-agents* primitive, one ingredient among six.
- `/goal` = goal + verification method + stopping condition, given once, then the system runs unattended until met or budget runs out.
- **Six primitives**, not all needed every time: Automations (the trigger), Worktrees (parallel isolation), Skills (pre-packaged context), Connectors (touches real tools/tickets/Slack), Sub-agents (maker ≠ checker), External State (memory on disk — the spine the other five depend on).
- **Four silent costs** that grow the longer a loop runs unattended: verification debt (skipped checks), comprehension rot (your own understanding of the codebase drifts), cognitive surrender (you stop having opinions on the output), token blowout (context grows ~quadratically without compaction).
- **Maturity ladder**: L1 goal runner → L2 scheduled single-task → L3 maker/checker split → L4 self-feeding (finds its own next task) → L5 fleet orchestration. Most real teams sit at L2–L3; L1 is the fastest path to seeing any return at all.

## 9. Graph engineering

> Take this loosely. The course itself flags "Graph Engineering" as largely a July-2026 rebrand of things that already existed for years (LangGraph since 2024, Anthropic's Dec-2024 agent patterns, classic state machines/DAG schedulers) — and some of the viral adoption stats behind the trend were fabricated. The underlying ideas are real; the branding isn't load-bearing.

Why move beyond a single loop at all — three structural failures a loop can't fix from inside itself:
- **Goodhart** — optimizing a metric detaches it from the outcome it stood for.
- **Blindness upward** — a loop can't ask "is this even the right goal."
- **Conflict** — independent loops optimizing different things quietly undermine each other.

- **Graph** = nodes (can be full agents, not just deterministic functions) + edges (can route dynamically off a verification result, not just hardcoded) + shared state + explicit routing rules. A workflow is the fully-deterministic special case of a graph.
- **Five criteria, need at least 3** to justify building one: independently decomposable work, real branch/rollback paths worth naming, intermediate state worth checkpointing, every node has an explicit verifiable "done," coordination benefit clearly exceeds coordination cost.
- **Orchestration tax**: starting parallel agents is cheap; reconciling and reviewing their output is not. You are the one serial resource in the system — a graph runs more agents in parallel, it does not parallelize your judgment.

## Vocabulary bridge

| This model's layer | Course concept | Lecture(s) |
|---|---|---|
| Always-loaded instructions | Instructions (entry file) | 1, 4 |
| On-demand instructions | Instructions (topic docs) | 3, 4 |
| Session state | State | 5, 6, 12 |
| *(no slot)* | Environment | 2 |
| Automation / tests | Feedback | 2, 9, 10 |
| *(implicit)* | Tools | 2 |
| Multi-agent review | Sub-agents | 9, 11, 13 |
| Loop engineering | Autonomous triggering + external state | 13 |
| Graph engineering | Nodes + edges + shared state | 14 |
