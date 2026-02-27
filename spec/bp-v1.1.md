# Business Primitives Specification

**Version:** 1.1.0
**Status:** Release
**License:** MIT
**Authors:** Andreas Siemes
**Date:** 2026-02-22

> A specification for composable, format-agnostic building blocks of business communication.

Business Primitives defines typed, structured units of business information — decisions, metrics, risks, actions, timelines, statuses, stakeholders, objectives — that compose into richer structures. Like Brad Frost's Atomic Design for UI, this spec applies layered composition to business communication: Atoms combine into Molecules, Molecules into Organisms, Organisms fill Templates, Templates become Pages.

The primitives are format-agnostic: the same Decision can render as a slide, a table row, an email paragraph, or a dashboard card. They are human-readable (YAML) and machine-parseable (JSON Schema). They encode *what* was decided, measured, or risked — not how it looks.

---

## 1. Design Principles

| # | Principle | Core | Source |
|---|-----------|------|--------|
| 1 | **Message over decoration** | Every primitive encodes a "so what". Signal-to-noise maximized. Nothing in communication is neutral. | Doumont Law 2, Tufte |
| 2 | **Human-readable, machine-parseable** | YAML as primary format. A business person reads it; a tool parses it. No proprietary binary formats. | OPI P1, Backstage |
| 3 | **Composable, not monolithic** | Atoms combine into Molecules, Molecules into Organisms. No primitive exists in isolation; none requires a specific container. | Brad Frost Atomic Design |
| 4 | **Format-agnostic** | A primitive defines structure and semantics, never layout. The same Decision renders as slide, table row, email, or dashboard card. | dbt Semantic Layer, Structurizr |
| 5 | **Three or fewer** | Molecules SHOULD NOT combine more than 3 atom types. Cognitive load management through constraint. | Doumont Rule of Three |
| 6 | **Consistent vocabulary** | Same concept = same field name across all primitives. Shared enum registry prevents synonym drift. | IBCS UNIFY |
| 7 | **Extensible** | Custom fields use `x-` prefix. Organizations extend without forking the spec. | OPI x- Pattern |

---

## 2. Prior Art and Differentiation

Business Primitives builds on decades of work in information design, business reporting, and software architecture. No existing framework offers composable, format-agnostic business communication primitives as an open specification.

| Framework | What It Provides | Gap for Business Primitives |
|-----------|-----------------|----------------------------|
| **IBCS** | 98 visual rules for charts/tables (ISO candidate) | No data model, no composability |
| **XBRL** | Financial reporting standard (SEC/EU mandated) | Financial domain only, extremely complex |
| **DMN** | Decision Model and Notation (OMG) | Decisions only, heavyweight |
| **ADR/MADR** | Decision records as Markdown | Software architecture only |
| **dbt Semantic Layer** | Metrics defined in YAML | Metrics only |
| **Backstage** | YAML entity catalog at scale (Spotify) | Software entities only |
| **StratML** | Strategy Markup Language (ISO 17469) | Strategy plans only, XML-based |
| **OKR Frameworks** | Objectives and Key Results methodology | Methodology only, no schema |
| **Agile Primitives** | 10 foundational agile principles | Principles, not data structures |
| **Brad Frost** | Atoms → Pages composition model | UI components, not business information |
| **Doumont** | Three Laws + Tree/Map/Theorem | Principles framework, no schema |
| **Minto** | Pyramid Principle, SCQA | Thinking model, no machine-readable format |
| **Tufte** | Data-ink ratio, graphical integrity | Visualization philosophy, no structure |

**Positioning:** Brad Frost's Atomic Design + Backstage's YAML Catalog + dbt's Semantic Layer + IBCS Visual Rules + Doumont's Three Laws — applied to business communication.

---

## 3. Terminology

| Term | Definition |
|------|-----------|
| **Primitive** | A reusable, typed unit of business information with defined fields and semantics. |
| **Atom** | An irreducible primitive. One semantic purpose. Cannot be meaningfully decomposed. |
| **Molecule** | A composition of 2–3 atom types conveying a richer message than any atom alone. |
| **Organism** | A section-level grouping of molecules and atoms serving a specific communication purpose. |
| **Template** | A named arrangement of organisms defining document structure. |
| **Page** | A concrete instance of a template, filled with actual data. |
| **Composition** | Combining primitives into higher-level structures, either inline or by reference. |
| **Rendering** | Transforming primitives into a specific output format (slide, PDF, dashboard, email). |
| **Enum Registry** | The central set of shared enumerations used across all primitives. |
| **Rendering Hint** | An optional field that communicates preferred display behavior to renderers without mandating it. |

---

## 4. Document Structure

A Business Primitives document is a YAML file with these top-level fields:

```yaml
bp: "1.1"                # Spec version (REQUIRED)
primitive: metric         # Primitive type (REQUIRED)
id: met-revenue-q1        # Machine-readable slug ID (optional)
meta:                     # Audit and lifecycle fields (optional)
  created: 2026-03-15
  updated_at: 2026-03-20
  created_by: "CFO"
  source: "Q1 Board Meeting"
  tags: [finance, q1]
  lifecycle: active
refs:                     # Cross-references to other primitives (optional)
  - id: dec-expansion
    rel: informs
# ... domain-specific fields follow
```

**Spec version** (`bp`) uses `"MAJOR.MINOR"` format. Parsers MUST reject documents with an unsupported major version. Parsers SHOULD warn on minor version mismatch but MUST NOT reject.

**Primitive type** (`primitive`) identifies which atom or molecule definition governs the document's fields.

**ID** (`id`) is a machine-readable slug matching `^[a-z0-9][a-z0-9-]*$`. IDs MUST be unique within a page or collection.

---

## 5. Common Fields

