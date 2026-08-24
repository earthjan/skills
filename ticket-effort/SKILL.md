---
name: ticket-effort
description: "Estimate story points, complexity, reasoning effort (low/medium/high/xhigh/max), the cheapest sufficient Claude model (Haiku 4.5 / Sonnet 5 / Opus 5), and whether a task needs multi-agent. Use when a ticket, GitHub issue, feature request, bug report, task description, or spec is presented and needs cost-aware routing. Biased toward Haiku and Sonnet — harness engineering (docs, seams, TDD, reviewer subagents) does the reasoning work a bigger model would otherwise have to do alone. Never spend max effort or Opus on a trivial task."
argument-hint: "Ticket, issue link, or task description to assess"
user-invocable: true
disable-model-invocation: false
---

# /ticket-effort — Estimate Effort, Reasoning, Model, and Agent Topology

## Outcome

For any incoming ticket or task, produce a single verdict block that answers four questions:

1. **Story points** — 1 / 2 / 3 / 5 / 8 / 13 (13 = must be split).
2. **Complexity** — trivial / simple / moderate / complex / very complex.
3. **Reasoning effort** — `low` / `medium` / `high` / `xhigh` / `max`.
4. **Model** — which Claude model to run the task on, plus **multi-agent** yes/no.

The verdict is driven by a 6-dimension score (0–18). Everything is evidence-based: cite the ticket's own wording, the files/specs it touches, and the project's docs it implicates (when the project has them).

## First Principle: Cheapest Sufficient Model, Biased Toward Haiku/Sonnet

Cost and latency scale with reasoning effort and model tier. The default answer is **Sonnet 5 at `medium`** for anything that needs real judgment, and **Haiku 4.5 at `low`** for anything mechanical and well-specified. You must justify every step up past Sonnet — Opus 5 is not a default, it's an escalation that needs a stated reason.

This bias is deliberate, not just cost-cutting: **harness engineering absorbs most of the reasoning load a bigger model would otherwise have to supply alone.** Confirmed seams, TDD's red→green loop, canonical `CONTEXT.md` docs, the ticket registry, and dedicated reviewer subagents (`tech-lead-review-*`, `ux-reviewer`, `goal-satisfaction-reviewer`, `dev-showcase-reviewer`) turn "does this model reason well enough on its own" into "does this model execute well inside a structure that already did the hard thinking." Haiku and Sonnet are strong at the latter. Reach for a better harness (a `ship-*` pipeline, a confirmed seam, an extra reviewer subagent) before reaching for a bigger model — it's usually both cheaper and more reliable, because independent review catches what raw model strength alone would have to get right on the first pass.

When between two tiers, pick the cheaper/smaller one. The user's reason effort budget is the boss; you are the advisor.

## Model Roster

| Model | Agent `model` value | Sweet spot | Cost tier |
|---|---|---|---|
| Haiku 4.5 | `haiku` | cheap default — mechanical, well-specified, single-seam changes; copy/config tweaks; boilerplate that follows an existing pattern verbatim | cheap |
| Sonnet 5 | `sonnet` | default workhorse — most feature work, moderate-to-complex logic, standard reviews, UI/UX build-out; this session's own default model | mid |
| Opus 5 | `opus` | reserved — genuinely ambiguous specs, undiagnosed hard bugs, architecture decisions, security/financial-integrity design calls. Never the default; always justify in the rationale. | premium |
| Fable 5 | `fable` | creative/narrative writing model — not relevant to engineering tickets. Only surfaces if a ticket is copy/narrative-heavy with no code seam (e.g., drafting long-form UX copy). Out of scope for most routing decisions here. | niche |

All four models are multimodal (image + text) — there is no text-only vs. image-capable split to route around, unlike some other model families. A screenshot or wireframe in the ticket does not by itself force a tier change; route on complexity as usual.

## Score the Ticket: 6 Dimensions (0–3 each)

Score honestly and from evidence. When evidence is thin, take the **lower** score and note the assumption.

