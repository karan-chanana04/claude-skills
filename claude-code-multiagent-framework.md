# Claude Code Multi-Agent Framework — Implementation Guide

> Portable step-by-step guide to set up a master orchestrator + domain subagents  
> for dbt / Python / Snowflake / SSIS workspaces.  
> Copy this file to any machine and follow in order.

---

## Prerequisites

- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)
- Claude Code authenticated (`claude login`)
- An enterprise Anthropic account with Bedrock or direct API access

---

## Part 1 — Folder Structure

Create the following structure. Projects under `workspace/` are added incrementally — only create what exists.

```
root/
├── CLAUDE.md                          ← Master orchestrator (Step 2)
├── README.md                          ← Root-level human docs (Step 3)
├── .claude/
│   ├── settings.json                  ← Claude Code settings (Step 4)
│   └── agents/
│       ├── dbt-agent.md               ← Domain agent (Step 5)
│       ├── python-agent.md            ← Domain agent (Step 5)
│       ├── snowflake-agent.md         ← Domain agent (Step 5)
│       └── ssis-agent.md             ← Domain agent (Step 5)
└── workspace/
    └── [project-name]/
        ├── CLAUDE.md                  ← Project context (Step 6)
        ├── README.md                  ← Project file inventory (Step 7)
        ├── dbt/
        │   ├── CLAUDE.md              ← Subfolder rules (Step 8)
        │   └── README.md              ← Subfolder inventory (Step 8)
        ├── python/
        │   ├── CLAUDE.md
        │   └── README.md
        ├── snowflake/
        │   ├── CLAUDE.md
        │   └── README.md
        └── ssis/
            ├── CLAUDE.md
            └── README.md
```

Run this to scaffold the base structure:

```bash
mkdir -p root/.claude/agents
mkdir -p root/workspace
```

---

## Part 2 — Root `CLAUDE.md` (Master Orchestrator)

**File:** `root/CLAUDE.md`  
**Size target:** 200–300 lines  
**Purpose:** Orchestration rules, routing logic, decision authority

```markdown
# Master Orchestrator

> Folder structure and workspace inventory: see `README.md` when needed.

## Scope

These instructions apply when working from the root directory.
Each project under `workspace/` has its own CLAUDE.md which
takes precedence for all project-level work.

---

## Operating Mode

Default: **Plan then confirm.**

Before dispatching any subagents, present:
- What you understood the task to be
- Which subagents you plan to invoke and why
- Whether execution is sequential or parallel
- Any assumptions you are making

Wait for explicit confirmation before proceeding.

**Exception:** If the request is prefixed with "go ahead and..." you may
execute without a confirmation step.

---

## Decision Authority

You decompose and delegate tasks. You do not make implementation decisions
autonomously. Pause and confirm before:

- Choosing which subagents to invoke if task scope is ambiguous
- Proceeding when subagent outputs conflict with each other
- Taking any action outside the active project folder
- Making assumptions that affect more than one domain simultaneously

---

## Workspace Layout

- All projects live under `workspace/[project-name]/`
- Each project has its own `CLAUDE.md` (context) and `README.md` (inventory)
- Each domain subfolder (dbt, python, snowflake, ssis) has its own `CLAUDE.md`
- Subagent definitions are in `.claude/agents/`

Read the active project's `CLAUDE.md` before delegating any task.

---

## Available Specialist Agents

| Domain   | Agent File          | Invoke When                                              |
|----------|---------------------|----------------------------------------------------------|
| dbt      | dbt-agent.md        | Model changes, schema tests, lineage, sources.yml        |
| Python   | python-agent.md     | Scripts, pipelines, utilities, virtual envs, pytest      |
| Snowflake| snowflake-agent.md  | DDL, stored procs, query optimisation, roles, grants     |
| SSIS     | ssis-agent.md       | SQL Server packages, ETL flows, connection managers      |

---

## Sub-Agent Routing Rules

### Parallel dispatch — run simultaneously when ALL conditions are met:
- Tasks are independent with no shared state or output dependencies
- Each task is scoped to a different domain
- No overlapping files

### Sequential dispatch — use when ANY condition is true:
- One task feeds into another (e.g., Python generates a file dbt then models)
- Shared config or connection files between domains
- Unclear scope — clarify before delegating

### Background dispatch:
- Research or analysis tasks that do not involve file modifications
- Results are not blocking current work

---

## Orchestration Protocol

1. Read `workspace/[project]/CLAUDE.md` for project context
2. Read `workspace/[project]/README.md` only if you need to locate files or
   scope impact
3. Identify which domains are involved
4. Present plan and wait for confirmation
5. Dispatch subagents (parallel or sequential per routing rules above)
6. Collect all outputs
7. Resolve any conflicts or integration issues
8. Return unified result

---

## Output Format

Always close with:

### Summary
- ✅ Completed by each subagent
- ⚠️ Issues or dependencies flagged
- 🔗 Integration notes (where subagent outputs must connect)
```