These fields are available on ALL primitives regardless of type.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bp` | string | MUST | Spec version (`"1.1"`) |
| `primitive` | string | MUST | Primitive type name |
| `id` | string | MAY | Slug ID (`^[a-z0-9][a-z0-9-]*$`) |
| `meta.created` | date | MAY | Creation date (YYYY-MM-DD) |
| `meta.updated_at` | date | MAY | Last modification date (YYYY-MM-DD) |
| `meta.created_by` | string | MAY | Creator (person or role) |
| `meta.source` | string | MAY | Provenance or origin |
| `meta.tags` | list | MAY | Discovery and classification tags |
| `meta.lifecycle` | enum | MAY | `draft` / `active` / `superseded` / `archived` |
| `refs` | list | MAY | Cross-references (objects with `id` and `rel`) |
| `render` | object | MAY | Rendering hints for display tools (see Section 16) |

> **V1.1 Change:** `meta.updated` renamed to `meta.updated_at` for consistency with `Status.updated_at`. Documents using `meta.updated` (V1.0 field name) SHOULD be migrated. Parsers MAY accept both during a transition period.

### Relationship Types

The `rel` field in `refs` entries uses these values:

| Relationship | Meaning |
|-------------|---------|
| `informs` | This primitive provides input to the referenced one |
| `triggers` | This primitive causes the referenced one to activate |
| `blocks` | This primitive prevents progress on the referenced one |
| `blocked-by` | This primitive is blocked by the referenced one *(new in V1.1)* |
| `supports` | This primitive provides evidence or backing |
| `supersedes` | This primitive replaces the referenced one |
| `part-of` | This primitive is a component of the referenced one |
| `derived-from` | This primitive was created based on the referenced one |

> **V1.1 Addition:** `blocked-by` clarifies the blocking direction. In V1.0, `blocks` was ambiguous (does "A blocks B" mean A is the blocker or A is blocked?). `blocked-by` makes the dependency explicit: the action using this rel is blocked by the referenced primitive.

### Shared Enum Registry

Enumerations used across multiple primitives are defined once here. Individual primitives reference these by name.

| Enum | Values | Used By |
|------|--------|---------|
| `trend` | `improving` / `stable` / `declining` / `volatile` | Metric, Status-Update |
| `direction` | `up` / `down` / `flat` | Metric (raw directional data) |
| `confidence` | `hypothesis` / `supported` / `strong` / `proven` | Argument, OKR |
| `priority` | `low` / `normal` / `high` / `critical` | Action |
| `severity` | `low` / `medium` / `high` / `critical` | Risk (probability, impact) |
| `rag` | `green` / `amber` / `red` | Status |
| `lifecycle` | `draft` / `active` / `superseded` / `archived` | Common (meta) |
| `influence` | `high` / `medium` / `low` | Stakeholder |
| `stance` | `champion` / `supporter` / `neutral` / `skeptic` / `blocker` | Stakeholder |
| `urgency` | `immediate` / `this-week` / `this-month` / `informational` | Briefing *(new in V1.1)* |

---

## 6. Atoms

Atoms are irreducible primitives. Each serves one semantic purpose and cannot be meaningfully decomposed further. Business Primitives V1.1 defines 8 atoms.

### 6.1 Decision

A decision captures who decided what, when, and why. The most fundamental unit of organizational action.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `decided_by` | string | MUST | Person or role that made the decision |
| `title` | string | MUST | The decision itself, stated clearly |
| `decided_date` | date | MUST | When the decision was made (YYYY-MM-DD) |
| `status` | enum | MUST | `proposed` / `approved` / `rejected` / `deferred` / `superseded` |
| `rationale` | string | MAY | Why this decision was made |
| `context` | string | MAY | Background information or constraints |
| `alternatives` | list | MAY | Options that were considered and rejected |

**Constraints:**
- If `status` is `approved` or `rejected`, `decided_date` MUST be in the past or today.
- If `status` is `superseded`, a `refs` entry with `rel: supersedes` SHOULD exist.

**Example:**
```yaml
bp: "1.1"
primitive: decision
id: dec-budget-q2
decided_by: "CFO"
title: "Approved Q2 2026 marketing budget increase to 150k EUR"
decided_date: 2026-03-15
status: approved
rationale: "ROI from Q1 campaign exceeded projections by 40%"
context: "Board requested growth acceleration"
alternatives:
  - "Maintain Q1 budget level"
  - "Redirect funds to product development"
```

---

### 6.2 Metric

A metric is a number with context. Numbers without context are noise; metrics are signal.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | What is being measured |
| `value` | number | MUST | The measured value |
| `unit` | string | MUST | Unit of measurement (%, EUR, count, hours) |
| `period` | string | MAY | Time period this metric covers *(changed from MUST in V1.0)* |
| `as_of` | datetime | MAY | Point-in-time timestamp for live/snapshot metrics (ISO 8601) *(new in V1.1)* |
| `trend` | enum | MAY | `improving` / `stable` / `declining` / `volatile` |
| `direction` | enum | MAY | `up` / `down` / `flat` (raw directional data) |
| `target` | number | MAY | Target or benchmark value |
| `previous` | number | MAY | Prior period value for self-contained delta calculation *(new in V1.1)* |
| `source` | string | MAY | Where this number comes from |

**Constraints:**
- At least one of `period` or `as_of` MUST be present (M8).
- `trend` is evaluative (improving = good regardless of direction). `direction` is raw (up = numerically increasing). Both MAY coexist.
- If `target` is provided, `unit` applies to both `value` and `target`.
- If `previous` is provided, it MUST use the same `unit` as `value`.

> **V1.1 Change:** `period` changed from MUST to MAY. `as_of` added for live/snapshot metrics where a fixed period doesn't apply (e.g. dashboard gauges, real-time KPIs). At least one of the two MUST be present (M8).

> **V1.1 Addition:** `previous` field for self-contained delta calculation. When a renderer has no access to historical data, `previous` enables it to show a delta without fetching prior primitives.

**Example (periodic):**
```yaml
bp: "1.1"
primitive: metric
id: met-retention-q1
name: "Customer Retention Rate"
value: 94.2
unit: "%"
period: "Q1 2026"
trend: improving
direction: up
target: 95.0
previous: 92.8
source: "CRM Dashboard"
```

**Example (live/snapshot):**
```yaml
bp: "1.1"
primitive: metric
id: met-server-uptime
name: "API Uptime"
value: 99.97
unit: "%"
as_of: "2026-03-15T14:30:00Z"
direction: up
source: "StatusPage"
```

---

### 6.3 Risk

A risk combines threat description, probability, impact, ownership, and mitigation. Making risks explicit is the first step to managing them.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `threat` | string | MUST | What could go wrong |
| `probability` | enum | MUST | `low` / `medium` / `high` / `critical` (severity scale) |
| `impact` | enum | MUST | `low` / `medium` / `high` / `critical` (severity scale) |
| `owner` | string | MUST | Person or role responsible for monitoring |
| `mitigation` | string | MAY | Planned response or prevention strategy |
| `status` | enum | MAY | `identified` / `monitoring` / `mitigating` / `resolved` / `accepted` |
| `risk_score` | string | MAY | Computed PxI label (e.g., "high-medium") |

**Constraints:**
- `risk_score`, if provided, MUST equal `"{probability}-{impact}"`.
- If `status` is `resolved`, `mitigation` SHOULD describe what was done.

**Example:**
```yaml
bp: "1.1"
primitive: risk
id: rsk-engineer-attrition
threat: "Key engineer leaving during migration project"
probability: medium
impact: high
owner: "Engineering Lead"
mitigation: "Cross-training program, documentation sprint in KW 12"
status: mitigating
risk_score: "medium-high"
```

---

### 6.4 Action

An action is a task with ownership and a due date. The bridge between decisions and outcomes.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `owner` | string | MUST | Person or role responsible |
| `title` | string | MUST | What needs to be done |
| `description` | string | MAY | Additional detail or acceptance criteria |
| `due_date` | date | MUST | When it is due (YYYY-MM-DD) |
| `status` | enum | MUST | `open` / `in_progress` / `blocked` / `done` / `cancelled` |
| `priority` | enum | MAY | `low` / `normal` / `high` / `critical` |
| `completed_date` | date | MAY | When the action was actually completed (YYYY-MM-DD) *(new in V1.1)* |

**Constraints:**
- If `status` is `done`, `due_date` SHOULD be in the past or today.
- If `status` is `done` and `completed_date` is provided, `completed_date` MUST be in the past or today.
- If `status` is `blocked`, a `refs` entry with `rel: blocked-by` SHOULD identify the blocker.

> **V1.1 Addition:** `completed_date` field. When `status: done`, this records the actual completion date (which may differ from `due_date`). Essential for on-time delivery tracking and retrospective analysis.

**Example:**
```yaml
bp: "1.1"
primitive: action
id: act-roadmap-q2
owner: "Product Manager"
title: "Finalize Q2 roadmap and share with stakeholders"
due_date: 2026-03-20
completed_date: 2026-03-18
status: done
priority: high
```

---

### 6.5 Timeline

A timeline milestone marks a point in time that matters, with explicit dependencies on what must happen before it.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `milestone` | string | MUST | Name of the milestone |
| `target_date` | date | MUST | Target date (YYYY-MM-DD) |
| `owner` | string | MUST | Person or role responsible |
| `status` | enum | MUST | `planned` / `on_track` / `at_risk` / `delayed` / `completed` |
| `dependencies` | list | MAY | What must be completed before this milestone — strings or primitive IDs |
| `deliverable` | string | MAY | What is produced at this milestone |

**Constraints:**
- If `status` is `completed`, `target_date` SHOULD be in the past or today.
- `dependencies` entries SHOULD reference primitive IDs (for machine-resolvable chains) or descriptive strings (for human-readable summaries). Mixed usage is permitted.

> **V1.1 Clarification:** `dependencies` entries may now be either primitive ID references (enabling `$ref`-resolution) or free-text strings. When using IDs, they should follow the `^[a-z0-9][a-z0-9-]*$` pattern and resolve within the current page or collection (rule X5).

**Example:**
```yaml
bp: "1.1"
primitive: timeline
id: tl-mvp-launch
milestone: "MVP Launch"
target_date: 2026-06-01
owner: "Product Lead"
status: on_track
dependencies:
  - "tl-api-complete"
  - "tl-security-audit"
