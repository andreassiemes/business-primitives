# Business Primitives Specification

**Version:** 1.0.0
**Status:** Release
**License:** MIT
**Authors:** Andreas Siemes
**Date:** 2026-02-21

> A specification for composable, format-agnostic building blocks of business communication.

Business Primitives defines typed, structured units of business information — decisions, metrics, risks, actions, timelines, statuses, stakeholders — that compose into richer structures. Like Brad Frost's Atomic Design for UI, this spec applies layered composition to business communication: Atoms combine into Molecules, Molecules into Organisms, Organisms fill Templates, Templates become Pages.

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

---

## 4. Document Structure

A Business Primitives document is a YAML file with these top-level fields:

```yaml
bp: "1.0"                # Spec version (REQUIRED)
primitive: metric         # Primitive type (REQUIRED)
id: met-revenue-q1        # Machine-readable slug ID (optional)
meta:                     # Audit and lifecycle fields (optional)
  created: 2026-03-15
  updated: 2026-03-20
  created_by: "CFO"
  source: "Q1 Board Meeting"
  tags: [finance, q1]
  lifecycle: active
refs:                     # Cross-references to other primitives (optional)
  - id: dec-expansion
    rel: informs
# ... domain-specific fields follow
```

**Spec version** (`bp`) uses `"MAJOR.MINOR"` format. Parsers MUST reject documents with an unsupported major version.

**Primitive type** (`primitive`) identifies which atom or molecule definition governs the document's fields.

**ID** (`id`) is a machine-readable slug matching `^[a-z0-9][a-z0-9-]*$`. IDs MUST be unique within a page or collection.

---

## 5. Common Fields

