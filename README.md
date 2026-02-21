# Business Primitives

> Your information is too important for PowerPoint.

**Business Primitives** defines the atomic building blocks of business communication — decisions, metrics, risks, arguments, timelines — as structured, format-agnostic, composable units.

Inspired by Brad Frost's [Atomic Design](https://atomicdesign.bradfrost.com/), **Atomic Business Design** applies the same layered composition model to business information:

| Level | Atomic Design (Web) | Atomic Business Design |
|-------|---------------------|----------------------|
| **Atoms** | Button, Label, Input | Decision, Metric, Risk, Action, Timeline, Status |
| **Molecules** | Search Form, Card | Argument, Comparison, Status Update |
| **Organisms** | Header, Product Grid | Executive Summary, Dashboard Panel |
| **Templates** | Product Page Layout | Quarterly Review, Decision Paper |
| **Pages** | Actual Product Page | Q1 2026 Review, Project X Status KW 8 |

## Why

Every organization communicates with the same building blocks. But nobody has defined them.

- **Format silos** — Information is trapped in PowerPoint, Word, Excel, Miro. Not reusable. Not machine-readable.
- **Inconsistency** — Every presentation is a snowflake. No design system. No standards.
- **Low information density** — Slides full of bullet points. No visual design. No data-ink ratio.
- **Person-dependent** — Only the creator understands the slide. No structured data behind it.
- **AI-blind** — LLMs can barely parse PowerPoint. Structured primitives they can.

## The Primitives

### Atoms

| Primitive | Description | Key Fields |
|-----------|-------------|------------|
| **Decision** | Who decided what, when, why | who, what, date, status, rationale |
| **Metric** | Number with context and trend | name, value, unit, trend, period |
| **Risk** | Assessed risk with mitigation | description, probability, impact, owner, mitigation |
| **Action** | Task with ownership | who, what, deadline, status |
| **Timeline** | Milestone with dependencies | milestone, date, owner, dependencies, status |
| **Status** | State assessment of an entity | entity, state, detail, updated |

### Molecules

| Primitive | Description | Composition |
|-----------|-------------|-------------|
| **Argument** | Claim + Evidence + Implication | claim, evidence[], implication |
| **Comparison** | A vs. B with criteria | options[], criteria[], recommendation |
| **Status Update** | Metric + Trend + Assessment | metric, trend, assessment, action_needed |

## Connection to Org as Code

Business Primitives is the communication layer. [Org as Code](https://org-as-code.com) is the structural layer. Together, they make organizations legible — their structures *and* their information.

A decision in OPI (Organizational Programming Interface) is simultaneously a Decision Primitive. The systems share the same data.

## Structure

```
spec/
├── atoms/          # 6 atomic primitives (YAML definitions)
└── molecules/      # 3 molecular compositions
examples/           # Real-world template compositions
```

## Status

**V0.5** — Initial definitions. Atoms and molecules defined. Examples sketched. This is the beginning.

## Author

**Andreas Siemes** — [andreassiemes.de](https://andreassiemes.de) · [Org as Code](https://org-as-code.com) · [LinkedIn](https://linkedin.com/in/andreassiemes)

## License

MIT