| # | Dimension | 0 | 1 | 2 | 3 |
|---|---|---|---|---|---|
| 1 | **Scope** — layers & files touched (per the project's layer/architecture model: UI → logic → data, if it has one) | single file / config / copy change | 1–2 files inside one layer | thin vertical slice across 2–3 layers | cross-module (multiple features/services), or schema + migration + indexes |
| 2 | **Domain rule novelty** — new logic vs. reusing an existing pattern | reuses existing function/pattern verbatim | minor extension (new enum member, extra param) | new business rule with edge cases (date math, derived state, ordering) | new domain concept or invariants spanning modules (money, obligations, integrity) |
| 3 | **Uncertainty** — how well-specified the ticket is | fully specified; ACs cover it; no open questions | minor gaps resolved by reading the project's canonical docs | significant ambiguity → needs a design decision or user confirmation | requirements conflict, missing spec, or requires research |
| 4 | **Data integrity & risk** | no persisted state | reads / local state only | writes to existing data, additive, covered by existing rules | schema change / migration, financial integrity, auth/security, destructive operations |
| 5 | **UI/UX depth** | no UI (pure logic) | reuses the project's existing components; one screen; standard states | new screen or significant interaction; loading/empty/error; a11y; copy | complex interaction, animation, design-system extension, multi-state flows, UX research |
| 6 | **Verification surface** | no tests needed | 1–3 unit tests on one seam | unit + integration across several seams, edge cases | large test matrix, timing/concurrency, security rules, e2e |

**Total = sum of six (0–18).**

## Story Points (Fibonacci)

| Total score | Points | Complexity label |
|---|---|---|
| 0–3 | 1 | trivial |
| 4–6 | 2 | simple |
| 7–9 | 3 | moderate |
| 10–12 | 5 | complex |
| 13–15 | 8 | very complex |
| 16–18 | **split** | epic — do not assign 13; recommend splitting and give per-slice estimates |

## Reasoning Effort

Effort measures **thinking depth required**, not volume of code. A big-but-mechanical batch of boilerplate scores medium effort and high points.

| Effort | Condition | Typical |
|---|---|---|
| `low` | total ≤ 3 AND every dimension ≤ 1 | copy/config tweak, trivial bug with clear repro, single-file change on a confirmed seam |
| `medium` | total 4–7, OR exactly one dimension = 2 | small vertical slice, new screen reusing components, moderate bug, additive query/cache change |
| `high` | total 8–11, OR ≥2 dimensions = 2, OR exactly one dimension = 3 | new feature with new rules, additive schema change, tricky logic, UI/UX build-out |
| `xhigh` | total 12–15, OR ≥2 dimensions = 3, OR (scope = 3 AND uncertainty ≥ 2) | cross-module feature with new domain invariants, non-trivial schema change, ambiguous UX flow needing a design decision |
| `max` | total ≥ 16, OR uncertainty = 3, OR (data integrity = 3 with financial/security impact) | architecture refactor, hard undiagnosed bug, irreversible or high-stakes decision |

## Model Routing (decision order)

Evaluate top-down; first match wins.

1. **effort = `max`** → **Opus 5**, effort `max`. This is the only tier where Opus is the default — state the specific reason in the rationale (true architecture ambiguity, undiagnosed hard bug, financial/security-integrity design call). Never route here on total score alone if the individual `max` triggers (uncertainty=3, integrity=3-with-impact) aren't actually present.
2. **effort = `xhigh`** → **Sonnet 5**, effort `xhigh`, by default. Escalate to **Opus 5** at `xhigh` only when uncertainty is *also* ≥2 — i.e., high complexity compounded with real ambiguity, not complexity alone. State the reason if you escalate.
3. **effort = `high`** → **Sonnet 5**, effort `high`. This is the default workhorse tier for genuine feature work — new rules, additive schema, tricky logic, UI/UX build-out.
4. **effort = `medium`** → **Sonnet 5**, effort `medium` (this session's own default), unless the task is narrow and mechanical (scope ≤1 AND domain novelty ≤1 — e.g., a repeated pattern applied to a new file) → then **Haiku 4.5**, effort `medium`.
5. **effort = `low`** → **Haiku 4.5**, effort `low`. This is the cheap default for anything simple and well-specified — copy, config, single-file changes on a confirmed seam.

**Tie-breaks:**
- Haiku vs. Sonnet at the same effort tier: prefer **Haiku** when the task sits on a confirmed seam with an existing pattern to follow (domain novelty ≤1). Prefer **Sonnet** when it introduces new-but-bounded logic (domain novelty = 2) even if the total score is otherwise low — new business rules deserve Sonnet's judgment even in a small ticket.
- Sonnet vs. Opus at `xhigh`: prefer **Sonnet** unless uncertainty is genuinely high — "this touches a lot of files" is not by itself a reason to escalate; "the spec conflicts with itself" or "the right design isn't decided yet" is.
- A ticket that names a screenshot/wireframe/design mock does **not** by itself force Opus or even Sonnet — all four Claude models can read images. Route on complexity, not modality.

## Multi-Agent Decision

Single agent is the default. Go multi-agent only when:

- **Full feature pipeline** — the task is a shippable feature and the project has a plan → implement → review → walkthrough flow (e.g., this repo's `ship-ui` / `ship-non-ui` / `ship-dev-showcase` skills, which dispatch dedicated reviewer subagents like `tech-lead-review-architecture-enforcer`, `tech-lead-review-code-quality`, `tech-lead-review-patterns`, `tech-lead-review-tests`, `ux-reviewer`, `goal-satisfaction-reviewer`, `dev-showcase-reviewer`) that would add value.
- **Independent review matters** — financial integrity, auth/security, or UX quality bar is high; an independent second set of eyes is worth the cost. Prefer this over escalating the implementer's own model tier — a Sonnet implementer plus a Sonnet reviewer subagent usually beats a lone Opus pass, because the reviewer is genuinely independent rather than the same context re-checking itself.
- **Parallelizable workstreams** — two or more independent slices can proceed concurrently (research + implementation; two modules with no shared seams).
- **Confidence is low AND stakes are high** — effort ≥ `high` with uncertainty = 2 or 3: parallel exploration or a plan-first pass before committing.

For anything ≤ 5 points, single agent unless a full feature pipeline is explicitly requested.

## Procedure

1. **Identify the ticket type** — bug / feature / chore / design / refactor / research. Read the ticket body, linked issue, or spec; if it's a GitHub link, fetch it.
2. **Score the 6 dimensions** 0–3 with cited evidence (ticket wording, files it names, docs it implicates — the project's README, CONTEXT docs, design system, and data model docs when present).
3. **Sum → points + complexity label.** If 16–18, recommend the split.
4. **Determine reasoning effort** from the table.
5. **Route the model** top-down.
6. **Decide multi-agent** and name the subagents if yes.
7. **Output the verdict block** below. Score all six dimensions internally with full evidence
   as you go (steps 1–6) — but compress that work into the 1–2 sentence rationale the block
   asks for. The scoring stays rigorous; only the explanation gets terse.
8. **If uncertainty = 2 or 3**, ask one clarifying question or state the assumption the estimate depends on — do not silently guess on high-stakes items.

## Output Format

Copy this block verbatim (fill the fields):

```
┌ Ticket assessment ──────────────────────────────
│ Ticket:   <id or title>
│ Type:     <bug | feature | chore | design | refactor | research>
│ Score:    n/18  (scope n · domain n · uncertainty n · integrity n · ui/ux n · testing n)
│ Points:   <1 | 2 | 3 | 5 | 8 | SPLIT>
│ Complexity: <trivial | simple | moderate | complex | very complex>
│ Effort:   <low | medium | high | xhigh | max>
│ Model:    <haiku | sonnet | opus | fable>
│ Multi-agent: <no | yes — <which subagents / which pipeline>>
│ Confidence: <high | medium | low>
│ ─ Rationale ─
│ <1–2 sentences: the single deciding factor, not a per-dimension walkthrough>
└─────────────────────────────────────────────────
```

The rationale is a compression, not a proof — its job is to let the user build their own
judgment for the next ticket, not to demonstrate the scoring work. Name the one or two
things that actually moved the verdict (the dimension that set the ceiling, an assumption
the estimate depends on) and stop. Do not restate all six dimension scores in prose — the
`Score:` line already has them. If the honest rationale genuinely needs more than two
sentences (rare — e.g. two dimensions pulling in opposite directions), say so briefly rather
than defaulting to a long form.

## Rules & Guardrails

- **Never** route a trivial task to `max` effort or Opus. The default is Sonnet 5 at `medium`, or Haiku 4.5 at `low` for mechanical work.
- **Opus is an escalation, not a tier.** Every Opus recommendation must name the specific trigger (genuine ambiguity, undiagnosed hard bug, architecture decision, financial/security integrity call) — "this is complex" is not sufficient on its own.
- When between two tiers, pick the cheaper/smaller one.
- Effort measures thinking depth, not lines of code.
- Model values must match the Agent tool's `model` enum exactly (`haiku` / `sonnet` / `opus` / `fable`); effort values must match the harness's effort enum (`low` / `medium` / `high` / `xhigh` / `max`).
- Override the estimate only with an explicit user reason (e.g., "ship it fast anyway", "this is actually a two-day job").
- **Re-assess mid-task:** if reality diverges from the estimate (task is materially harder or easier), re-run the scoring, and say so — switching models/effort mid-flight is cheaper than finishing on the wrong tier.
- The user is the final call; you recommend, they decide.

## Quality Checklist

- [ ] Every dimension score traces to concrete evidence (ticket wording, file paths, doc references) — even though only the deciding one or two surface in the rationale.
- [ ] The rationale is 1–2 sentences and names the actual deciding factor, not a per-dimension recap.
- [ ] Model choice is the cheapest sufficient option, not the best available — Opus only with a stated trigger.
- [ ] Effort value is one of `low` / `medium` / `high` / `xhigh` / `max`.
- [ ] Multi-agent answer names the specific subagents/pipeline, or is a plain `no`.
- [ ] Assumptions and low-confidence calls are stated explicitly, even inside the short rationale.