These fields are available on ALL primitives regardless of type.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bp` | string | MUST | Spec version (`"1.0"`) |
| `primitive` | string | MUST | Primitive type name |
| `id` | string | MAY | Slug ID (`^[a-z0-9][a-z0-9-]*$`) |
| `meta.created` | date | MAY | Creation date (YYYY-MM-DD) |
| `meta.updated` | date | MAY | Last modification date |
| `meta.created_by` | string | MAY | Creator (person or role) |
| `meta.source` | string | MAY | Provenance or origin |
| `meta.tags` | list | MAY | Discovery and classification tags |
| `meta.lifecycle` | enum | MAY | `draft` / `active` / `superseded` / `archived` |
| `refs` | list | MAY | Cross-references (objects with `id` and `rel`) |

### Relationship Types

The `rel` field in `refs` entries uses these values:

| Relationship | Meaning |
|-------------|---------|
| `informs` | This primitive provides input to the referenced one |
| `triggers` | This primitive causes the referenced one to activate |
| `blocks` | This primitive prevents progress on the referenced one |
| `supports` | This primitive provides evidence or backing |
| `supersedes` | This primitive replaces the referenced one |
| `part-of` | This primitive is a component of the referenced one |
| `derived-from` | This primitive was created based on the referenced one |

### Shared Enum Registry

Enumerations used across multiple primitives are defined once here. Individual primitives reference these by name.

| Enum | Values | Used By |
|------|--------|---------|
| `trend` | `improving` / `stable` / `declining` / `volatile` | Metric, Status-Update |
| `direction` | `up` / `down` / `flat` | Metric (raw directional data) |
| `confidence` | `hypothesis` / `supported` / `strong` / `proven` | Argument |
| `priority` | `low` / `normal` / `high` / `critical` | Action |
| `severity` | `low` / `medium` / `high` / `critical` | Risk (probability, impact) |
| `rag` | `green` / `amber` / `red` | Status |
| `lifecycle` | `draft` / `active` / `superseded` / `archived` | Common (meta) |
| `influence` | `high` / `medium` / `low` | Stakeholder |
| `stance` | `champion` / `supporter` / `neutral` / `skeptic` / `blocker` | Stakeholder |

---

## 6. Atoms

Atoms are irreducible primitives. Each serves one semantic purpose and cannot be meaningfully decomposed further. Business Primitives defines 7 atoms.

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
bp: "1.0"
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

### 6.2 Metric

A metric is a number with context. Numbers without context are noise; metrics are signal.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | What is being measured |
| `value` | number | MUST | The measured value |
| `unit` | string | MUST | Unit of measurement (%, EUR, count, hours) |
| `period` | string | MUST | Time period this metric covers |
| `trend` | enum | MAY | `improving` / `stable` / `declining` / `volatile` |
| `direction` | enum | MAY | `up` / `down` / `flat` (raw directional data) |
| `target` | number | MAY | Target or benchmark value |
| `source` | string | MAY | Where this number comes from |

**Constraints:**
- `trend` is evaluative (improving = good regardless of direction). `direction` is raw (up = numerically increasing). Both MAY coexist.
- If `target` is provided, `unit` applies to both `value` and `target`.

**Example:**
```yaml
bp: "1.0"
primitive: metric
id: met-retention-q1
name: "Customer Retention Rate"
value: 94.2
unit: "%"
period: "Q1 2026"
trend: improving
direction: up
target: 95.0
source: "CRM Dashboard"
```

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
bp: "1.0"
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

**Constraints:**
- If `status` is `done`, `due_date` SHOULD be in the past or today.
- If `status` is `blocked`, a `refs` entry with `rel: blocks` SHOULD identify the blocker.

**Example:**
```yaml
bp: "1.0"
primitive: action
id: act-roadmap-q2
owner: "Product Manager"
title: "Finalize Q2 roadmap and share with stakeholders"
due_date: 2026-03-20
status: in_progress
priority: high
```

### 6.5 Timeline

A timeline milestone marks a point in time that matters, with explicit dependencies on what must happen before it.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `milestone` | string | MUST | Name of the milestone |
| `target_date` | date | MUST | Target date (YYYY-MM-DD) |
| `owner` | string | MUST | Person or role responsible |
| `status` | enum | MUST | `planned` / `on_track` / `at_risk` / `delayed` / `completed` |
| `dependencies` | list | MAY | What must be completed before this milestone |
| `deliverable` | string | MAY | What is produced at this milestone |

**Constraints:**
- If `status` is `completed`, `target_date` SHOULD be in the past or today.
- `dependencies` entries SHOULD reference primitive IDs where possible.

**Example:**
```yaml
bp: "1.0"
primitive: timeline
id: tl-mvp-launch
milestone: "MVP Launch"
target_date: 2026-06-01
owner: "Product Lead"
status: on_track
dependencies:
  - "API integration complete"
  - "Security audit passed"
deliverable: "Production-ready platform with core features"
```

### 6.6 Status

A status captures the current state of any entity — a project, team, system, or goal. Traffic-light simplicity with structured detail.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entity` | string | MUST | What is being assessed |
| `state` | enum | MUST | `green` / `amber` / `red` (RAG) |
| `detail` | string | MUST | Brief explanation of current state |
| `updated_at` | date | MUST | When this assessment was last updated |
| `owner` | string | MAY | Who is responsible for this entity |

**Constraints:**
- `updated_at` MUST NOT be in the future.

**Example:**
```yaml
bp: "1.0"
primitive: status
id: sts-migration
entity: "Data Platform Migration"
state: amber
detail: "On schedule but resource risk — 2 engineers on vacation in KW 14"
updated_at: 2026-03-10
owner: "Migration Lead"
```

### 6.7 Stakeholder

A stakeholder maps the people and roles that matter to a decision, project, or initiative. Influence and stance drive engagement strategy.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | Person or role name |
| `role` | string | MUST | Organizational role or title |
| `influence` | enum | MUST | `high` / `medium` / `low` |
| `stance` | enum | MUST | `champion` / `supporter` / `neutral` / `skeptic` / `blocker` |
| `interests` | list | MAY | What this stakeholder cares about |