deliverable: "Production-ready platform with core features"
```

---

### 6.6 Status

A status captures the current state of any entity — a project, team, system, or goal. Traffic-light simplicity with structured detail.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | MUST | What is being assessed |
| `state` | enum | MUST | `green` / `amber` / `red` (RAG) |
| `detail` | string | MUST | Brief explanation of current state |
| `updated_at` | date | MUST | When this assessment was last updated |
| `owner` | string | MAY | Who is responsible for this entity |
| `previous_state` | enum | MAY | Prior RAG state — enables trend detection without a separate Metric *(new in V1.1)* |

**Constraints:**
- `updated_at` MUST NOT be in the future.

> **V1.1 Addition:** `previous_state` field. When a renderer shows Status, it can now indicate direction (e.g. "was amber → now green" or "was green → now red") without requiring a separate Metric primitive or cross-reference lookup. This is a common display pattern in executive dashboards.

**Example:**
```yaml
bp: "1.1"
primitive: status
id: sts-migration
entity: "Data Platform Migration"
state: amber
previous_state: green
detail: "On schedule but resource risk — 2 engineers on vacation in KW 14"
updated_at: 2026-03-10
owner: "Migration Lead"
```

---

### 6.7 Stakeholder

A stakeholder maps the people and roles that matter to a decision, project, or initiative. Influence and stance drive engagement strategy.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | Person or role name |
| `role` | string | MUST | Organizational role or title |
| `influence` | enum | MUST | `high` / `medium` / `low` |
| `stance` | enum | MUST | `champion` / `supporter` / `neutral` / `skeptic` / `blocker` |
| `interests` | list | MAY | What this stakeholder cares about |

**Example:**
```yaml
bp: "1.1"
primitive: stakeholder
id: sh-cfo
name: "Maria Schmidt"
role: "CFO"
influence: high
stance: champion
interests:
  - "Cost efficiency"
  - "Revenue growth"
  - "Risk management"
```

---

### 6.8 Objective *(new in V1.1)*

An objective is a declared goal with a clear timeframe and owner. Where a Metric describes what *is* (measured), and a Status describes how things *stand* (assessed), an Objective declares what *should be* (intended). It is qualitative and motivational by nature.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | MUST | The goal, stated clearly and motivationally |
| `timeframe` | string | MUST | Time horizon (`Q1 2026`, `H1 2026`, `2026`, `Sprint 12`) |
| `owner` | string | MUST | Person, role, or team responsible |
| `status` | enum | MUST | `draft` / `active` / `achieved` / `missed` / `deferred` |
| `description` | string | MAY | Why this objective matters — strategic context |
| `success_criteria` | list | MAY | Qualitative conditions for "achieved" — complements quantitative Key Results |

**Constraints:**
- If `status` is `achieved` or `missed`, the `timeframe` SHOULD refer to a past or current period.
- `success_criteria` is intentionally qualitative. Quantitative measurement belongs in Metric primitives (linked via OKR molecule or `refs`).

**Positioning vs. related primitives:**
- **Objective vs. Action:** An objective is a destination. An action is a step toward it.
- **Objective vs. Status:** An objective is forward-looking (what we intend). Status is present-looking (how we stand).
- **Objective vs. Metric:** An objective is qualitative direction. A metric is a quantitative measurement.

**Example:**
```yaml
bp: "1.1"
primitive: objective
id: obj-dach-leadership
title: "Establish DACH market leadership in enterprise analytics"
timeframe: "H1 2026"
owner: "VP Sales DACH"
status: active
description: "Strategic priority for 2026. DACH represents our largest untapped market in Europe."
success_criteria:
  - "Recognized as top-3 vendor by at least 2 analyst reports"
  - "Reference customer in each of the 3 DACH countries"
  - "DACH team fully staffed and onboarded"
