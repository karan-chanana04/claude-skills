# /review-dag

Review the following Airflow DAG for production readiness.

## Airflow DAG Review

### 🔴 Critical Issues
Pipeline failures, data duplication, or incorrect backfill behaviour.
If none: _None found._

### 🟡 Warnings
Operational risk, SLA misses, or silent failures.
If none: _None found._

### 🟢 Suggestions
Best practice improvements for observability and resilience.
If none: _None found._

### 📋 Summary
Verdict: SAFE TO MERGE / NEEDS CHANGES / NEEDS DISCUSSION

Check for: catchup=False missing, retries not set, retry_delay not a timedelta,
idempotency issues (append without dedup), hardcoded connection strings or passwords,
missing on_failure_callback, ExternalTaskSensor in poke mode,
pool assignment on heavy tasks, SLA not set on critical tasks,
SnowflakeOperator / MsSqlOperator config completeness.

$ARGUMENTS