**Constraints:**
- No additional constraints beyond required fields and enum values.

**Example:**
```yaml
bp: "1.0"
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

## 7. Molecules

Molecules compose 2–3 atom types into a richer unit of meaning. A molecule conveys a message that no single atom can express alone. Per Design Principle 5, molecules SHOULD NOT combine more than 3 atom types.

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
bp: "1.0"
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

### 7.2 Comparison

**Composed of:** Metric, Risk, Stakeholder

A comparison structures the evaluation of alternatives. Instead of pros/cons lists or gut feelings, it provides criteria-based assessment with a clear recommendation.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `question` | string | MUST | What decision does this comparison support |
| `options` | list | MUST | The alternatives being compared (objects with `name` and `scores`) |
| `criteria` | list | MUST | Evaluation dimensions (objects with `name` and `weight`) |
| `recommendation` | string | MUST | Which option is recommended and why |

**Constraints:**
- `options` MUST contain at least 2 items.
- `criteria` weights SHOULD sum to 1.0.
- Each option's `scores` keys MUST match the `criteria` names.

**Example:**
```yaml
bp: "1.0"
primitive: comparison
id: cmp-cicd-platform
question: "Which CI/CD platform should we migrate to?"
options:
  - name: "GitHub Actions"
    scores: { cost: 8, integration: 9, learning_curve: 7 }
  - name: "GitLab CI"
    scores: { cost: 7, integration: 6, learning_curve: 8 }
  - name: "Jenkins"
    scores: { cost: 9, integration: 5, learning_curve: 4 }
criteria:
  - { name: cost, weight: 0.3 }
  - { name: integration, weight: 0.5 }
  - { name: learning_curve, weight: 0.2 }
recommendation: "GitHub Actions — best integration score outweighs minor cost difference"
```

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
bp: "1.0"
primitive: status-update
id: su-velocity-kw12
metric:
  name: "Sprint Velocity"
  value: 42
  unit: "story points"
trend: declining
assessment: "Third consecutive sprint below target. Team capacity reduced by parental leave."
action_needed: true
actions:
  - "Adjust sprint commitment to 35 SP until team is back to full capacity in KW 16"
```

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
bp: "1.0"
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

## 8. Organisms

Organisms are section-level groupings of molecules and atoms that serve a specific communication purpose. They are not standalone YAML files but patterns for composing primitives into meaningful sections.

An organism defines **slots** — typed positions that accept specific primitives with cardinality constraints.

| Organism | Slots | Purpose |
|----------|-------|---------|
| **Executive Summary** | `status: 1`, `metrics: 1..3`, `decisions: 0..3`, `risks: 0..3` | Document overview at maximum density |
| **Risk Register** | `risks: 1..n`, `actions: 0..n` | Risk table with mitigations and response actions |
| **Action Plan** | `actions: 1..n`, `timelines: 0..n` | Next steps with milestones and dependencies |
| **Performance Dashboard** | `status-updates: 1..n`, `metrics: 0..n` | KPI tracking with trends and assessments |

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

---

## 10. Pages

A page is a concrete instance of a template, filled with actual data. It is the final level of the composition hierarchy — what people actually read, present, or review.

```yaml
bp: "1.0"
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
actions:
  - $ref: "act-hire-dach"
  - owner: "Product Manager"
    title: "Launch self-service onboarding"
    due_date: 2026-05-15
    status: in_progress
```

### Composition Rules

1. **Atoms compose into Molecules.** A molecule's `composed_of` declaration specifies which atom types it uses.
2. **Molecules and Atoms compose into Organisms.** An organism's slot definition specifies accepted types and cardinalities.
3. **Organisms compose into Templates.** A template's section definition specifies required and optional organisms.
4. **Templates instantiate into Pages.** A page references its template and fills sections with data.
5. **No level skipping.** An atom MUST NOT appear directly in a template section without an organism wrapper.
6. **References resolve within scope.** A `$ref` resolves within the current page or collection. Cross-page references use `refs` with `rel`.
7. **Inline primitives inherit context.** An inline primitive within a molecule inherits the molecule's `meta` and `refs` unless it defines its own.
8. **Extension fields are preserved.** Custom `x-` fields MUST be preserved through composition and rendering.