```

---

## 7. Molecules

Molecules compose 2–3 atom types into a richer unit of meaning. A molecule conveys a message that no single atom can express alone. Per Design Principle 5, molecules SHOULD NOT combine more than 3 atom types.

A molecule's `composed_of` declaration specifies which atom types it *understands* and *can embed*. Atoms listed in `composed_of` are not all required — they indicate the semantic domain. Individual field requirements are defined per-molecule.

### 7.1 Argument

**Composed of:** Metric, Decision

An argument is the fundamental unit of persuasion in business: a claim supported by evidence, leading to an implication. Every good slide, memo, and pitch is built from arguments.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `claim` | string | MUST | The assertion being made |
| `evidence` | list | MUST | Supporting data points, metrics, or facts |
| `implication` | string | MUST | What follows — the "so what" |
| `confidence` | enum | MAY | `hypothesis` / `supported` / `strong` / `proven` |

**Constraints:**
- `evidence` MUST contain at least one item.
- Evidence items MAY be strings or inline metric primitives.

**Example:**
```yaml
bp: "1.1"
primitive: argument
id: arg-dach-expansion
claim: "We should expand into the DACH market in Q3"
evidence:
  - "DACH revenue grew 35% YoY without dedicated sales"
  - "3 inbound enterprise leads from Germany in Q1"
  - "Competitor X just exited the German market"
implication: "A dedicated DACH team could capture 2M EUR ARR within 12 months"
confidence: supported
```

---

### 7.2 Comparison

**Composed of:** Metric, Risk, Stakeholder

A comparison structures the evaluation of alternatives. Instead of pros/cons lists or gut feelings, it provides criteria-based assessment with a clear recommendation.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `question` | string | MUST | What decision does this comparison support |
| `options` | list | MUST | The alternatives being compared (objects with `name`, `scores`, optional `total`) |
| `criteria` | list | MUST | Evaluation dimensions (objects with `name` and `weight`) |
| `recommendation` | string | MUST | Which option is recommended and why |

**Constraints:**
- `options` MUST contain at least 2 items.
- `criteria` weights SHOULD sum to 1.0 (M7: validator MUST warn if sum deviates by more than 0.01).
- Each option's `scores` keys MUST match the `criteria` names.
- Option `total`, if provided, SHOULD equal the weighted sum of scores against criteria weights.

> **V1.1 Clarification:** Weight-sum validation (M7) is now explicit. A validator at Standard conformance MUST warn when `sum(criteria[*].weight)` is not within 0.01 of 1.0. Options MAY include a `total` field for pre-computed weighted scores.

**Example:**
```yaml
bp: "1.1"
primitive: comparison
id: cmp-cicd-platform
question: "Which CI/CD platform should we migrate to?"
options:
  - name: "GitHub Actions"
    scores: { cost: 8, integration: 9, learning_curve: 7 }
    total: 8.3
  - name: "GitLab CI"
    scores: { cost: 7, integration: 6, learning_curve: 8 }
    total: 6.7
  - name: "Jenkins"
    scores: { cost: 9, integration: 5, learning_curve: 4 }
    total: 6.0
criteria:
  - { name: cost, weight: 0.3 }
  - { name: integration, weight: 0.5 }
  - { name: learning_curve, weight: 0.2 }
recommendation: "GitHub Actions — best integration score outweighs minor cost difference"
```

---

### 7.3 Status Update

**Composed of:** Metric, Status, Action

A status update is what most weekly reports try to be: a metric with its trend, an assessment, and whether action is needed. Structured, not narrated.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `metric` | object | MUST | The metric being reported (inline metric primitive) |
| `trend` | enum | MUST | `improving` / `stable` / `declining` / `volatile` |
| `assessment` | string | MUST | Human interpretation of what the numbers mean |
| `action_needed` | boolean | MUST | Whether this requires intervention |
| `actions` | list | MAY | Recommended next steps (inline action primitives or strings) |

**Constraints:**
- `metric` MUST include at minimum `name`, `value`, and `unit`.
- If `action_needed` is `true`, `actions` SHOULD contain at least one item.

**Example:**
```yaml
bp: "1.1"
primitive: status-update
id: su-velocity-kw12
metric:
  name: "Sprint Velocity"
  value: 42
  unit: "story points"
  period: "KW 12"
trend: declining
assessment: "Third consecutive sprint below target. Team capacity reduced by parental leave."
action_needed: true
actions:
  - "Adjust sprint commitment to 35 SP until team is back to full capacity in KW 16"
```

---

### 7.4 Progress

**Composed of:** Metric, Timeline, Action

A progress report captures where a project or initiative stands relative to its plan. It combines the quantitative (metrics, velocity) with the structural (milestones, next actions) into a single, renderable unit.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `checkpoint` | string | MUST | Where we stand right now |
| `velocity` | string | MAY | Rate of progress (qualitative or quantitative) |
| `forecast` | string | MAY | Projected completion or next milestone |
| `blockers` | list | MAY | What is slowing progress |
| `milestones` | list | MAY | Timeline primitives (inline or by reference) |
| `metrics` | list | MAY | Metric primitives tracking progress |
| `next_actions` | list | MAY | Action primitives for next steps |

**Constraints:**
- At least one of `milestones`, `metrics`, or `next_actions` SHOULD be present.
- `blockers` items MAY be strings or inline risk primitives.

**Example:**
```yaml
bp: "1.1"
primitive: progress
id: prg-migration-kw12
checkpoint: "Phase 2 of 4 complete. Data layer migrated, API layer in progress."
velocity: "2 milestones per month (on plan)"
forecast: "Full migration by KW 22 if current pace holds"
blockers:
  - "Vendor API documentation incomplete for auth module"
milestones:
  - milestone: "Data Layer Complete"
    target_date: 2026-02-28
    owner: "Data Lead"
    status: completed
  - milestone: "API Layer Complete"
    target_date: 2026-03-31
    owner: "Backend Lead"
    status: on_track
metrics:
  - name: "Endpoints Migrated"
    value: 34
    unit: "of 78"
    period: "KW 12"
    direction: up
next_actions:
  - owner: "Backend Lead"
    title: "Complete auth module migration"
    due_date: 2026-03-15
    status: in_progress
    priority: high