---

## Part 3 — Root `README.md`

**File:** `root/README.md`  
**Purpose:** Human-readable workspace map. Claude reads this only when navigating.

```markdown
# Workspace Root

## Structure

root/
├── .claude/agents/     → Specialist agent definitions
└── workspace/          → All active projects

## Projects

| Project | Domains | Status |
|---------|---------|--------|
| [project-name] | dbt, Python, Snowflake | Active |

## Adding a New Project

1. Create `workspace/[project-name]/`
2. Copy project CLAUDE.md template (see Implementation Guide Step 6)
3. Create domain subfolders for active domains only
4. Add entry to this table
```

---

## Part 4 — `.claude/settings.json`

**File:** `root/.claude/settings.json`  
**Purpose:** Claude Code configuration including optional subagent model routing

```json
{
  "model": "claude-opus-4-5",
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "claude-sonnet-4-5-20250929"
  }
}
```

> **Why two models?** Orchestrator runs Opus for complex reasoning and routing.
> Subagents run Sonnet for focused, well-scoped execution. This cuts token cost
> significantly without sacrificing quality on domain tasks.

To enable experimental Agent Teams (cross-agent communication):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_SUBAGENT_MODEL": "claude-sonnet-4-5-20250929"
  }
}
```

> Agent Teams require Claude Code v2.1.32 or later. Check with `claude --version`.
> Only enable if subagents need to communicate with each other directly.
> For sequential domain handoffs, the standard subagent pattern is sufficient.

---

## Part 5 — Specialist Agent Definitions

Create one file per domain in `root/.claude/agents/`.

### `dbt-agent.md`

```markdown
---
name: dbt-agent
description: Handles all dbt tasks — model creation, schema tests, sources,
             lineage tracing, and ref() dependencies. Use when task involves
             .sql models, schema.yml, sources.yml, or dbt CLI commands.
tools: Read, Write, Bash, Glob
---

You are a dbt specialist. Work only within the `dbt/` subfolder of the
active project unless explicitly told otherwise.

## Responsibilities
- Create and modify dbt models (.sql files)
- Update schema.yml (descriptions, tests, column definitions)
- Manage sources.yml and exposures
- Run dbt compile, dbt test, dbt run as needed
- Trace lineage using ref() and source()
- Flag any upstream source changes that affect downstream models

## Hard Rules
- Never modify files outside the `dbt/` subfolder
- Never drop or truncate source tables
- Always run `dbt compile` before `dbt run` on changed models

## Output Format
Return: files changed, test results, lineage warnings, any schema
conflicts that the orchestrator needs to be aware of.
```

---

### `python-agent.md`

```markdown
---
name: python-agent
description: Handles Python scripts, pipelines, and utilities. Use when task
             involves .py files, requirements.txt, virtual environments,
             data processing logic, or pytest.
tools: Read, Write, Bash, Glob
---

You are a Python specialist. Work only within the `python/` subfolder
of the active project unless explicitly told otherwise.

## Responsibilities
- Write and refactor Python scripts and pipelines
- Manage dependencies (requirements.txt / pyproject.toml)
- Implement data extraction, transformation, and loading utilities
- Write and run tests (pytest)
- Virtual environment setup and management

## Hard Rules
- Never modify files outside the `python/` subfolder
- Always include error handling on file I/O and API calls
- Never hardcode credentials — use environment variables or config files

## Output Format
Return: files changed, test results, any import or dependency issues,
and any output file paths that downstream agents (dbt, Snowflake) will need.
```

---

### `snowflake-agent.md`

```markdown
---
name: snowflake-agent
description: Handles Snowflake DDL, stored procedures, query optimisation,
             role/grant management, and data warehouse object design.
tools: Read, Write, Bash, Glob
---

You are a Snowflake specialist. Work only within the `snowflake/` subfolder
of the active project unless explicitly told otherwise.

## Responsibilities
- Write DDL (CREATE TABLE, VIEW, STREAM, TASK, PIPE)
- Stored procedures and UDFs (SQL and JavaScript)
- Query optimisation and clustering key recommendations
- Role and privilege scripts
- Stage definitions and file format objects

## Hard Rules
- Never write DROP statements without an explicit instruction and confirmation
- Always specify database and schema explicitly — never rely on session defaults
- Never include plaintext credentials in any script

## Output Format
Return: SQL files changed, object dependencies flagged, grant requirements,
and any table/view names that dbt or Python agents will reference.
```

---

### `ssis-agent.md`

```markdown
---
name: ssis-agent
description: Handles SQL Server Integration Services — ETL packages,
             connection managers, data flows, control flows, and
             SQL Server source/destination configuration.
