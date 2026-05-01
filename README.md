# Claude DE Skills & Commands Library
### Backend Data Engineering — Snowflake · SQL Server · dbt · Airflow · SSIS

A reusable library of Claude skills (system prompts) and slash commands for backend data engineering PR reviews, code analysis, and documentation generation.

---

## What's in This Library

| File | Type | Trigger | Purpose |
|------|------|---------|---------|
| `sql_review_skill.md` | Skill | `/review-sql` | Full SQL review for correctness, performance, best practices |
| `airflow_dag_review_skill.md` | Skill | `/review-dag` | Airflow DAG review for idempotency, retries, SLA, security |
| `dbt_review_skill.md` | Skill | `/review-dbt` | dbt model + schema.yml review, generates missing schema.yml |
| `ssis_review_skill.md` | Skill | `/review-ssis` | SSIS .dtsx XML review for hardcoded values, missing error handling |
| `dbt_test_generator_skill.md` | Skill | `/gen-dbt-tests` | Auto-generates complete dbt schema.yml with tests + docs |
| `sql_explainer_skill.md` | Skill | `/explain-query` | Explains SQL in plain English + flags risks |

---

## How to Use

### Option A — Claude.ai (Web / Desktop)

1. Open a new Claude conversation
2. Copy the **System Prompt** section from any skill `.md` file
3. Paste it as your first message or save it as a Project with custom instructions
4. From then on, just paste your code and say the trigger word

**Even simpler:** Save each skill as a **Claude Project**:
- Go to claude.ai → Projects → New Project
- Paste the skill's system prompt as "Project Instructions"
- Name the project e.g. "SQL Reviewer" or "dbt Agent"
- Every conversation in that project automatically uses the skill

### Option B — Claude Code (Terminal) — Slash Commands

1. In your data engineering repo, create the `.claude/commands/` directory:
```bash
mkdir -p .claude/commands
```

2. Create one `.md` file per command (copy from `claude-commands/` folder):
```bash
# Example for /review-sql
cp claude-commands/review-sql.md /your-repo/.claude/commands/review-sql.md
```

3. Open Claude Code in your repo:
```bash
claude
```

4. Use the commands:
```
/review-sql

SELECT * FROM orders o
JOIN customers c ON o.cust_id = c.id
WHERE status = 'active'
```

```
/review-dag

[paste your dag.py content here]
```

```
/gen-dbt-tests

[paste your dbt model SQL here]
```

### Option C — GitHub Actions (Automated PR Review)

The commands work as prompts inside the `de_review.py` script (in the companion `de-agent` repo).
Each changed file type routes to the matching skill's checklist automatically.

---

## Review Severity Guide

| Icon | Level | Meaning |
|------|-------|---------|
| 🔴 | Critical | Will break prod, cause data loss, or expose security issue. Block merge. |
| 🟡 | Warning | Performance risk, operational concern, or correctness edge case. Discuss before merge. |
| 🟢 | Suggestion | Best practice improvement. Nice to fix, not blocking. |

---

## Skill Coverage Detail

### `/review-sql` checks
- Cartesian joins (missing JOIN condition)
- NULL handling (`= NULL` vs `IS NULL`, NULLs in aggregations)
- Date range edge cases (BETWEEN inclusivity)
- Non-SARGable predicates (functions on WHERE columns)
- Snowflake: clustering key bypass, SELECT * on wide tables
- SQL Server: implicit type conversions on indexed columns, cursor usage
- Missing GROUP BY columns, UNION vs UNION ALL confusion
- Hardcoded dates, env names, DELETE/UPDATE without WHERE

### `/review-dag` checks
- `catchup=False` missing (causes backfill on deploy)
- `retries` and `retry_delay` not set
- Idempotency (append-only writes without dedup)
- Hardcoded connection strings / passwords
- Missing `on_failure_callback` or `email_on_failure`
- `ExternalTaskSensor` in poke mode (use reschedule)
- Pool assignment, SLA, parallelism settings
- SnowflakeOperator / MsSqlOperator config

### `/review-dbt` checks
- `ref()` vs hardcoded `schema.table`
- `source()` for raw layer tables
- Incremental model `unique_key` and `is_incremental()` usage
- Materialisation appropriateness for data volume
- `unique` + `not_null` tests on primary keys
- `accepted_values` on status/type/flag columns
- Model and column `description` presence
- Naming conventions: `stg_` / `int_` / `fct_` / `dim_`

### `/review-ssis` checks
- Hardcoded server names, credentials in ConnectionString
- Package protection level (`EncryptSensitiveWithUserKey` bad for CI/CD)
- Unconnected error outputs on Data Flow tasks
- Missing `OnError` event handlers
- Sort transformation in Data Flow (should be in SQL source)
- Lookup cache mode appropriateness
- Project Deployment Model usage
- 32/64-bit driver compatibility

### `/gen-dbt-tests` generates
- `unique` + `not_null` on inferred primary key
- `not_null` on all `_id`, `_date`, `_at`, `amount`, `status` columns
- `accepted_values` on status/type/flag/is_*/has_* columns
- `relationships` tests where FK pattern detected
- PII tagging (`meta: {pii: true}`) on name/email/phone columns
- Full `description` for model and every column

### `/explain-query` outputs
- Plain English business purpose (2–4 sentences)
- Logical step-by-step breakdown (not line-by-line)
- CTE-by-CTE explanation if present
- Window function frame explanation in plain English
- Risks and edge cases flagged
- Output grain and shape description
- Up to 3 improvement suggestions

---

## Adding Your Own Skills

Copy any skill file, modify the checklist section, and save with a new name.
The structure is always:
1. **Trigger** (what command invokes it)
2. **Output format** (the Markdown structure Claude will follow)
3. **Internal checklist** (the domain-specific rules Claude checks against)

---

## Repo Structure

```
skills/
├── README.md                          ← This file
├── sql_review_skill.md
├── airflow_dag_review_skill.md
├── dbt_review_skill.md
├── ssis_review_skill.md
├── dbt_test_generator_skill.md
├── sql_explainer_skill.md
└── claude-commands/
    ├── review-sql.md                  ← Copy to .claude/commands/ in your repo
    ├── review-dag.md
    ├── review-dbt.md
    ├── review-ssis.md
    ├── gen-dbt-tests.md
    └── explain-query.md
```

---

*Part of the Claude DE Agent project | https://github.com/karan-chanana04/Code-review-mcp*