```

---

### 7.5 OKR *(new in V1.1)*

**Composed of:** Objective, Metric

An OKR (Objectives and Key Results) pairs a qualitative goal with the quantitative measures that signal its achievement. The Objective declares where we want to go; the Key Results (Metrics) tell us whether we are getting there.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objective` | object | MUST | Inline Objective primitive |
| `key_results` | list | MUST | Metric primitives as Key Results (min 1, max 5) |
| `overall_progress` | number | MAY | Aggregated progress 0–100 (%) across all Key Results |
| `confidence` | enum | MAY | `hypothesis` / `supported` / `strong` / `proven` — confidence in achieving the objective |

**Constraints:**
- `key_results` MUST contain at least 1 and SHOULD NOT contain more than 5 items.
- Each Key Result MUST be a valid Metric primitive (inline or by `$ref`).
- Each Key Result SHOULD have a `target` field to enable progress calculation.
- `overall_progress`, if provided, MUST be between 0 and 100 inclusive.
- `objective` MUST include at minimum `title`, `timeframe`, and `owner`.

**Example:**
```yaml
bp: "1.1"
primitive: okr
id: okr-dach-h1
objective:
  title: "Establish DACH market leadership in enterprise analytics"
  timeframe: "H1 2026"
  owner: "VP Sales DACH"
  status: active
key_results:
  - name: "DACH ARR"
    value: 340000
    unit: "EUR"
    period: "KW 12"
    target: 800000
    trend: improving
    direction: up
  - name: "Enterprise Customers (DACH)"
    value: 4
    unit: "customers"
    period: "KW 12"
    target: 12
    trend: improving
    direction: up
  - name: "DACH NPS"
    value: 52
    unit: "score"
    period: "KW 12"
    target: 60
    trend: stable
overall_progress: 38
confidence: supported
```

---

### 7.6 Briefing *(new in V1.1)*

**Composed of:** Argument, Decision

A briefing is the executive communication pattern: concise, structured, recommendation-first. It applies Minto's Pyramid Principle and Doumont's "Message over decoration" to a single composable unit. Where an Argument builds a case bottom-up (claim → evidence → implication), a Briefing delivers it top-down (recommendation first, then the why).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `subject` | string | MUST | What this briefing is about (one sentence) |
| `situation` | string | MUST | Current state of affairs — factual, neutral |
| `significance` | string | MUST | Why this matters right now — the "so what" |
| `recommendation` | string | MUST | What is recommended — concrete and direct |
| `urgency` | enum | MAY | `immediate` / `this-week` / `this-month` / `informational` |
| `support` | list | MAY | Evidence, data, or references backing the recommendation |

**Constraints:**
- `support` items MAY be strings or inline metric/argument/risk primitives.
- `recommendation` SHOULD be actionable — a verb phrase, not a description.
- `situation` SHOULD be factual and free of recommendations. Recommendations belong in `recommendation`.

**Briefing vs. Argument:**
- Argument: Bottom-up (claim → evidence → implication). Built for building a case.
- Briefing: Top-down (recommendation first, then why). Built for executive consumption.

**Example:**
```yaml
bp: "1.1"
primitive: briefing
id: brf-attrition-risk
subject: "Senior engineer attrition risk during Q2 platform migration"
situation: "Two of our five senior platform engineers have flagged burnout concerns. Both are key knowledge holders for the auth module — the current migration bottleneck."
significance: "Loss of either engineer would delay migration by 6–8 weeks and increase security risk during a critical compliance window."
recommendation: "Approve retention bonuses for both engineers this week and schedule dedicated 1:1s with the Engineering Lead before end of KW 13."
urgency: immediate
support:
  - "Engineering Lead 1:1 notes from KW 11 and KW 12"
  - "Auth module completion: 22% (behind plan by 3 weeks)"
refs:
  - id: rsk-engineer-attrition
    rel: informs
```

---

## 8. Organisms

Organisms are section-level groupings of molecules and atoms that serve a specific communication purpose. They are not standalone YAML files but patterns for composing primitives into meaningful sections.

An organism defines **slots** — typed positions that accept specific primitives with cardinality constraints.

| Organism | Slots | Purpose |
|----------|-------|---------|
| **Executive Summary** | `status: 1`, `metrics: 1..3`, `decisions: 0..3`, `risks: 0..3` | Document overview at maximum density |
| **Risk Register** | `risks: 1..n`, `actions: 0..n` | Risk table with mitigations and response actions |
| **Action Plan** | `actions: 1..n`, `timelines: 0..n` | Next steps with milestones and dependencies |
| **Performance Dashboard** | `status-updates: 1..n`, `metrics: 0..n` | KPI tracking with trends and assessments |
| **OKR Board** | `okrs: 1..n` | Objectives and Key Results overview *(new in V1.1)* |

### Organism Syntax

```yaml
sections:
  - name: "Executive Summary"
    type: executive-summary
    primitives:
      - primitive: status
        # ... fields
      - primitive: metric
        # ... fields (up to 3)
      - primitive: decision
        # ... fields (up to 3)
```

**Constraints:**
- Slot cardinalities MUST be respected. An executive summary with 5 metrics violates the `1..3` constraint.
- Organisms MAY contain primitives not listed in their slot definition if prefixed with `x-`.
- An organism MUST contain at least the minimum required slots.

> **V1.1 Addition:** OKR Board organism. Provides a dedicated container for one or more OKR molecules, suitable for quarterly planning sessions, strategy reviews, and team-level goal tracking.

---

## 9. Templates

Templates are named arrangements of organisms that define complete document structures. A template specifies which organisms appear, in what order, with which are required vs. optional.

### 9.1 Decision Paper

A structured document for making and recording a decision.

| Section | Organism | Required |
|---------|----------|----------|
| Context | Argument (1) | MUST |
| Options | Comparison (1) | MUST |
| Risks | Risk Register (1) | SHOULD |
| Decision | Decision (1) | MUST |
| Next Steps | Action Plan (1) | SHOULD |

### 9.2 Quarterly Review

A comprehensive periodic review of business performance.

| Section | Organism | Required |
|---------|----------|----------|
| Executive Summary | Executive Summary (1) | MUST |
| Performance | Performance Dashboard (1) | MUST |
| Key Decisions | Decisions (1..n) | SHOULD |
| Risks | Risk Register (1) | SHOULD |
| Actions | Action Plan (1) | MUST |

### 9.3 Weekly Status

A lightweight recurring status report.

| Section | Organism | Required |
|---------|----------|----------|
| Summary | Executive Summary (1) | MUST |
| Progress | Progress (1..n) | MUST |
| Blockers | Risk Register (1) | MAY |
| Next Week | Action Plan (1) | SHOULD |

### 9.4 OKR Review *(new in V1.1)*

A structured OKR check-in document, suitable for monthly or quarterly OKR reviews.

