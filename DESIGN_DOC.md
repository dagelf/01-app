## Executive summary

- **Problem**: Teams (and solo builders) lose time moving from idea → selection → validation → MVP → launch → revenue → ops because work is scattered across tools, governance is ad-hoc, evidence is low-quality, and automation is unsafe or opaque.
- **Mission**: Take an idea to “unicorn” (or any goal) through maximum altruism + pragmatic insight, optimizing for **time-to-market/time-to-revenue/time-to-impact**.
- **Primary users**: Tech-literate generalists first; later anyone. **The user is the core agent/principal.** AI agents are additional principals that act under the same permission + budget model.
- **Scope**: A SaaS platform that orchestrates and optionally executes the full project lifecycle via humans and configurable agents, with **lists-of-lists** permissions, budget controls, auditability, and zero-copy integrations.
- **Non-goals (v1)**:
  - **No hard non-budget gates** by default (the platform recommends/warns; budgets/time enforce).
  - **No single mandated workflow** (opinionated defaults + full configurability).
  - **No forced data duplication** (zero-copy default; cache summaries/metadata only).

---

## Personas + top user journeys

### Personas

- **Builder-Generalist**: turns vague ideas into shipped MVPs fast; edits objectives, rubrics, and agent configs.
- **Portfolio Selector**: runs many ideas through Selection, allocates budgets, funds/defunds quickly.
- **Agent Tuner (Arena operator)**: benchmarks agents, improves prompts/tooling, graduates autonomy.
- **Collaborator**: joins via lists; contributes research/build/marketing under scoped permissions.

### Top journeys (end-to-end)

- **Journey A: Idea → Selection → Validation → MVP (≤7 days)**
  - Create Project (goal + budget) → Selection squads research/score → choose candidates → Validation experiments → decision (`proceed|pivot|fork|kill`) → generate MVP tasks → build + ship → monitor telemetry/revenue.
- **Journey B: Portfolio selection**
  - Intake many ideas → run iterative Selection loops → compare via objective function + rubric + evidence quality → fund top candidates → fork when necessary.
- **Journey C: Launch → operate**
  - Launch plan → execute channel tasks (agents/humans) under budgets → track KPIs → iterate → operate with runbooks + incident workflows.
- **Journey D: Reuse + sharing**
  - Import/export component packs (agents/workflows/metrics) and reuse across projects without copying external artifacts.

---

## System concept: core objects

- **`User`**: primary principal; owns resources, lists, budgets, and projects.
- **`Agent`**: a principal with model endpoint + tool configuration + system prompt; belongs to lists; audited like users.
- **`Workspace`**: SaaS tenant boundary and top see-through container for sharing policies/resources (can represent “my shard”).
- **`Project`**: **a budgeted goal**; lifecycle state; contains artifacts/evidence/tasks/runs.
- **`Budget` + `LedgerEntry`**: time + cost caps; attribution by principal, tool/action, and run.
- **`Stage`**: lifecycle step (templated, customizable).
- **`GateDecision`**: `proceed | pivot | fork | kill` with rubric snapshot + rationale + evidence links.
- **`Idea`** (optional explicit object): intake seed; can be promoted into a Project or forked.
- **`ObjectiveFunction`**: visible equation + assumptions + weights; configurable per workspace/project.
- **`Rubric`**: weighted scoring + narrative justification.
- **`Evidence`**: typed claims with provenance, quality rating, and links to sources.
- **`Experiment`**: hypothesis → method → result → decision; produces evidence and/or telemetry.
- **`Task`**: unit of work assigned to principals; supports nesting (task trees).
- **`Workflow`**: long-running orchestration of tasks/experiments with checkpoints.
- **`ArtifactRef`**: zero-copy pointer to external artifact + cached summaries/metadata.
- **`KPI`**: definitions + thresholds + data source mapping.
- **`Risk`**: risk register items with triggers and mitigations.
- **`AuditEvent`**: append-only record of all actions, tool calls, permission changes, and budget spend.

---

## Workflow + governance

### Stage definitions (default template)

- **Intake**
  - **Entry**: idea statement + rough goal + initial budget request
  - **Exit**: initial objective function + rubric template + initial task graph proposal
- **Selection (iterable research/scoring loop)**
  - **Purpose**: cheaply explore many ideas before paying validation costs
  - **Exit**: for each idea: research artifacts, score, evidence snapshot, recommendation, and candidate list for Validation
- **Validation**
  - **Exit**: experiments run; telemetry/signups/revenue signals captured; gate decision: `proceed|pivot|fork|kill`
- **MVP Build**
  - **Exit**: deployable MVP + instrumentation + baseline KPIs
- **Launch**
  - **Exit**: channel execution + tracking + support readiness
- **Operate / Scale**
  - **Exit**: stable operations, automated runbooks, scaling guardrails
- **Retire**
  - **Exit**: deprecation plan, retention/deletion actions, learnings archived

### Governance model (lists-of-lists as the organizing primitive)

- **Access control** is attached to objects via **named lists** (which can contain users, agents, and other lists).
- **Inheritance**: parent object lists apply to child objects unless overridden.
- **Selection squads**: implemented as nested lists under a `research` parent list (e.g., `research/competitors`, `research/pricing`, `research/channels`).