tools: Read, Write, Glob
---

You are an SSIS specialist. Work only within the `ssis/` subfolder
of the active project unless explicitly told otherwise.

## Responsibilities
- Design and document SSIS package logic
- Connection manager configurations (SQL Server, flat file, OLE DB)
- Data flow and control flow task definitions
- Error handling and logging patterns
- Package parameter and variable management

## Hard Rules
- Never store credentials directly in package files — use SSIS package parameters
  or environment variables
- Always document connection manager dependencies
- Never modify SQL Server source tables directly — read-only access only

## Output Format
Return: package changes documented, connection dependencies listed
(with credentials masked), any schema dependencies on SQL Server or
Snowflake objects noted for the orchestrator.
```

---

## Part 6 — Project `CLAUDE.md` Template

**File:** `root/workspace/[project-name]/CLAUDE.md`  
**Size target:** 100–150 lines  
**Purpose:** Project-specific context for the orchestrator and subagents

```markdown
# Project: [Project Name]

> Full folder hierarchy and file inventory: see `README.md`
> Read README.md when locating files, scoping impact, or onboarding
> to this project for the first time in a session.

## Active Domains
- [ ] dbt        — models in `dbt/models/`
- [ ] Python     — scripts in `python/src/`
- [ ] Snowflake  — DDL in `snowflake/ddl/`
- [ ] SSIS       — packages in `ssis/packages/`

Mark only the domains that apply to this project.

---

## Data Sources

| Source       | Type       | Details                              |
|--------------|------------|--------------------------------------|
| Snowflake    | Warehouse  | DB: `PROD_DB`, schemas: `RAW`, `TRANSFORMED` |
| SQL Server   | Relational | Server: `[server]`, DB: `[db_name]`  |

---

## Naming Conventions

- dbt models: `snake_case`, staging prefix `stg_`, marts prefix `dim_` / `fct_`
- Python files: `snake_case.py`, classes `PascalCase`
- Snowflake objects: lowercase in DDL
- SSIS packages: `[domain]_[source]_[destination].dtsx`

---

## Inter-Agent Dependencies

Document data flow between domains. The orchestrator uses this to determine
sequential vs parallel dispatch.

Example:
- Python extracts raw data → lands in Snowflake `RAW` schema
- dbt models transform `RAW` → `TRANSFORMED`
- SSIS is independent — no dependency on other domains in this project

---

## Hard Rules for This Project

- Do not modify files outside this project folder
- Do not alter shared Snowflake objects used by other projects
- All changes to production objects require explicit instruction

---

## Out of Scope

- [List anything explicitly excluded from this project]
```

---

## Part 7 — Project `README.md` Template

**File:** `root/workspace/[project-name]/README.md`  
**Purpose:** File inventory and folder map. Loaded on demand, not always.

```markdown
# Project: [Project Name] — File Inventory

## Folder Structure

workspace/[project-name]/
├── dbt/
│   ├── models/
│   │   ├── staging/        → stg_ models (raw layer)
│   │   └── marts/          → dim_ and fct_ models
│   ├── tests/              → singular tests
│   ├── macros/             → reusable macros
│   ├── seeds/              → static reference data
│   ├── dbt_project.yml     → project config
│   └── schema.yml          → model documentation and tests
├── python/
│   ├── src/                → main scripts
│   ├── tests/              → pytest tests
│   └── requirements.txt    → dependencies
├── snowflake/
│   ├── ddl/                → CREATE scripts
│   ├── procs/              → stored procedures
│   └── grants/             → role and privilege scripts
└── ssis/
    ├── packages/           → .dtsx package files
    └── configs/            → connection manager configs

## Key Files

| File | Purpose |
|------|---------|
| `dbt/dbt_project.yml` | dbt project configuration |
| `python/requirements.txt` | Python dependencies |
| `snowflake/ddl/[table].sql` | Table definitions |
| `ssis/packages/[name].dtsx` | ETL package |

## Ownership

| Domain | Owner | Notes |
|--------|-------|-------|
| dbt | | |
| Python | | |
| Snowflake | | |
| SSIS | | |
```

---

## Part 8 — Subfolder `CLAUDE.md` and `README.md`

Each domain subfolder gets two files. Keep them tight.

### Subfolder `CLAUDE.md` — Rules only (~50–80 lines)

**Example: `workspace/[project]/dbt/CLAUDE.md`**

```markdown
# dbt — [Project Name]

> Folder structure and file list: see `README.md`
> Read it when locating a model, tracing lineage, or scoping impact.

## What This Module Does
Transforms raw Snowflake data from `RAW` schema into analytics-ready
models in `TRANSFORMED` schema.