| Section | Organism | Required |
|---------|----------|----------|
| Goals | OKR Board (1) | MUST |
| Risks | Risk Register (1) | SHOULD |
| Decisions | Decision (1..n) | SHOULD |
| Next Actions | Action Plan (1) | MUST |

### 9.5 Executive Briefing *(new in V1.1)*

A concise, single-topic briefing for executive decision-makers.

| Section | Organism | Required |
|---------|----------|----------|
| Briefing | Briefing (1) | MUST |
| Supporting Evidence | Performance Dashboard (1) | MAY |
| Decision | Decision (1) | MAY |
| Actions | Action Plan (1) | MAY |

---

## 10. Pages

A page is a concrete instance of a template, filled with actual data. It is the final level of the composition hierarchy — what people actually read, present, or review.

```yaml
bp: "1.1"
page: quarterly-review           # Template name
title: "Q1 2026 Business Review"
author: "Strategy Team"
date: 2026-04-01
meta:
  tags: [q1, review, 2026]

sections:
  - name: "Executive Summary"
    type: executive-summary
    primitives:
      - primitive: status
        entity: "Q1 2026 Overall"
        state: green
        # ...
      - primitive: metric
        name: "Revenue"
        value: 2400000
        # ...
  # ... more sections
```

**Page vs. Template:** A template defines structure. A page fills that structure with data. `quarterly-review` is a template; "Q1 2026 Business Review" is a page.

---

## 11. Composition Model

Composition is the mechanism by which primitives combine into higher-level structures. Business Primitives supports three composition modes.

### 11.1 Inline Embedding

Primitives are nested directly within their parent structure.

```yaml
# Metric inline within a status-update
metric:
  name: "Sprint Velocity"
  value: 42
  unit: "story points"
  period: "KW 12"
```

### 11.2 Reference

Primitives reference other primitives by ID using the `$ref` syntax.

```yaml
# Reference to a separately defined primitive
metric:
  $ref: "met-velocity-kw12"
```

### 11.3 Hybrid

Mix of inline and referenced primitives in the same document.

```yaml
metric:
  name: "Revenue"
  value: 2400000
  unit: "EUR"
  period: "Q1 2026"
actions:
  - $ref: "act-hire-dach"
  - owner: "Product Manager"
    title: "Launch self-service onboarding"
    due_date: 2026-05-15
    status: in_progress
```

### 11.4 Collection

A collection is a multi-document set of primitives that share a namespace, enabling cross-document `$ref` resolution.

```yaml
# collection.yaml — declares a shared namespace
bp: "1.1"
collection: "q1-review-2026"
documents:
  - path: "atoms/met-revenue-q1.yaml"
    id: met-revenue-q1
  - path: "atoms/dec-dach-expansion.yaml"
    id: dec-dach-expansion
  - path: "pages/q1-review.yaml"
    id: q1-review
```

> **V1.1 Addition:** Collection document type. In V1.0, `$ref` resolved "within the current page or collection" without defining what a collection is. V1.1 introduces the `collection` document type to formalize multi-file BP projects.

### Composition Rules

1. **Atoms compose into Molecules.** A molecule's `composed_of` declaration specifies which atom types it understands.
2. **Molecules and Atoms compose into Organisms.** An organism's slot definition specifies accepted types and cardinalities.
3. **Organisms compose into Templates.** A template's section definition specifies required and optional organisms.
4. **Templates instantiate into Pages.** A page references its template and fills sections with data.
5. **No level skipping.** An atom MUST NOT appear directly in a template section without an organism wrapper.
6. **References resolve within scope.** A `$ref` resolves within the current page or collection. Cross-collection references use `refs` with `rel`.
7. **Inline primitives inherit context.** An inline primitive within a molecule inherits the molecule's `meta` and `refs` unless it defines its own.
8. **Extension fields are preserved.** Custom `x-` fields MUST be preserved through composition and rendering.

---

## 12. Connection to OPI Spec

Business Primitives and the Organizational Programming Interface (OPI) are complementary specifications by the same author. OPI defines organizational structure (roles, circles, flows); BP defines organizational communication (decisions, metrics, risks).

### Concept Mapping (BP → OPI)

| BP Concept | OPI Concept | Relationship |
|-----------|------------|-------------|
| Decision | `decision` within a Circle/Flow | Same data, different context |
| Metric | KPI in a Role's accountabilities | BP provides the structured format |
| Risk | Risk in a Decision Flow | BP adds probability/impact semantics |
| Action | Task in a Role's domain | BP adds priority/status tracking |
| Stakeholder | Role/Person | BP adds influence/stance analysis |
| Timeline | Milestone in a Decision Flow | BP adds dependency tracking |
| Status | Health of any OPI entity | BP provides the assessment format |
| Objective | Purpose in a Circle | BP adds timeframe/success criteria |

### Concept Mapping (OPI → BP)

| OPI Concept | BP Primitive | What BP Adds |
|------------|-------------|-------------|
| Circle | Stakeholder (group) | Communication context and stance |
| Role | Stakeholder | Individual influence mapping |
| Flow | Progress + Timeline | Milestone tracking and blockers |
| Governance Meeting | Decision + Action | Structured decision records |
| Operations Meeting | Status Update | Recurring metric assessments |
| Proposal | Argument + Comparison | Claim/evidence/recommendation |
| KPI | Metric | Period, trend, target, direction |

### Cross-Reference Convention

Documents may reference across specs using prefixed IDs:

```yaml
refs:
  - id: "opi:circle-engineering"
    rel: part-of
  - id: "bp:dec-platform-migration"
    rel: triggers
```

### Combined Usage Example

A Holacracy-style governance meeting produces both OPI and BP artifacts from the same event:

```yaml
# OPI: The structural record
# opi:circle-engineering/governance-2026-03-15.yaml

# BP: The communication record
bp: "1.1"
primitive: decision
id: dec-platform-migration
decided_by: "Engineering Circle"
title: "Migrate from Jenkins to GitHub Actions"
decided_date: 2026-03-15
status: approved
rationale: "Integration score and team familiarity outweigh cost delta"
refs:
  - id: "opi:circle-engineering"
    rel: part-of
  - id: "bp:cmp-cicd-platform"
    rel: derived-from
```

---

## 13. Validation Rules

Validation rules are organized in four groups. Conformance levels (Section 14) determine which groups apply.

### Structural Rules (S1–S8)

