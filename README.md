# Business Primitives

> Your information is too important for PowerPoint.

**Business Primitives** defines the atomic building blocks of business communication — decisions, metrics, risks, actions, timelines, statuses, stakeholders — as structured, format-agnostic, composable units.

Inspired by Brad Frost's [Atomic Design](https://atomicdesign.bradfrost.com/), **Atomic Business Design** applies the same layered composition model to business information:

| Level | Atomic Design (Web) | Atomic Business Design |
|-------|---------------------|----------------------|
| **Atoms** | Button, Label, Input | Decision, Metric, Risk, Action, Timeline, Status, Stakeholder, Objective |
| **Molecules** | Search Form, Card | Argument, Comparison, Status Update, Progress, OKR, Briefing |
| **Organisms** | Header, Product Grid | Executive Summary, Risk Register, Action Plan, Performance Dashboard |
| **Templates** | Product Page Layout | Decision Paper, Quarterly Review, Weekly Status |
| **Pages** | Actual Product Page | Q1 2026 Review, Project X Decision Paper |

## Why

Every organization communicates with the same building blocks. But nobody has defined them.

- **Format silos** — Information is trapped in PowerPoint, Word, Excel, Miro. Not reusable. Not machine-readable.
- **Inconsistency** — Every presentation is a snowflake. No design system. No standards.
- **Low information density** — Slides full of bullet points. No visual design. No data-ink ratio.
- **Person-dependent** — Only the creator understands the slide. No structured data behind it.
- **AI-blind** — LLMs can barely parse PowerPoint. Structured primitives they can.

## The Primitives

### Atoms (8)

| Primitive | Description | Key Fields |
|-----------|-------------|------------|
| **Decision** | Who decided what, when, why | decided_by, title, decided_date, status, rationale |
| **Metric** | Number with context and trend | name, value, unit, trend, period/as_of, target |
| **Risk** | Assessed risk with mitigation | threat, probability, impact, owner, mitigation |
| **Action** | Task with ownership | owner, title, due_date, status, priority, completed_date |
| **Timeline** | Milestone with dependencies | milestone, target_date, owner, dependencies, status |
| **Status** | State assessment of an entity | entity, state, detail, updated_at, previous_state |
| **Stakeholder** | Person with influence and stance | name, role, influence, stance, interests |
| **Objective** | Declared goal with timeframe | title, timeframe, owner, status, success_criteria |

### Molecules (6)

| Primitive | Description | Composition |
|-----------|-------------|-------------|
| **Argument** | Claim + Evidence + Implication | claim, evidence[], implication, confidence |
| **Comparison** | Options + Criteria + Recommendation | question, options[], criteria[], recommendation |
| **Status Update** | Metric + Trend + Assessment | metric, trend, assessment, action_needed, actions |
| **Progress** | Checkpoint + Velocity + Forecast | checkpoint, milestones[], metrics[], next_actions[] |
| **OKR** | Objective + Key Results | objective, key_results[], overall_progress, confidence |
| **Briefing** | Recommendation-first executive summary | subject, situation, significance, recommendation, urgency |

## Spec

The full specification lives in [`spec/bp-v1.1.md`](spec/bp-v1.1.md) (current) and [`spec/bp-v1.0.md`](spec/bp-v1.0.md) (foundation). V1.1 defines:

- 7 design principles
- 8 atoms and 6 molecules with field-level definitions
- 5 organism patterns and 5 templates
- A composition model (inline, reference, hybrid, collection)
- 29 validation rules and 4 conformance levels
- Rendering hints for display tools
- A shared enum registry and cross-reference system
- Bidirectional connection to the [OPI Spec](https://org-as-code.com)

JSON Schemas for automated validation: [`spec/bp-v1.1.schema.json`](spec/bp-v1.1.schema.json) | [`spec/bp-v1.0.schema.json`](spec/bp-v1.0.schema.json)

## Structure

```
spec/
├── bp-v1.1.md              # Current specification
├── bp-v1.1.schema.json     # JSON Schema (V1.1)
├── bp-v1.0.md              # Foundation specification
├── bp-v1.0.schema.json     # JSON Schema (V1.0)
├── atoms/                   # 8 atom definitions (YAML)
│   ├── action.yaml
│   ├── decision.yaml
│   ├── metric.yaml
│   ├── objective.yaml
│   ├── risk.yaml
│   ├── stakeholder.yaml
│   ├── status.yaml
│   └── timeline.yaml
└── molecules/               # 6 molecule definitions (YAML)
    ├── argument.yaml
    ├── briefing.yaml
    ├── comparison.yaml
    ├── okr.yaml
    ├── progress.yaml
    └── status-update.yaml
examples/                    # Real-world document compositions
├── decision-paper.yaml
└── quarterly-review.yaml
templates/                   # Template definitions
├── decision-paper.yaml
├── quarterly-review.yaml
└── weekly-status.yaml
```

## Connection to Org as Code

Business Primitives is the communication layer. [Org as Code](https://org-as-code.com) is the structural layer. Together, they make organizations legible — their structures *and* their information.

A decision in OPI (Organizational Programming Interface) is simultaneously a Decision Primitive. The systems share the same data.

## Rendering with Prism

[Prism](https://github.com/andreassiemes/bp-prism) is the open-source rendering engine for Business Primitives. Write your content as YAML, render it as HTML in five profiles: Card, Slide, Doc, Table, Board.

```bash
python prism.py build --profile card examples/decision-paper.yaml
```

## Status

**V1.1** (current) — 8 atoms, 6 molecules, 5 organisms, 5 templates. New: Objective, OKR, Briefing. Rendering hints. Collection documents. 29 validation rules. Backward compatible with V1.0.

**V1.0** — Foundation release. 7 atoms, 4 molecules, 4 organisms, 3 templates. JSON Schema. 26 validation rules.

Website live at [businessprimitives.com](https://businessprimitives.com).

## Author

**Andreas Siemes** — [andreassiemes.de](https://andreassiemes.de) · [Business Primitives](https://businessprimitives.com) · [Org as Code](https://org-as-code.com) · [Prism](https://github.com/andreassiemes/bp-prism) · [LinkedIn](https://linkedin.com/in/andreassiemes)

## License

MIT