---

## 12. Connection to OPI Spec

Business Primitives and the Organizational Programming Interface (OPI) are complementary specifications by the same author. OPI defines organizational structure (roles, circles, flows); BP defines organizational communication (decisions, metrics, risks).

### Concept Mapping

| BP Concept | OPI Concept | Relationship |
|-----------|------------|-------------|
| Decision | `decision` within a Circle/Flow | Same data, different context |
| Metric | KPI in a Role's accountabilities | BP provides the structured format |
| Risk | Risk in a Decision Flow | BP adds probability/impact semantics |
| Action | Task in a Role's domain | BP adds priority/status tracking |
| Stakeholder | Role/Person | BP adds influence/stance analysis |
| Timeline | Milestone in a Decision Flow | BP adds dependency tracking |
| Status | Health of any OPI entity | BP provides the assessment format |

### Cross-Reference Convention

Documents may reference across specs using prefixed IDs:

```yaml
refs:
  - id: "opi:circle-engineering"
    rel: part-of
  - id: "bp:dec-platform-migration"
    rel: triggers
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

### Semantic Rules (M1–M6)

| Rule | Description | Level |
|------|-------------|-------|
| M1 | If metric has both `trend` and `direction`, they MUST NOT contradict (declining + up is valid for cost metrics) | Standard |
| M2 | If action `status` is `done`, a completion indicator SHOULD exist | Standard |
| M3 | Risk `probability` and `impact` MUST both be present (not one without the other) | Minimal |
| M4 | If `risk_score` is present, it MUST equal `"{probability}-{impact}"` | Basic |
| M5 | Status `updated_at` MUST NOT be in the future | Minimal |
| M6 | Decision with `status: superseded` SHOULD have a `refs` entry with `rel: supersedes` | Basic |

### Cross-Reference Rules (X1–X4)

| Rule | Description | Level |
|------|-------------|-------|
| X1 | `refs` target IDs SHOULD resolve to existing primitives in the collection | Complete |
| X2 | Circular reference chains MUST NOT exist (A refs B refs A) | Complete |
| X3 | IDs MUST be unique within a page | Standard |
| X4 | Cross-spec references (opi:/bp:) MUST use the correct prefix | Complete |

---

## 14. Conformance Levels

Implementations and documents may claim conformance at one of four levels. Each level includes all rules from lower levels.

| Level | Requirements | Typical Use |
|-------|-------------|-------------|
| **Minimal** | Atoms only, required fields, structural rules S1–S6, semantic rule M3/M5 | Quick capture, drafts, manual authoring |
| **Basic** | + `id`, `meta`, `refs` fields; rules S7–S8, M4, M6 | Team collaboration, version-controlled docs |
| **Standard** | + Molecules, Organisms, Templates; composition rules C1–C8, semantic rules M1–M2, cross-ref X3 | Production documents, CI validation |
| **Complete** | + Pages, full cross-reference resolution; rules X1–X2, X4; JSON Schema validation | Tooling, renderers, automated pipelines |

---

## 15. JSON Schema

A JSON Schema for Business Primitives 1.0 is provided at `bp-v1.0.schema.json`. It validates all atom and molecule types at the Standard conformance level.

**Usage with IDE:**

```json
{
  "$schema": "./spec/bp-v1.0.schema.json"
}
```

**Usage with CI:**

```bash
# Validate a single file
ajv validate -s spec/bp-v1.0.schema.json -d examples/decision-paper.yaml

# Validate all YAML files
find . -name '*.yaml' -exec ajv validate -s spec/bp-v1.0.schema.json -d {} \;
```

---

## 16. Changelog

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