| Rule | Description | Level |
|------|-------------|-------|
| S1 | `bp` field MUST be present and match `"MAJOR.MINOR"` | Minimal |
| S2 | `primitive` field MUST be present and match a defined type | Minimal |
| S3 | `id`, if present, MUST match `^[a-z0-9][a-z0-9-]*$` | Minimal |
| S4 | All REQUIRED fields for the primitive type MUST be present | Minimal |
| S5 | Enum fields MUST contain values from the shared enum registry | Minimal |
| S6 | Date fields MUST use YYYY-MM-DD format | Minimal |
| S7 | `refs` entries MUST contain `id` (string) and `rel` (valid relationship type) | Basic |
| S8 | `meta.lifecycle` MUST be a valid lifecycle enum value | Basic |

### Composition Rules (C1–C8)

| Rule | Description | Level |
|------|-------------|-------|
| C1 | A molecule SHOULD NOT compose more than 3 atom types | Standard |
| C2 | Organism slots MUST respect their cardinality constraints | Standard |
| C3 | Template sections MUST include all MUST organisms | Standard |
| C4 | A `$ref` MUST resolve to an existing primitive within the page or collection | Standard |
| C5 | Pages MUST reference a valid template name | Standard |
| C6 | No level skipping: atoms MUST NOT appear directly in template sections | Standard |
| C7 | Inline primitives within molecules MUST match the `composed_of` types | Standard |
| C8 | Template sections MUST appear in the defined order | Standard |

### Semantic Rules (M1–M8)

| Rule | Description | Level |
|------|-------------|-------|
| M1 | If metric has both `trend` and `direction`, they MUST NOT contradict (declining + up is valid for cost metrics) | Standard |
| M2 | If action `status` is `done`, a completion indicator SHOULD exist | Standard |
| M3 | Risk `probability` and `impact` MUST both be present (not one without the other) | Minimal |
| M4 | If `risk_score` is present, it MUST equal `"{probability}-{impact}"` | Basic |
| M5 | Status `updated_at` MUST NOT be in the future | Minimal |
| M6 | Decision with `status: superseded` SHOULD have a `refs` entry with `rel: supersedes` | Basic |
| M7 | Comparison `criteria` weights SHOULD sum to 1.0; validator MUST warn if deviation exceeds 0.01 *(new in V1.1)* | Standard |
| M8 | Metric MUST have at least one of `period` or `as_of` *(new in V1.1)* | Minimal |

### Cross-Reference Rules (X1–X5)

| Rule | Description | Level |
|------|-------------|-------|
| X1 | `refs` target IDs SHOULD resolve to existing primitives in the collection | Complete |
| X2 | Circular reference chains MUST NOT exist (A refs B refs A) | Complete |
| X3 | IDs MUST be unique within a page | Standard |
| X4 | Cross-spec references (opi:/bp:) MUST use the correct prefix | Complete |
| X5 | Timeline `dependencies` entries that match the slug pattern (`^[a-z0-9][a-z0-9-]*$`) SHOULD resolve to a primitive ID within the collection *(new in V1.1)* | Complete |

---

## 14. Conformance Levels

Implementations and documents may claim conformance at one of four levels. Each level includes all rules from lower levels.

| Level | Requirements | Typical Use |
|-------|-------------|-------------|
| **Minimal** | Atoms only, required fields, structural rules S1–S6, semantic rules M3/M5/M8 | Quick capture, drafts, manual authoring |
| **Basic** | + `id`, `meta`, `refs` fields; rules S7–S8, M4, M6 | Team collaboration, version-controlled docs |
| **Standard** | + Molecules, Organisms, Templates; composition rules C1–C8, semantic rules M1–M2/M7, cross-ref X3 | Production documents, CI validation |
| **Complete** | + Pages, Collections, full cross-reference resolution; rules X1–X2, X4–X5; JSON Schema validation | Tooling, renderers, automated pipelines |

---

## 15. JSON Schema

A JSON Schema for Business Primitives 1.1 is provided at `bp-v1.1.schema.json`. It validates all atom and molecule types at the Standard conformance level.

**Usage with IDE:**

```json
{
  "$schema": "./spec/bp-v1.1.schema.json"
}
```

**Usage with CI:**

```bash
# Validate a single file
ajv validate -s spec/bp-v1.1.schema.json -d examples/decision-paper.yaml

# Validate all YAML files in a collection
find . -name '*.yaml' -exec ajv validate -s spec/bp-v1.1.schema.json -d {} \;
```

> The V1.1 schema fixes a field name inconsistency from V1.0: `meta.updated` is now correctly named `meta.updated_at` in the schema, matching the spec prose and the existing `status.updated_at` field convention.

---

## 16. Rendering Hints *(new in V1.1)*

Rendering hints are optional metadata that communicate preferred display behavior to renderers. They are non-normative — a renderer MAY ignore them. They allow BP authors to express intent without locking primitives to a specific format.

Rendering hints live in the `render` object at the top level of any primitive.

### Common Rendering Fields

| Field | Type | Description |
|-------|------|-------------|
| `render.priority` | integer | Display order within a group (lower = earlier) |
| `render.highlight` | boolean | Mark this primitive for visual emphasis |
| `render.collapsed` | boolean | Default to collapsed/summary view in rich renderers |
| `render.icon` | string | Icon name or emoji hint for visual renderers |
| `render.color` | string | Hex color or semantic color token for visual renderers |
| `render.format` | string | Preferred number format (`currency`, `percent`, `integer`, `decimal`) |

### Rendering Profiles

Renderers may declare support for a rendering profile. A profile defines which `render` fields it respects and how.

| Profile | Target Output | Typical Consumer |
|---------|--------------|-----------------|
| `slide` | Presentation slide | PowerPoint, Google Slides, Marp |
| `table` | Tabular row | Excel, Confluence table, markdown table |
| `card` | Dashboard card | Notion, Notion-like widgets, web dashboards |
| `email` | Email paragraph | Outlook, Gmail, Slack message |
| `doc` | Document section | Word, Confluence page, Notion page |
| `api` | Machine consumption | JSON API response, webhook payload |

### Example

```yaml
bp: "1.1"
primitive: metric
id: met-revenue-q1
name: "Revenue"
value: 2400000
unit: "EUR"
period: "Q1 2026"
trend: improving
render:
  highlight: true
  priority: 1
  format: currency
  icon: "💰"
```

### Design Intent

The format-agnostic principle (Design Principle 4) remains intact: `render` fields do not change the semantic meaning of a primitive. They are hints, not instructions. A validator MUST NOT fail a document because of unsupported `render` fields. A renderer MAY ignore any or all `render` fields.

---

## 17. Tooling Spec *(new in V1.1)*

This section specifies the expected behavior of BP tooling at each conformance level.

### 17.1 Validator

A BP validator checks documents against the validation rules in Section 13.