### Gate outcomes (explicit)

- **Proceed**: continue with current plan/budget.
- **Pivot**: modify goal/objective/rubric; keep project identity.
- **Fork**: create a new project with a new budget/goal, inheriting selected components/artifact refs/evidence refs; divergence recorded.
- **Kill**: stop new spend; archive state; keep audit history.

---

## Decision framework

### Default objective (configurable)

- **Default**: maximize **risk-adjusted expected value per unit time**, under budget/time constraints.
- **Config requirements**:
  - **Visible equation** (versioned)
  - **Explicit assumptions** (linked/referenced)
  - **Chosen metrics** with rationale (human + agent-assisted)

### Rubric (hybrid)

- **Quantitative**: configurable weighted rubric producing a score.
- **Narrative**: short “why” memo + “what would change my mind.”
- **Evidence-linked**: every major claim is backed by evidence objects.

### Evidence weighting + quality

- **Primary evidence signals (default ordering)**: **usage telemetry**, **landing page signups**, **revenue** (others supported).
- **Quality score components**:
  - **Provenance rating**: user-judged (AI suggests; user tunes/overrides)
  - **Sample size + duration**
  - **Falsifiability**
  - **Cost to obtain**
  - **Replication / triangulation**
- **Behavior**: warn/recommend by default; budgets can enforce “no further spend” if configured.

---

## AI design: agents, boundaries, arena, and safety

### Agent model

- **An agent = model endpoint + tool config (MCP) + system prompt + name**.
- Agents are principals in lists-of-lists; they can be granted narrowly scoped permissions and budgets.

### Permission model (capabilities + `.ro` only)

There is no write/execute split and no `.act`.

- **Capability naming**: a capability is a string like `plan`, `rubric`, `task`, `smtp`, `ads.publish`, `github.merge`, `deploy.prod`, `stripe.charge`.
- **Access levels**:
  - **No capability**: no access.
  - **`<capability>.ro`**: read-only access for that capability’s scope (view drafts, view status/logs, view configs/results).
  - **`<capability>` (no suffix)**: full permission for that capability (create/update/perform/commit as defined by the capability).

**Examples**

- Drafting vs sending email is modeled as different capabilities:
  - `email.draft` allows creating/updating drafts.
  - `smtp` allows actually sending via the SMTP connector.
  - Read-only variants are `email.draft.ro` and `smtp.ro`.
- GitHub:
  - `github.pr` (open/update PRs)
  - `github.merge` (merge PRs)
  - Read-only variants: `github.pr.ro`, `github.merge.ro`

**Budgets/time are still hard constraints**: even if a principal has a capability, policy and funding can deny or limit actions (e.g., spend caps, tool-specific limits, approval policies).

### Human-in-the-loop (defaults)

- Required for granting high-risk capabilities (e.g., `smtp`, `ads.publish`, `deploy.prod`, `stripe.charge`) unless explicitly configured otherwise.
- Funding changes and budget increases require explicit approval policies (configurable).

### Agent Arena (graduation mechanism)

- Tracks cost/time/quality per agent and aggregates by list and lists-of-lists.
- Supports:
  - **Automated evaluation** (golden tasks, objective metrics, regression detection)
  - **Human oversight** (pairwise ranking, spot checks)
- Policies can reference Arena performance to recommend or auto-grant capabilities within budget envelopes.

---

## Architecture proposal (high-level)

### Principles

- **SaaS-first** multi-tenant, with workspace isolation.
- **Zero-copy default**: store references; cache summaries/metadata for UX and speed.
- **Audit-first**: every call and state mutation is attributable and browsable.
- **Composable and abstractable modules** with strong defaults.

### Services / modules

- **UI (dashboard-first)**: portfolio + project cockpit, Selection loop UX, evidence timeline, Arena, budgets, permissions editor.
- **Control Plane API**: object graph, stages/gates, objective/rubric engine, policy evaluation, import/export.
- **Execution Plane**: workflow orchestrator (long-running), stateless jobs, retries/checkpoints, tool proxy (MCP), run tracking.
- **Integrations layer**: MCP connectors, zero-copy artifact refs, telemetry ingestion.
- **Analytics**: KPI computation, portfolio views, Arena aggregation.
- **Audit**: append-only audit service + query UI.

### Persistence (Q13 concrete recommendation)

- **Primary store**: **Postgres**
- **Audit**: append-only `audit_events` (structured), with allowlisted payload retention; hashes where appropriate
- **Eventing**: transactional outbox (to add streaming later without rewrites)
- **Artifacts**: external refs + cached summaries/metadata (explicitly marked derived)
- **Telemetry**: start minimal; graduate to a columnar store (e.g., ClickHouse) when needed

### Config and log formats

- **Configs**: JSON (validated via JSON Schema; versioned)
- **Logs**: NDJSON (append-only; envelope + event-specific payload)

---

## Analytics plan

### Metrics taxonomy

