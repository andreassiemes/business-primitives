# Business Primitives

> Your information is too important for PowerPoint.

**Business Primitives** defines the atomic building blocks of business communication — decisions, metrics, risks, actions, timelines, statuses, stakeholders — as structured, format-agnostic, composable units.

Inspired by Brad Frost's [Atomic Design](https://atomicdesign.bradfrost.com/), **Atomic Business Design** applies the same layered composition model to business information:

| Level | Atomic Design (Web) | Atomic Business Design |
|-------|---------------------|----------------------|
| **Atoms** | Button, Label, Input | Decision, Metric, Risk, Action, Timeline, Status, Stakeholder |
| **Molecules** | Search Form, Card | Argument, Comparison, Status Update, Progress |
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

### Atoms (7)

| Primitive | Description | Key Fields |
|-----------|-------------|------------|
| **Decision** | Who decided what, when, why | decided_by, title, decided_date, status, rationale |
| **Metric** | Number with context and trend | name, value, unit, trend, period, target |
| **Risk** | Assessed risk with mitigation | threat, probability, impact, owner, mitigation |
| **Action** | Task with ownership | owner, title, due_date, status, priority |
| **Timeline** | Milestone with dependencies | milestone, target_date, owner, dependencies, status |
| **Status** | State assessment of an entity | entity, state, detail, updated_at |
| **Stakeholder** | Person with influence and stance | name, role, influence, stance, interests |

### Molecules (4)

| Primitive | Description | Composition |
|-----------|-------------|-------------|
| **Argument** | Claim + Evidence + Implication | claim, evidence[], implication, confidence |
| **Comparison** | Options + Criteria + Recommendation | question, options[], criteria[], recommendation |
| **Status Update** | Metric + Trend + Assessment | metric, trend, assessment, action_needed, actions |
| **Progress** | Checkpoint + Velocity + Forecast | checkpoint, milestones[], metrics[], next_actions[] |

## Spec

The full specification lives in [`spec/bp-v1.0.md`](spec/bp-v1.0.md). It defines:

- 7 design principles
- 7 atoms and 4 molecules with field-level definitions
- 4 organism patterns and 3 templates
- A composition model (inline, reference, hybrid)
- 26 validation rules and 4 conformance levels
- A shared enum registry and cross-reference system
- Connection to the [OPI Spec](https://org-as-code.com)

A JSON Schema for automated validation is at [`spec/bp-v1.0.schema.json`](spec/bp-v1.0.schema.json).

## Structure

```
spec/
├── bp-v1.0.md              # Main specification
├── bp-v1.0.schema.json     # JSON Schema
├── atoms/                   # 7 atom definitions (YAML)
│   ├── action.yaml
│   ├── decision.yaml
│   ├── metric.yaml
│   ├── risk.yaml
│   ├── stakeholder.yaml
│   ├── status.yaml
│   └── timeline.yaml
└── molecules/               # 4 molecule definitions (YAML)
    ├── argument.yaml
    ├── comparison.yaml
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

## Status

**V1.0** — Foundation release. 7 atoms, 4 molecules, 4 organisms, 3 templates. JSON Schema. 26 validation rules. Website live at [businessprimitives.com](https://businessprimitives.com).

## Author

**Andreas Siemes** — [andreassiemes.de](https://andreassiemes.de) · [Business Primitives](https://businessprimitives.com) · [Org as Code](https://org-as-code.com) · [LinkedIn](https://linkedin.com/in/andreassiemes)

## License

MIT
