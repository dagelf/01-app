# Platform Lifecycle Design Document
## Codename: **UNICORN FORGE**

**Version:** 0.1.0-draft  
**Date:** 2024-12-28  
**Status:** Design Phase  
**Owner:** Platform Architecture Team

---

## Executive Summary

### Problem Statement
Current product lifecycle tooling is fragmented, expensive, slow, and requires significant human coordination overhead. The gap between "idea" and "revenue" typically spans months/years with bloated processes, duplicated work, and vanity metrics masquerading as validation.

### Solution
**Unicorn Forge** is an agent-orchestrated, zero-copy, shardable platform that takes projects from idea validation through full lifecycle execution (market research → MVP → iteration → marketing → launch → revenue → scaling → operations) with:
- **Idea → MVP in <7 days** (not weeks/months)
- **Cycle times of hours** (not sprints)
- **Near-zero marginal cost** at massive scale
- **Agent-first architecture** where LLM agents execute work, humans steer budgets

### Users
Tech-literate generalists who excel across disciplines (product, engineering, marketing, research) and can appreciate, tweak, and improve platform function. Initially power-users; later accessible to broader audiences through refined UX.

### Value Proposition
| Traditional Approach | Unicorn Forge |
|---------------------|---------------|
| 6-18 month cycles | <7 day idea→MVP |
| $50K-500K burn to validate | Near-zero validation cost |
| Manual coordination | Agent-orchestrated parallelism |
| Siloed tools | Zero-copy, composable primitives |
| One-size-fits-all | User-steered budgets & agent selection |

### Scope
**In Scope:**
- Idea intake, validation, and portfolio prioritization
- Research orchestration (interviews, surveys, competitor analysis, pricing tests)
- PRD generation and agent-parallel job decomposition
- MVP specification and agent-driven build coordination
- Launch, marketing, and revenue operations
- Self-improving agent arena
- Shardable multi-instance federation

**Non-Goals (v1):**
- Enterprise SSO/SAML (single role model initially)
- Native IDE or CI/CD (agents use external tools via MCP)
- Regulated domain compliance (health/finance/kids—future)
- Mobile-native apps (web-first)

---

## Personas & User Journeys

### Primary Persona: The Generalist Builder
**Profile:** Technical founder, indie hacker, or full-stack PM who can code, market, and close deals. Values speed over polish. Comfortable tweaking prompts and agent configurations.

**Key Behaviors:**
- Submits 5-20 ideas per month
- Wants brutal, fast validation (kill bad ideas in hours, not weeks)
- Manages budget across agents like allocating cloud compute
- Exports/imports project states between instances

### Secondary Persona: The Agent Operator
**Profile:** AI/ML practitioner focused on optimizing agent performance. Runs arena experiments, compares prompts, measures cost/quality tradeoffs.

**Key Behaviors:**
- Clones and forks agent configurations
- Runs A/B tests on agent prompts
- Monitors agent performance dashboards
- Contributes improved agents to shared pools

### Journey 1: Idea → Validated MVP (Target: <7 days)

```
Day 0 (Hour 0-2): IDEA INTAKE
├── User submits idea (2-3 adaptive yes/no questions)
├── Platform auto-generates initial PRD draft
├── Agent decomposes PRD into parallel job graph
└── User sets budget envelope ($0-100 / $100-1K / custom)

Day 0-1 (Hour 2-24): PARALLEL VALIDATION
├── Research Agent: TAM analysis, competitor scan
├── Interview Agent: Schedules/conducts 5 user interviews (or synthesizes from public data)
├── Pricing Agent: Runs willingness-to-pay analysis
├── Feasibility Agent: Technical complexity scoring
└── All agents write to shared evidence log (zero-copy)

Day 1-2: GATE DECISION
├── Platform synthesizes evidence into Go/No-Go recommendation
├── User reviews dashboard, adjusts weights if desired
├── Decision logged with full audit trail
└── If No-Go: Idea archived with learnings; If Go: Proceeds to MVP

Day 2-5: MVP BUILD
├── Spec Agent: Generates detailed MVP specification
├── Architecture Agent: Proposes stack, generates scaffold
├── Build Agents: Execute parallel coding tasks (via MCP → GitHub/Cursor/Replit)
├── QA Agent: Generates and runs test suites
└── User reviews, steers, approves incremental outputs

Day 5-7: LAUNCH PREP
├── Marketing Agent: Guerrilla marketing plan (Hormozi/Levelsio tactics)
├── Launch Agent: Submits to app directories, ProductHunt, HN
├── Pricing Agent: Configures payment (Stripe/Gumroad/LemonSqueezy)
└── Revenue tracking initialized

Day 7+: OPERATE & ITERATE
├── Metrics Agent: Monitors activation, retention, LTV/CAC
├── Feedback Agent: Aggregates user feedback into prioritized backlog
├── Iteration cycles of hours (agent-driven PRD updates → parallel builds)
└── Scale or Sunset decision based on metrics
```

### Journey 2: Agent Arena Optimization

