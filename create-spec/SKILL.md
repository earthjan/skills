---
name: create-spec
description: "Create implementation-ready GitHub Issue specs for Copilot Agent in ListaNatin. Use when refining vague feature or bug requests, asking clarification first, enforcing repository architecture constraints, and producing testable acceptance criteria with actionable tasks."
argument-hint: "Feature, bug, or initiative to convert into an agent-executable GitHub Issue"
user-invocable: true
disable-model-invocation: true
---

# ListaNatin GitHub Issue Spec (Clarify First)

## Outcome

Produce a complete GitHub Issue spec that is executable by a coding agent and aligned with ListaNatin architecture and repository constraints.

## Use When

- Writing new issue specs for implementation work.
- Converting ambiguous product requests into actionable engineering tickets.
- Preparing a Copilot Agent-ready issue with explicit scope, risks, and verification steps.

## Required Behavior

1. Clarification is mandatory before final spec generation.
2. Do not assume missing details silently.
3. Ask concise, high-value questions with recommended answers.
4. Do not cap clarification questions; ask as many as required for quality.
5. If the user declines or cannot answer, proceed using explicit assumptions and list them under Open Questions.

## Repository Guardrails To Inject Automatically

Load and apply these constraints before drafting:

- Read README architecture and Firestore source-of-truth sections first.
- Keep app/ route files as routing-only; no business logic or direct data access.
- Keep presenter/page files as wiring-only.
- Keep business logic in services/app-logic or services/core.
- Keep Firebase access within API boundaries only.
- Keep TanStack Query logic in query boundaries only.
- Keep cache policy logic only in query/cache.
- Preserve model invariants and base fields for persisted entities.

## Workflow

### Phase 1: Context Intake

1. Restate the requested goal in one sentence.
2. Identify relevant modules, files, and architecture boundaries.
3. Detect ambiguity in scope, dependencies, constraints, acceptance criteria, and risks.

### Phase 2: Clarification (Mandatory)

1. Ask focused questions needed to remove ambiguity.
2. For each question, include a recommended answer and rationale.
3. Group by decision area to reduce back-and-forth.
4. Wait for user answers before generating the final spec.

Question format:

- Question: <short question>
- Why this matters: <one line>
- Recommended answer: <specific proposal>

### Phase 3: Resolution

1. Merge user answers into the plan.
2. If answers conflict, call out the conflict and request a tie-breaker.
3. If unresolved items remain, carry them to Open Questions.

### Phase 4: Spec Generation

Generate the final issue using the strict template in this file.

### Phase 5: Quality Gate

Verify before final output:

- Scope is explicit and non-overlapping.
- Technical notes reference concrete repository files or modules when known.
- Acceptance criteria are testable and verifiable.
- Tasks are implementation-oriented and sequenced.
- Risks and edge cases are concrete.
- Open questions are explicit, or state None.

## Decision Branches

- If request is too broad:
  - Propose splitting into multiple issues and provide suggested split.
- If architecture placement is unclear:
  - Add explicit file/module placement notes in Technical Notes.
- If user cannot provide clarification now:
  - Proceed with explicit assumptions and include validation tasks.

## Output Template (Strict)

```markdown
# <Issue Title>

## Summary

<Short description>

## Background

<Context and reasoning>

## Scope

### In Scope

- ...

### Out of Scope

- ...

## Technical Notes

- Relevant files/components:
- Implementation approach:
- Dependencies:

## Acceptance Criteria

- [ ] ...

## Tasks

- [ ] Task 1
- [ ] Task 2

## Risks / Edge Cases

- ...

## Open Questions

- ...
```

## Authoring Rules

- Use clear and unambiguous language.
- Optimize for agent execution, not narrative explanation.
- Avoid overengineering.
- Explicitly label assumptions and unknowns.
- Keep criteria and tasks measurable.

## Completion Checklist

- [ ] Clarification asked first.
- [ ] Recommended answers included for key ambiguities.
- [ ] Final spec follows strict template.
- [ ] Acceptance criteria are verifiable.
- [ ] Task checklist is actionable.
- [ ] Repository guardrails are respected.