- **Speed**: intake→selection, selection→validation, validation→MVP, MVP→launch
- **Learning**: experiments/day, time-to-decision, evidence quality trend
- **Efficiency**: cost per experiment, cost per decision, agent cost/time per task type
- **Outcome**: activation/retention/revenue, margin proxy
- **Portfolio**: throughput by stage, kill/pivot/fork rates, reuse rate

### Dashboards

- **Project cockpit**: spend burn-down, stage status, gate recommendations, KPIs, run logs
- **Selection board**: idea funnel, scores, evidence quality, “advance list”
- **Portfolio**: objective function view, ranks, budget allocation heatmap
- **Arena**: frontier charts (cost vs quality vs time), list-of-lists aggregates

---

## Security model

- **Isolation**: workspace-level tenant boundary.
- **Authorization**: lists-of-lists granting `<capability>` or `<capability>.ro`, scoped per object and connector.
- **Secrets**: bring-your-own keys, scoped per list; revocable; expiry supported.
- **Auditability**: every action produces `AuditEvent` with actor (user/agent), capability, cost, and references to inputs/outputs.

---

## MVP scope and roadmap

### v1 (ship the core loop)

- **Objects**: Workspace, Project, Stage, GateDecision (incl fork), Budget/Ledger, Lists-of-lists, Task/Workflow/Run, Evidence/Experiment, ArtifactRef, AuditEvent, Agent.
- **Selection loop UX** with research squads (nested lists).
- **Objective function + rubric** (visible, editable, versioned).
- **Capability model** (`<capability>` vs `<capability>.ro`) with budget enforcement and approval policies.
- **Audit UI** (search/filter by actor/capability/run).
- **Import/export packs** (JSON configs + NDJSON logs + refs).

### v2

- **Agent Arena** fully integrated with policy recommendations and regression testing.
- **Richer connector packs** (MCP presets), more see-through analytics.
- **Federated sharing** (signed packs, provenance).

### v3

- **Self-improving workflows**: agents propose workflow/rubric improvements and validate via experiments.
- **Operational automation**: incident playbooks, scaling advisors, cost optimizers.

---

## Open questions + assumptions

### Assumptions

- **ASSUMPTION**: Default objective is risk-adjusted EV/time; users can override per workspace/project.
- **ASSUMPTION**: Default gates are advisory except budget/time hard stops (configurable).
- **ASSUMPTION**: External artifacts are referenced; see-through cached summaries are allowed and clearly labeled derived.

### Open questions

- **Default approval policy** for high-risk capabilities (`smtp`, `ads.publish`, `github.merge`, `deploy.prod`, `stripe.charge`).
- **Provenance retention**: hashes-only vs payload retention per connector/event type.
- **Fork governance**: rate limits / naming / merge-back semantics (if any).

---

## Risks + mitigations

- **Over-configurability risk**: too many knobs.
  - **Mitigation**: ship “golden default packs” (Selection squad templates, objective presets, rubric presets).
- **Agent blast radius**: mis-sends, overspends, bad deploys.
  - **Mitigation**: capability scoping, budgets, allowlists, approvals, Arena graduation, exhaustive audit.
- **Data integrity under speed**: inconsistent state with many agents.
  - **Mitigation**: Postgres transactions, idempotent runs, checkpoints, append-only audit, outbox.
- **Adoption/trust**: users won’t trust automation.
  - **Mitigation**: transparency-first UI, evidence-linked decisions, reversible drafts, clear cost ledger.

---

## Decision Log

| Decision | Options | Chosen | Rationale | Owner | Date | Revisit trigger |
|---|---|---|---|---|---|---|
| Deployment default | Self-host / SaaS | **SaaS** | Default trust boundary and UX for broad adoption | Platform Architect | 2025-12-28 | Enterprise-only demand for self-host |
| Core principal | “Org-first” / User-first | **User is core agent/principal** | Matches your worldview and list-based sharing | Platform Architect | 2025-12-28 | Need for strict enterprise hierarchy |
| Stage model | Intake→Validation direct vs Selection loop | **Add Selection stage (iterable)** | Cheap research+scoring before validation spend | Platform Architect | 2025-12-28 | Users consistently bypass selection |
| Gate outcomes | proceed/pivot/kill vs +fork | **Include `fork`** | Enables parallel divergence with provenance and budgets | Platform Architect | 2025-12-28 | Fork spam / governance issues |
| Decision style | Rubric / Narrative / Hybrid | **Hybrid** | Visible equations + explainability + evidence discipline | Platform Architect | 2025-12-28 | Users reject numeric scoring |
| Permissions model | Roles / write-execute / capabilities | **Capabilities + `.ro` only** | Aligns with “can do” vs “read-only”; simplest rule set | Platform Architect | 2025-12-28 | Need role templates for enterprise |
| Storage | Event-sourced / RDBMS+Audit / Doc-first | **Postgres + append-only audit + outbox** | Fast MVP, strong audit, evolvable | Platform Architect | 2025-12-28 | Need full replay/time-travel as core |
| Config/log formats | JSON/TOML/etc | **JSON configs + NDJSON logs** | Uniform tooling; strict schema validation; append-only logs | Platform Architect | 2025-12-28 | Config ergonomics issues |