## Hard Rules
- Never modify staging models without checking downstream dependencies
- All new models must have at minimum a not_null test on the primary key
- Never use hardcoded schema names — always use {{ source() }} or {{ ref() }}

## Conventions
- Staging models: `stg_[source]_[entity].sql`
- Mart models: `dim_[entity].sql` or `fct_[entity].sql`
- Every model documented in `schema.yml`

## Key Dependencies
- Upstream: Snowflake `RAW` schema (loaded by Python or SSIS)
- Downstream: BI tool reads from `TRANSFORMED` schema
```

### Subfolder `README.md` — Inventory only

```markdown
# dbt — File Inventory

## Models

| File | Layer | Description |
|------|-------|-------------|
| `models/staging/stg_orders.sql` | Staging | Raw orders from source |
| `models/marts/fct_revenue.sql` | Mart | Daily revenue fact table |

## Tests

| File | Covers |
|------|--------|
| `tests/assert_revenue_positive.sql` | fct_revenue |

## Key Config

- Target schema: `TRANSFORMED`
- Source database: `PROD_DB`
- dbt version: [version]
```

---

## Part 9 — Verification Checklist

Run through this on every new machine before starting work.

```
[ ] Claude Code installed:          claude --version
[ ] Authenticated:                  claude login
[ ] Root CLAUDE.md exists:          cat root/CLAUDE.md
[ ] settings.json exists:           cat root/.claude/settings.json
[ ] All 4 agent files exist:        ls root/.claude/agents/
[ ] At least one project set up:    ls root/workspace/
[ ] Project CLAUDE.md exists:       cat root/workspace/[project]/CLAUDE.md
[ ] Project README.md exists:       cat root/workspace/[project]/README.md
[ ] Subfolders have CLAUDE.md:      ls root/workspace/[project]/[domain]/
```

---

## Part 10 — How to Launch

### Orchestrator mode (from root)
```bash
cd root/
claude
```
Claude reads `root/CLAUDE.md` → activates as master orchestrator.
Use this when a task spans multiple domains or projects.

### Direct project mode (from project folder)
```bash
cd root/workspace/[project-name]/
claude
```
Claude reads project `CLAUDE.md` + inherits root `CLAUDE.md`.
The mode detection guard in root `CLAUDE.md` switches it to direct mode.
Use this for focused work within a single project.

### Direct domain mode (from subfolder)
```bash
cd root/workspace/[project-name]/dbt/
claude
```
Claude reads subfolder `CLAUDE.md` + project `CLAUDE.md` + root `CLAUDE.md`.
Most focused context — use for deep work in a single domain.

---

## Part 11 — Adding a New Project

1. Create the project folder:
   ```bash
   mkdir -p root/workspace/[new-project]/{dbt,python,snowflake,ssis}
   ```

2. Copy and fill in the `CLAUDE.md` template (Part 6)

3. Copy and fill in the `README.md` template (Part 7)

4. For each active domain, copy the subfolder templates (Part 8)

5. Mark only the active domains in the project `CLAUDE.md`

6. Update `root/README.md` project table

---

## Part 12 — Common Mistakes to Avoid

| Mistake | Why It's a Problem | Fix |
|---|---|---|
| Putting file inventory in `CLAUDE.md` | Loads into context on every task | Move to `README.md` |
| Putting rules in `README.md` | Claude may not read it for rule-bound tasks | Move to `CLAUDE.md` |
| Large root `CLAUDE.md` (500+ lines) | Instruction dilution — rules ignored | Split into project-level files |
| No mode detection guard | Orchestrator bleeds into direct sessions | Add guard clause to root `CLAUDE.md` |
| Missing inter-agent dependency map | Orchestrator parallelises sequential tasks | Document flow in project `CLAUDE.md` |
| Credentials in agent files | Security risk | Use env vars, reference config files |
| Enabling Agent Teams by default | High token cost, coordination overhead | Only enable when agents need to talk to each other |

---

## Quick Reference Card

```
File                              | Always loaded | Purpose
----------------------------------|---------------|---------------------------
root/CLAUDE.md                    | Yes           | Orchestration rules
root/README.md                    | On demand     | Workspace map
root/.claude/settings.json        | Yes           | Model config
root/.claude/agents/[x]-agent.md  | On demand     | Domain specialist rules
workspace/[p]/CLAUDE.md           | Yes (in proj) | Project context
workspace/[p]/README.md           | On demand     | Project file inventory
workspace/[p]/[domain]/CLAUDE.md  | Yes (in subfolder) | Domain rules
workspace/[p]/[domain]/README.md  | On demand     | Domain file inventory
```

---

*Last updated: May 2026*  
*Claude Code version tested: v2.1.32+*