| Conformance Level | Expected Validator Behavior |
|-------------------|----------------------------|
| **Minimal** | Parse YAML; check S1–S6, M3, M5, M8; report errors |
| **Basic** | + Check S7–S8, M4, M6; report errors |
| **Standard** | + Check C1–C8, M1–M2, M7, X3; report errors and warnings |
| **Complete** | + Check X1–X2, X4–X5; resolve `$ref` and collection; report errors |

**Validator output format** (SHOULD be machine-readable):

```json
{
  "document": "examples/decision-paper.yaml",
  "conformance": "standard",
  "valid": true,
  "errors": [],
  "warnings": [
    {
      "rule": "M7",
      "message": "criteria weights sum to 0.98, expected 1.0",
      "path": "sections[1].primitives[0].criteria"
    }
  ]
}
```

### 17.2 Renderer

A BP renderer transforms a primitive or page into an output format.

| Capability | Required Behavior |
|-----------|------------------|
| **Minimal Renderer** | Render any atom as human-readable text; emit all MUST fields |
| **Table Renderer** | Render lists of atoms as tabular format; respect field order |
| **Card Renderer** | Render a single primitive as a structured card; support `render.highlight` |
| **Page Renderer** | Render a complete page by template; apply organism layout rules |

A renderer MUST preserve `x-` extension fields in its output where the format permits.

### 17.3 Converter

A BP converter transforms BP documents to/from other formats.

| Conversion | Expected Behavior |
|-----------|------------------|
| BP YAML → JSON | Lossless; preserve all fields including `render` and `x-` |
| BP YAML → Markdown | Best-effort; MUST render all MUST fields, MAY omit `render` |
| BP YAML → CSV | Atom-list only; flatten to tabular; MUST include `id`, MUST fields |
| ADR (Markdown) → BP Decision | Map `title`, `status`, `context`, `decision` to BP fields |
| Jira Issue → BP Action | Map `summary`→`title`, `assignee`→`owner`, `duedate`→`due_date`, `status`→`status` |

---

## 18. Migration Guide: V1.0 → V1.1

V1.1 is backward-compatible. Existing V1.0 documents remain valid at Minimal conformance. This section describes how to migrate to full V1.1 compliance.

### Breaking Changes

None. V1.1 introduces no breaking changes.

### Non-Breaking Changes Requiring Attention

| Change | V1.0 | V1.1 | Action Required |
|--------|------|------|----------------|
| `meta.updated` renamed | `meta.updated` | `meta.updated_at` | Update field name in existing documents |
| Metric `period` | MUST | MAY (M8: one of `period`/`as_of` MUST exist) | No action for periodic metrics; add `as_of` for live metrics |
| `bp` version field | `"1.0"` | `"1.1"` | Update version string in documents targeting V1.1 |
| Relationship `blocked-by` | not defined | added | Use for `status: blocked` actions instead of `blocks` |

### Automated Migration

A shell one-liner for bulk `meta.updated` → `meta.updated_at` rename (requires `yq`):

```bash
find . -name '*.yaml' -exec yq e -i '.meta.updated_at = .meta.updated | del(.meta.updated)' {} \;
```

---

## 19. Changelog

### V1.1.0 (2026-02-22)

**New atoms:**
- `objective` — declared goal with timeframe, owner, status, and success criteria

**New molecules:**
- `okr` — Objective + Key Results (1–5 Metric primitives with optional progress and confidence)
- `briefing` — Executive briefing: situation → significance → recommendation

**New organism:**
- `okr-board` — Container for one or more OKR molecules

**New templates:**
- `okr-review` — OKR check-in document
- `executive-briefing` — Single-topic briefing for decision-makers

**New fields on existing atoms:**
- `metric.as_of` — Point-in-time timestamp for live/snapshot metrics
- `metric.previous` — Prior period value for self-contained delta calculation
- `action.completed_date` — Actual completion date for done actions
- `status.previous_state` — Prior RAG state for trend indication

**Metric: `period` changed from MUST to MAY**
- New validation rule M8 requires at least one of `period` or `as_of`

**Comparison: weight-sum validation (M7)**
- Validator MUST warn when criteria weights deviate from 1.0 by more than 0.01
- Option `total` field added (optional pre-computed weighted score)

**Relationship types:**
- `blocked-by` added — explicit blocking direction for Action primitives with `status: blocked`

**Common fields:**
- `meta.updated` renamed to `meta.updated_at` (consistency with `status.updated_at`)
- `render` object added — optional rendering hints for display tools

**Composition model:**
- Section 11.4: Collection document type formalized

**New sections:**
- Section 16: Rendering Hints
- Section 17: Tooling Spec (Validator, Renderer, Converter)
- Section 18: Migration Guide V1.0 → V1.1

**New validation rules:**
- M7: Comparison criteria weight-sum validation (Standard level)
- M8: Metric must have `period` or `as_of` (Minimal level)
- X5: Timeline dependency resolution for slug-pattern entries (Complete level)

**Enum registry:**
- `urgency` enum added: `immediate` / `this-week` / `this-month` / `informational`

**OPI Connection (Section 12):**
- Bidirectional concept mapping added (OPI → BP direction)
- Combined usage example added

---

### V1.0.0 (2026-02-21)

**New primitives:**
- Stakeholder atom (name, role, influence, stance, interests)
- Progress molecule (checkpoint, velocity, forecast, blockers, milestones, metrics, next_actions)

**Renamed fields (breaking from V0.5):**
- Decision: `who` → `decided_by`, `what` → `title`, `date` → `decided_date`
- Action: `who` → `owner`, `what` → `title`, `deadline` → `due_date`
- Risk: `description` → `threat`
- Timeline: `date` → `target_date`
- Status: `updated` → `updated_at`
- Status Update: `proposed_action` → `actions` (now a list)

**New features:**
- Common fields (bp, primitive, id, meta, refs) on all primitives
- Shared enum registry with 9 enumerations
- `trend` enum changed from directional (up/down) to evaluative (improving/declining)
- New `direction` enum (up/down/flat) for raw directional data
- Organism definitions with slot cardinalities
- Template definitions (Decision Paper, Quarterly Review, Weekly Status)
- Page concept for template instances
- Composition model (inline, reference, hybrid)
- 26 validation rules in 4 groups
- 4 conformance levels (Minimal, Basic, Standard, Complete)
- JSON Schema for automated validation
- Cross-reference system with 7 relationship types
- OPI Spec connection and cross-reference convention
- Extension mechanism via `x-` prefix

**Removed:**
- `blockers` field on Action (use `refs` with `rel: blocks` instead)
- `composed_of` as top-level molecule field in YAML files (moved to spec prose)