```
1. User identifies underperforming agent (e.g., Research Agent too slow)
2. Enters Arena Mode for that agent type
3. Clones current agent config, creates variant with modified prompt
4. Runs both against standardized test cases
5. Platform measures: Time, Cost (tokens), Quality (rubric or user rating)
6. Winner becomes new default; loser archived
7. Leaderboard tracks best agents per task type
```

### Journey 3: Instance Federation

```
1. User A runs instance with 10 projects
2. User B runs separate instance with 5 projects
3. They agree to federate (share access via standard adapter)
4. Projects visible across instances (read-only or configurable)
5. Shared learnings: "This agent config works well for SaaS MVPs"
6. Budget pools can be shared or siloed
```

---

## System Concept: Core Objects

### Object Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        INSTANCE                                 │
│  (Shardable, federable deployment unit)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │  USER   │───▶│ BUDGET  │───▶│  AGENT  │───▶│   JOB   │     │
│  │         │    │  POOL   │    │ CONFIG  │    │         │     │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│       │                             │              │           │
│       │                             │              ▼           │
│       │                             │        ┌─────────┐       │
│       │                             └───────▶│ARTIFACT │       │
│       │                                      └─────────┘       │
│       ▼                                           │            │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐       │            │
│  │ PROJECT │───▶│  STAGE  │───▶│  GATE   │◀──────┘            │
│  │         │    │         │    │         │                     │
│  └─────────┘    └─────────┘    └─────────┘                     │
│       │              │              │                          │
│       │              ▼              ▼                          │
│       │        ┌─────────┐    ┌─────────┐                      │
│       │        │EVIDENCE │    │DECISION │                      │
│       │        │         │    │         │                      │
│       │        └─────────┘    └─────────┘                      │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │  KPI    │    │ LAUNCH  │    │REVENUE  │    │  RISK   │     │
│  │         │    │  EVENT  │    │  MODEL  │    │         │     │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Object Definitions

| Object | Description | Key Attributes |
|--------|-------------|----------------|
| **Instance** | Shardable deployment unit; can federate with others | `id`, `owner`, `federation_keys[]`, `config` |
| **User** | Single-role participant; shares ranged API key access | `id`, `api_keys[]`, `budget_allocations[]`, `access_grants[]` |
| **Budget Pool** | Allocated resources ($ or tokens) for agent work | `id`, `owner`, `amount`, `spent`, `hard_limit`, `soft_limit` |
| **Agent Config** | LLM endpoint + MCP config + system prompt + name | `id`, `name`, `llm_endpoint`, `mcp_servers[]`, `system_prompt`, `version` |
| **Job** | Unit of work assigned to an agent | `id`, `agent_config_id`, `input`, `output`, `status`, `cost`, `duration`, `quality_score` |
| **Project** | Container for an idea's full lifecycle | `id`, `name`, `current_stage`, `budget_pool_id`, `metadata` |
| **Stage** | Lifecycle phase (Intake, Validation, Build, Launch, Operate, Sunset) | `id`, `name`, `entry_criteria`, `exit_criteria`, `artifacts_required[]` |
| **Gate** | Decision point between stages | `id`, `stage_from`, `stage_to`, `decision_criteria`, `approval_mode` |
| **Evidence** | Research/experiment output supporting decisions | `id`, `type`, `source`, `confidence`, `data`, `job_id` |
| **Artifact** | Deliverable (PRD, spec, code, assets, etc.) | `id`, `type`, `content_ref`, `version`, `job_id` |
| **Decision** | Recorded go/no-go with rationale | `id`, `gate_id`, `outcome`, `evidence_ids[]`, `rationale`, `decider` |
| **KPI** | Tracked metric with target and actual | `id`, `name`, `target`, `actual`, `timestamp` |
| **Launch Event** | Submission to channel (ProductHunt, AppStore, etc.) | `id`, `channel`, `status`, `metrics`, `timestamp` |
| **Revenue Model** | Pricing configuration (subscription, one-time, usage) | `id`, `type`, `tiers[]`, `provider_config` |
| **Risk** | Identified risk with likelihood/impact/mitigation | `id`, `description`, `likelihood`, `impact`, `mitigation`, `status` |

---

## Workflow & Governance

### Stage Definitions

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           LIFECYCLE STAGES                               │
├──────────┬───────────┬──────────┬──────────┬───────────┬────────────────┤
│  INTAKE  │ VALIDATE  │  BUILD   │  LAUNCH  │  OPERATE  │    SUNSET      │
│  (Hours) │  (1-2d)   │  (2-4d)  │  (1-2d)  │  (Ongoing)│   (When due)   │
└──────────┴───────────┴──────────┴──────────┴───────────┴────────────────┘
     │           │           │          │           │            │
     ▼           ▼           ▼          ▼           ▼            ▼
  PRD Draft   Evidence    MVP Live   Revenue    Metrics      Archive
  Job Graph   Synthesis   Deployed   Flowing    Dashboard    Learnings
```

| Stage | Entry Criteria | Exit Criteria | Required Artifacts | Typical Duration |
|-------|---------------|---------------|-------------------|------------------|
| **Intake** | Idea submitted | PRD draft + job graph approved | `idea_input`, `prd_draft`, `job_graph` | 1-4 hours |
| **Validate** | Budget allocated | Evidence synthesis complete, gate decision made | `research_summary`, `evidence_log`, `gate_decision` | 1-2 days |
| **Build** | Go decision | MVP deployed, QA passed | `mvp_spec`, `codebase`, `test_results`, `deploy_config` | 2-4 days |
| **Launch** | MVP ready | At least one channel live, revenue config active | `marketing_plan`, `launch_submissions`, `pricing_config` | 1-2 days |
| **Operate** | Revenue possible | Sunset criteria met OR scale decision | `kpi_dashboard`, `feedback_backlog`, `iteration_log` | Ongoing |
| **Sunset** | Metrics below threshold OR strategic decision | Archived with learnings | `postmortem`, `learnings_export` | 1 day |

### Gate Decision Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Auto-Approve** | If all criteria pass, proceed automatically | Low-budget experiments, high-confidence patterns |
| **Recommend** | Platform recommends, human confirms with 1-click | Default for most gates |
| **Require** | Human must review evidence and explicitly approve | High-budget, high-risk, or strategic initiatives |

**ASSUMPTION:** Default gate mode is **Recommend** with 24-hour auto-escalation if no human response.

### Kill Switches

| Trigger | Action | Reversible? |
|---------|--------|-------------|
| Hard budget exceeded | All agents paused, user notified | Yes, with budget increase |
| Time constraint exceeded | Gate auto-fails, decision required | Yes, with time extension |
| Quality score below threshold (3 consecutive jobs) | Agent flagged, fallback agent activated | Yes, after agent review |
| User explicit kill | Project archived immediately | Yes, within 7 days |

---

## Decision Framework

### Scoring Rubric (User-Configurable)

Default weights (user can override):

| Criterion | Weight | Measurement |
|-----------|--------|-------------|
| **TAM (Total Addressable Market)** | 20% | Agent-estimated market size |
| **Willingness to Pay** | 25% | Interview/survey signals |
| **Technical Feasibility** | 15% | Complexity score (1-10) |
| **Time to Market** | 20% | Estimated days to MVP |
| **Strategic Fit** | 10% | User-defined rubric |
| **Margin Potential** | 10% | Revenue model analysis |

### Evidence Weighting

| Evidence Type | Base Confidence | Adjustments |
|---------------|-----------------|-------------|
| User interview (live) | 0.8 | +0.1 if paying customer, -0.2 if friend/family |
| User interview (synthetic/public data) | 0.4 | +0.2 if verified source |
| Survey (n>100) | 0.7 | +0.1 per 100 additional respondents |
| Survey (n<100) | 0.4 | Linear scale |
| Competitor analysis | 0.6 | +0.1 if primary sources used |
| Landing page test | 0.7 | +0.2 if payment intent captured |
| Pricing test (actual transactions) | 0.9 | Highest confidence |

### Portfolio Prioritization

When multiple projects compete for resources:

```
Priority Score = (
    Validation_Score × 0.4 +
    Expected_Revenue × 0.3 +
    Time_to_Market_Inverse × 0.2 +
    Strategic_Fit × 0.1
) × Budget_Efficiency_Multiplier

Budget_Efficiency_Multiplier = 1 + (Budget_Remaining / Budget_Allocated)
```

**Deduplication/Synergy Detection:**
- Platform scans for overlapping TAM, similar tech stacks, shared customer segments
- Suggests merges, shared components, or explicit sequencing
- Flags when two projects might cannibalize each other

---

## AI Design

### Agent Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      AGENT RUNTIME                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    AGENT CONFIG                          │   │
│  │  ┌─────────────┬─────────────┬─────────────────────┐    │   │
│  │  │ LLM Endpoint│ MCP Servers │   System Prompt     │    │   │
│  │  │ (configurable)│ (tool access)│  (task-specific)   │    │   │
│  │  └─────────────┴─────────────┴─────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    JOB EXECUTOR                          │   │
│  │  1. Receive job input                                    │   │
│  │  2. Execute via LLM + MCP tools                         │   │
│  │  3. Validate output against schema                       │   │
│  │  4. Write artifacts (zero-copy reference)               │   │
│  │  5. Log cost, duration, quality                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    ARTIFACT STORE                        │   │
│  │  (Content-addressable, zero-copy, versioned)            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Agent Types (Default Set)

| Agent | Purpose | MCP Tools |
|-------|---------|-----------|
| **PRD Agent** | Generate/refine product requirements | `filesystem`, `web_search`, `memory` |
| **Decomposer Agent** | Split PRD into parallel job graph | `graph_builder`, `dependency_analyzer` |
| **Research Agent** | TAM, competitor, market analysis | `web_search`, `data_extraction`, `calculator` |
| **Interview Agent** | Conduct/synthesize user interviews | `calendar`, `video_call`, `transcription`, `web_search` |
| **Pricing Agent** | WTP analysis, pricing model design | `calculator`, `ab_test_runner`, `payment_apis` |
| **Feasibility Agent** | Technical complexity scoring | `codebase_analyzer`, `stack_detector`, `web_search` |
| **Spec Agent** | Detailed MVP specification | `filesystem`, `diagram_generator`, `memory` |
| **Architecture Agent** | Stack selection, scaffold generation | `codebase_generator`, `template_library`, `web_search` |
| **Build Agent** | Code generation and modification | `github`, `filesystem`, `terminal`, `browser` |
| **QA Agent** | Test generation and execution | `test_runner`, `browser`, `terminal` |
| **Marketing Agent** | Guerrilla marketing plans, copy | `social_apis`, `content_generator`, `web_search` |
| **Launch Agent** | Platform submissions | `producthunt_api`, `appstore_apis`, `web_forms` |
| **Metrics Agent** | KPI tracking and alerting | `analytics_apis`, `database`, `alerting` |
| **Feedback Agent** | User feedback aggregation | `support_apis`, `sentiment_analyzer`, `prioritizer` |

### Agent Arena Mode

```
┌─────────────────────────────────────────────────────────────────┐
│                      ARENA EXPERIMENT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Test Case Set (standardized inputs for agent type)            │
│       │                                                         │
│       ├──▶ Agent A (baseline config)                           │
│       │         └──▶ Output A + Cost A + Time A + Quality A    │
│       │                                                         │
│       ├──▶ Agent B (variant config)                            │
│       │         └──▶ Output B + Cost B + Time B + Quality B    │
│       │                                                         │
│       └──▶ Agent C (variant config)                            │
│                 └──▶ Output C + Cost C + Time C + Quality C    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    SCORING                               │   │
│  │  Quality: User rating (1-5) or rubric score             │   │
│  │  Cost: Token usage × rate                                │   │
│  │  Time: Wall-clock seconds                                │   │
│  │  Composite = Quality × 0.5 - Cost × 0.3 - Time × 0.2    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Winner → New default (with rollback capability)               │
│  Loser → Archived with learnings                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Human-in-the-Loop Points

| Checkpoint | Default Behavior | Override |
|------------|------------------|----------|
| Budget threshold (50%) | Notify user, continue | Pause and require approval |
| Gate decision | Recommend, user confirms | Auto-approve if confidence >0.9 |
| External communication (email, social) | Draft for approval | **HARD NO: Never auto-send** |
| Payment/financial actions | Draft for approval | **HARD NO: Never auto-execute** |
| Code deployment | Stage for review | Auto-deploy to preview only |
| Agent config changes | Require approval | Auto-apply in Arena only |

### Safety Policies

**Hard Constraints (Never Violate):**
1. Never exceed hard budget limits
2. Never exceed hard time constraints
3. Never auto-send external communications
4. Never auto-execute financial transactions
5. Never modify agent configs outside Arena without approval
6. Never access resources beyond granted API key scope

**Soft Constraints (Warn and Proceed):**
1. Approaching budget limits (>80%)
2. Quality scores declining trend
3. Unusual token consumption patterns
4. External API rate limiting

---

## Architecture Proposal

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │  Web App    │  │  CLI        │  │  API        │  │  Federation │   │
│  │  (Dashboard)│  │  (Power     │  │  (External  │  │  (Instance  │   │
│  │             │  │   Users)    │  │   Integr.)  │  │   Sync)     │   │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘   │
└─────────┼────────────────┼────────────────┼────────────────┼───────────┘
          │                │                │                │
          ▼                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY                                   │
│  • Authentication (API key-based, ranged access)                       │
│  • Rate limiting                                                        │
│  • Request logging (every call → audit log)                            │
│  • Request routing                                                      │
└─────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         CORE SERVICES                                   │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐              │
│  │   Project     │  │    Agent      │  │     Job       │              │
│  │   Service     │  │   Service     │  │   Orchestrator│              │
│  │               │  │               │  │               │              │
│  │ • CRUD        │  │ • Config CRUD │  │ • Job queue   │              │
│  │ • Lifecycle   │  │ • Arena       │  │ • Execution   │              │
│  │ • Gates       │  │ • Versioning  │  │ • Retry       │              │
│  └───────────────┘  └───────────────┘  └───────────────┘              │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐              │
│  │   Evidence    │  │   Budget      │  │   Federation  │              │
│  │   Service     │  │   Service     │  │   Service     │              │
│  │               │  │               │  │               │              │
│  │ • Store       │  │ • Allocation  │  │ • Sync        │              │
│  │ • Query       │  │ • Tracking    │  │ • Discovery   │              │
│  │ • Aggregate   │  │ • Alerts      │  │ • Adapters    │              │
│  └───────────────┘  └───────────────┘  └───────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        AGENT RUNTIME                                    │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                    MCP GATEWAY                                 │     │
│  │  • Route tool calls to configured MCP servers                 │     │
│  │  • Credential injection (from user's API key grants)         │     │
│  │  • Response normalization                                     │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                              │                                          │
│         ┌────────────────────┼────────────────────┐                    │
│         ▼                    ▼                    ▼                    │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐            │
│  │ MCP: GitHub │      │ MCP: Web    │      │ MCP: Custom │            │
│  │             │      │   Search    │      │             │            │
│  └─────────────┘      └─────────────┘      └─────────────┘            │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                      │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐              │
│  │   Primary DB  │  │   Artifact    │  │   Audit Log   │              │
│  │   (Postgres/  │  │   Store       │  │   (Append-    │              │
│  │    SQLite)    │  │   (S3/Local)  │  │    Only)      │              │
│  │               │  │               │  │               │              │
│  │ Objects,      │  │ Content-      │  │ Every API     │              │
│  │ Relations,    │  │ addressable,  │  │ call logged,  │              │
│  │ Metadata      │  │ zero-copy     │  │ browsable     │              │
│  └───────────────┘  └───────────────┘  └───────────────┘              │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────┐     │
│  │                    EVENT BUS                                   │     │
│  │  • Job completion events                                      │     │
│  │  • Stage transition events                                    │     │
│  │  • Budget threshold events                                    │     │
│  │  • Federation sync events                                     │     │
│  └───────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Data Model (Simplified)

```sql
-- Core entities
CREATE TABLE instances (
    id UUID PRIMARY KEY,
    owner_id UUID,
    config JSONB,
    federation_keys TEXT[],
    created_at TIMESTAMPTZ
);

CREATE TABLE users (
    id UUID PRIMARY KEY,
    instance_id UUID REFERENCES instances(id),
    api_keys JSONB,  -- encrypted
    access_grants JSONB,  -- what others can access
    created_at TIMESTAMPTZ
);

CREATE TABLE budget_pools (
    id UUID PRIMARY KEY,
    owner_id UUID REFERENCES users(id),
    amount_cents BIGINT,
    spent_cents BIGINT,
    hard_limit_cents BIGINT,
    soft_limit_cents BIGINT,
    created_at TIMESTAMPTZ
);

CREATE TABLE agent_configs (
    id UUID PRIMARY KEY,
    name TEXT,
    llm_endpoint TEXT,
    mcp_servers JSONB,
    system_prompt TEXT,
    version INT,
    parent_id UUID REFERENCES agent_configs(id),
    is_default BOOLEAN,
    created_at TIMESTAMPTZ
);

CREATE TABLE projects (
    id UUID PRIMARY KEY,
    instance_id UUID REFERENCES instances(id),
    name TEXT,
    current_stage TEXT,
    budget_pool_id UUID REFERENCES budget_pools(id),
    metadata JSONB,
    created_at TIMESTAMPTZ
);

CREATE TABLE jobs (
    id UUID PRIMARY KEY,
    project_id UUID REFERENCES projects(id),
    agent_config_id UUID REFERENCES agent_configs(id),
    input JSONB,
    output JSONB,
    status TEXT,  -- pending, running, completed, failed
    cost_tokens BIGINT,
    duration_ms BIGINT,
    quality_score FLOAT,
    created_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
);

CREATE TABLE artifacts (
    id UUID PRIMARY KEY,
    job_id UUID REFERENCES jobs(id),
    type TEXT,
    content_hash TEXT,  -- content-addressable
    content_ref TEXT,   -- pointer to artifact store
    version INT,
    created_at TIMESTAMPTZ
);

CREATE TABLE evidence (
    id UUID PRIMARY KEY,
    project_id UUID REFERENCES projects(id),
    job_id UUID REFERENCES jobs(id),
    type TEXT,
    source TEXT,
    confidence FLOAT,
    data JSONB,
    created_at TIMESTAMPTZ
);

CREATE TABLE decisions (
    id UUID PRIMARY KEY,
    project_id UUID REFERENCES projects(id),
    gate_id TEXT,
    outcome TEXT,  -- go, no_go, defer
    evidence_ids UUID[],
    rationale TEXT,
    decider_id UUID REFERENCES users(id),
    decider_type TEXT,  -- human, agent, auto
    created_at TIMESTAMPTZ
);

CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    user_id UUID,
    action TEXT,
    resource_type TEXT,
    resource_id UUID,
    request JSONB,
    response JSONB,
    ip_address INET,
    retention_until TIMESTAMPTZ
);

-- Zero-copy: artifacts reference content by hash
-- Federation: instances sync via event bus + adapter protocol
```

### Technology Recommendations

| Layer | Recommended | Rationale |
|-------|-------------|-----------|
| **Frontend** | React + Vite + TailwindCSS | Fast, modern, single-page dashboard |
| **API** | FastAPI (Python) or Hono (TypeScript) | Async-first, fast, good typing |
| **Database** | PostgreSQL (cloud) / SQLite (local) | Proven, flexible, good JSONB support |
| **Artifact Store** | S3-compatible (cloud) / Local filesystem | Content-addressable, cheap storage |
| **Job Queue** | BullMQ (Redis) or Temporal | Durable, supports complex workflows |
| **Event Bus** | Redis Streams or NATS | Lightweight, fast, supports pub/sub |
| **LLM Gateway** | LiteLLM or custom | Abstracts multiple LLM providers |
| **MCP Runtime** | Node.js or Python SDK | Official MCP implementations |

**ASSUMPTION:** Start with SQLite + local filesystem for single-instance MVP, migrate to Postgres + S3 for scale.

---

## Analytics Plan

### Metrics Taxonomy

| Category | Metric | Definition | Target |
|----------|--------|------------|--------|
| **Speed** | Idea-to-MVP Time | Days from intake to deployed MVP | <7 days |
| **Speed** | Cycle Time | Hours per iteration loop | <4 hours |
| **Speed** | Gate Decision Time | Hours from evidence complete to decision | <24 hours |
| **Quality** | Validation Accuracy | % of Go decisions that achieve revenue | >60% |
| **Quality** | Agent Success Rate | % of jobs completing without retry | >90% |
| **Quality** | Evidence Confidence | Weighted avg confidence of evidence used | >0.7 |
| **Cost** | Cost per Validation | $ spent in Validate stage | <$50 |
| **Cost** | Cost per MVP | $ spent through Launch stage | <$500 |
| **Cost** | Token Efficiency | Output quality / tokens consumed | Trending up |
| **Portfolio** | Active Projects | Projects in Build/Launch/Operate | Track |
| **Portfolio** | Kill Rate | % of projects killed at Validate gate | 50-70% (healthy) |
| **Portfolio** | Revenue per Project | Avg monthly revenue of Operate projects | Trending up |

### Dashboard Views

**1. Portfolio Overview (Main Dashboard)**
- Pipeline funnel: counts per stage
- Budget allocation heatmap
- Active agents and job queue
- Recent decisions
- Alerts/warnings

**2. Project Detail**
- Current stage and progress
- Evidence log with confidence scores
- Job history with costs
- Artifact browser
- KPI charts (if in Operate)

**3. Agent Arena**
- Agent leaderboard by type
- Recent experiments with results
- Cost/quality tradeoff scatter plots
- Prompt diff viewer

**4. Audit Log Browser**
- Filterable API call log
- Request/response inspector
- Export capability

### Data Pipeline

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Events    │────▶│   Stream    │────▶│   Aggregate │────▶│   Dashboard │
│   (writes)  │     │   Processor │     │   (hourly)  │     │   (live)    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                              │
                                              ▼
                                        ┌─────────────┐
                                        │   Export    │
                                        │   (on-demand)│
                                        └─────────────┘
```

**ASSUMPTION:** No external data warehouse in v1; dashboards query primary DB directly with materialized views for aggregates.

---

## Security Model

### Authentication & Authorization

**Single Role Model:**
- Every user has full capabilities within their own scope
- No viewer/editor/approver hierarchy
- Access controlled via API key grants (ranged access)

**API Key Grants:**
```json
{
  "grant_id": "uuid",
  "granter_id": "user_a",
  "grantee_id": "user_b",
  "resources": {
    "projects": ["project_1", "project_2"],
    "agent_configs": ["*"],
    "budget_pools": {
      "pool_1": {"max_spend": 10000}
    }
  },
  "expires_at": "2025-12-31T00:00:00Z",
  "revocable": true
}
```

### Audit Logging

**Every API Call Logged:**
- Timestamp
- User ID (or API key identifier)
- Action (method + endpoint)
- Resource type and ID
- Request body (sensitive fields redacted)
- Response status
- IP address

**Configurable Retention:**
- Default: 90 days
- User-configurable: 30 days to unlimited
- Privacy deletion: on-demand purge of user-specific logs

**Frontend Browser:**
- Filterable by time, user, resource type, action
- Full request/response inspection
- Export to JSON/CSV

### Instance Isolation

- Each instance is fully isolated by default
- Federation requires explicit key exchange
- No cross-instance data access without federation
- Credentials stored encrypted at rest

### Compliance Notes

**ASSUMPTION:** v1 does not target SOC2/ISO/GDPR certification, but architecture supports future compliance:
- Audit logging ✓
- Data retention controls ✓
- Export/delete capability ✓
- Encryption at rest ✓ (planned)
- Encryption in transit ✓ (TLS required)

---

## MVP Scope & Roadmap

### v1.0 — Foundation (8-12 weeks)

**Must Ship:**
1. **Core workflow:** Intake → Validate → Build → Launch (4 stages)
2. **Agent runtime:** Execute agents via configurable LLM + MCP
3. **Default agents:** PRD, Research, Feasibility, Spec, Build, Marketing (6 agents)
4. **Job orchestrator:** Parallel job execution with dependency awareness
5. **Dashboard:** Portfolio view, project detail, job monitor
6. **Budget system:** Allocation, tracking, hard limits
7. **Audit logging:** Every API call, browsable UI
8. **Export/Import:** Full project state in JSON

**Explicit Cuts:**
- Federation (v2)
- Agent Arena (v2)
- Operate/Sunset stages (v2)
- Advanced analytics (v2)
- Multiple LLM endpoint management (single provider in v1)

### v2.0 — Optimization (6-8 weeks after v1)

**Features:**
1. Agent Arena mode with A/B testing
2. Operate and Sunset stages
3. Federation protocol and adapters
4. Advanced analytics dashboards
5. Multiple LLM provider support
6. Interview Agent with live scheduling

### v3.0 — Scale (8-12 weeks after v2)

**Features:**
1. Cloud deployment options (managed hosting)
2. Marketplace for agent configs
3. Plugin system for custom MCP servers
4. Advanced portfolio optimization
5. Revenue tracking integrations (Stripe, etc.)

### Milestone Chart

```
Week 1-2:   Data model, API skeleton, basic auth
Week 3-4:   Agent runtime, MCP integration, job queue
Week 5-6:   Default agents (PRD, Research, Feasibility)
Week 7-8:   Default agents (Spec, Build, Marketing)
Week 9-10:  Dashboard UI, audit log browser
Week 11-12: Budget system, export/import, polish, launch
```

### Risks & Dependencies

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM API rate limits | High | Medium | Multi-provider failover, request queuing |
| Agent quality variance | High | High | Arena mode (v2), human checkpoints |
| MCP ecosystem immaturity | Medium | Medium | Build custom adapters, abstract MCP layer |
| Scope creep | High | High | Strict v1 cuts, weekly scope reviews |
| Single-developer bottleneck | Medium | High | Modular architecture, clear interfaces |

---

## Open Questions & Assumptions

### Open Questions

| # | Question | Impact | Decision Deadline |
|---|----------|--------|-------------------|
| 1 | What is the primary deployment model: local-first or cloud-first? | Architecture, pricing | Before Week 1 |
| 2 | Should we build our own MCP servers or rely on community? | Dev effort, feature scope | Before Week 3 |
| 3 | What LLM provider(s) to support in v1? | Cost, capability | Before Week 1 |
| 4 | How to handle secrets/credentials for user's external services? | Security architecture | Before Week 2 |
| 5 | What is the licensing model (open source, source-available, proprietary)? | Community, monetization | Before Launch |
| 6 | How to measure "quality" for agent Arena without excessive human rating burden? | Arena viability | Before v2 |

### Assumptions (Marked)

| # | Assumption | Fallback if Wrong |
|---|------------|-------------------|
| A1 | Users are comfortable with BYOK (Bring Your Own Keys) for LLM APIs | Build managed LLM pool with usage-based billing |
| A2 | Default gate mode is Recommend with 24h auto-escalation | Make fully configurable per gate |
| A3 | SQLite sufficient for single-instance MVP | Migrate to Postgres earlier |
| A4 | 6 default agents cover 80% of use cases | Prioritize additional agents based on feedback |
| A5 | Zero-copy content-addressable storage is worth the complexity | Simplify to versioned blob storage |
| A6 | Single role model sufficient | Add role hierarchy if user feedback demands |
| A7 | 90-day default audit log retention is acceptable | Make configurable from day 1 |
| A8 | Local-first is the right MVP approach | Pivot to cloud-first if distribution is hard |

---

## Risks & Mitigations

### Product Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Users don't trust agent decisions | Medium | High | Transparent evidence, easy override, recommend-not-decide default |
| 7-day MVP target unrealistic for complex projects | High | Medium | Clear scoping UX, complexity warnings, explicit de-scoping |
| Platform too complex for target users | Medium | High | Progressive disclosure, sensible defaults, wizard flows |

### Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM cost unexpectedly high | Medium | High | Budget hard limits, cost estimation before execution, model selection |
| MCP tool failures cascade | Medium | Medium | Retry logic, fallback agents, circuit breakers |
| Job queue bottleneck | Low | High | Horizontal scaling design from day 1 |
| Export/import format lock-in | Medium | Medium | Versioned schema, migration tooling |

### Legal Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM provider TOS violations | Low | High | Review TOS, document usage patterns, user responsibility disclaimer |
| User-generated content liability | Low | Medium | Clear TOS, user owns their data, no moderation |
| Open source license contamination | Low | Medium | License audit, dependency scanning |

### Adoption Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| No differentiation from existing tools | Medium | High | Focus on speed metric (7-day MVP), agent-first architecture |
| Power-user focus limits market | Medium | Medium | v2 UX improvements, progressive disclosure |
| Federation never used | Medium | Low | Core value doesn't depend on federation |

---

## Decision Log

| Decision | Options Considered | Chosen | Rationale | Owner | Date | Revisit Trigger |
|----------|-------------------|--------|-----------|-------|------|-----------------|
| Gate default mode | Auto-approve, Recommend, Require | **Recommend** | Balances speed with human oversight | Architect | 2024-12-28 | User feedback on friction |
| Primary database | Postgres, SQLite, MongoDB | **SQLite (v1) → Postgres (scale)** | Simplest start, proven migration path | Architect | 2024-12-28 | >1000 projects or multi-user |
| Agent definition | Custom DSL, JSON config, Code | **JSON config (endpoint + MCP + prompt)** | Simple, versionable, portable | Architect | 2024-12-28 | Agent complexity needs grow |
| Auth model | Role-based, Single-role + grants | **Single-role + grants** | Matches user request, simpler | Architect | 2024-12-28 | Enterprise customers demand roles |
| Audit log storage | Separate DB, Same DB, External | **Same DB (append-only table)** | Simpler ops, sufficient for v1 | Architect | 2024-12-28 | Log volume >10M rows |
| LLM abstraction | Direct API calls, LiteLLM, Custom | **LiteLLM (ASSUMPTION)** | Multi-provider, maintained, simple | Architect | 2024-12-28 | Feature gaps or performance |
| Job orchestration | Simple queue, Temporal, Custom DAG | **Simple queue + dependency graph** | Minimal complexity for v1 | Architect | 2024-12-28 | Complex workflow needs |
| Frontend framework | React, Vue, Svelte, HTMX | **React + Vite + Tailwind** | Ecosystem, hiring, component libraries | Architect | 2024-12-28 | Team preference differs |
| Deployment model | Local-only, Cloud-only, Both | **Local-first (ASSUMPTION)** | Matches indie hacker ICP, lower barrier | Architect | 2024-12-28 | Distribution friction too high |
| Export format | JSON, YAML, Custom binary | **JSON (versioned schema)** | Universal, debuggable, scriptable | Architect | 2024-12-28 | Performance issues with large exports |

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Agent** | LLM endpoint + MCP config + system prompt; executes Jobs |
| **Arena** | Mode for A/B testing agent configurations |
| **Artifact** | Output of a Job (PRD, code, report, etc.) |
| **Evidence** | Research/experiment output supporting a Decision |
| **Federation** | Protocol for sharing data between Instances |
| **Gate** | Decision checkpoint between Stages |
| **Instance** | Shardable deployment unit of the platform |
| **Job** | Unit of work assigned to an Agent |
| **MCP** | Model Context Protocol; standard for LLM tool access |
| **Stage** | Lifecycle phase (Intake, Validate, Build, Launch, Operate, Sunset) |
| **Zero-copy** | Content-addressable storage avoiding duplication |

---

## Appendix B: PRD Decomposition Example

**Input PRD:**
> "Build a SaaS tool that helps indie hackers validate ideas in 24 hours using AI-powered customer interviews."

**Decomposer Agent Output:**

```yaml
job_graph:
  - id: research_tam
    agent: research_agent
    input: "TAM analysis for idea validation SaaS, target: indie hackers"
    depends_on: []
    
  - id: research_competitors
    agent: research_agent
    input: "Competitor analysis: existing idea validation tools"
    depends_on: []
    
  - id: research_pricing
    agent: pricing_agent
    input: "WTP analysis for indie hacker tooling, comparable: Gumroad, Carrd"
    depends_on: []
    
  - id: feasibility_analysis
    agent: feasibility_agent
    input: "Technical feasibility: AI interview system, voice/text, scheduling"
    depends_on: []
    
  - id: synthesize_validation
    agent: prd_agent
    input: "Synthesize research into validation summary"
    depends_on: [research_tam, research_competitors, research_pricing, feasibility_analysis]
    
  - id: gate_decision
    agent: decision_agent
    input: "Make go/no-go recommendation based on validation summary"
    depends_on: [synthesize_validation]
```

**Parallelism:** Jobs 1-4 execute simultaneously; Job 5 waits for all; Job 6 waits for Job 5.

---

## Appendix C: Agent Config Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "name", "llm_endpoint", "system_prompt"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "name": {
      "type": "string",
      "description": "Human-readable agent name"
    },
    "llm_endpoint": {
      "type": "object",
      "properties": {
        "provider": {"type": "string", "enum": ["openai", "anthropic", "google", "local", "custom"]},
        "model": {"type": "string"},
        "base_url": {"type": "string", "format": "uri"},
        "api_key_ref": {"type": "string", "description": "Reference to stored credential"}
      },
      "required": ["provider", "model"]
    },
    "mcp_servers": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": {"type": "string"},
          "transport": {"type": "string", "enum": ["stdio", "http", "ws"]},
          "command": {"type": "string"},
          "args": {"type": "array", "items": {"type": "string"}},
          "env": {"type": "object"}
        }
      }
    },
    "system_prompt": {
      "type": "string",
      "description": "System prompt defining agent behavior"
    },
    "parameters": {
      "type": "object",
      "properties": {
        "temperature": {"type": "number", "minimum": 0, "maximum": 2},
        "max_tokens": {"type": "integer"},
        "timeout_ms": {"type": "integer"}
      }
    },
    "version": {
      "type": "integer",
      "minimum": 1
    },
    "parent_id": {
      "type": "string",
      "format": "uuid",
      "description": "Parent agent config if this is a variant"
    }
  }
}
```

---

*Document generated by Platform Architecture. Last updated: 2024-12-28.*
