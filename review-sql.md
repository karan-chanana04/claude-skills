# /review-sql
# Place this file at: .claude/commands/review-sql.md
# Usage in Claude Code: /review-sql (then paste your SQL)

Review the following SQL for correctness, performance, and best practices.
Target platforms: Snowflake and SQL Server.

Use this exact output structure:

## SQL Review

### 🔴 Critical Issues
Issues that will cause incorrect results, data loss, or production failures.
If none: _None found._

### 🟡 Performance Warnings
Issues that will cause slow queries, full table scans, or high compute cost.
If none: _None found._

### 🟢 Best Practice Suggestions
Improvements for readability, maintainability, and long-term safety.
If none: _None found._

### 📋 Summary
Verdict: SAFE TO MERGE / NEEDS CHANGES / NEEDS DISCUSSION

Check for: cartesian joins, NULL handling errors, non-SARGable predicates, SELECT *,
hardcoded dates or env names, DELETE/UPDATE without WHERE, missing GROUP BY columns,
UNION vs UNION ALL confusion, type mismatches in JOINs, Snowflake clustering key bypass,
SQL Server implicit conversions on indexed columns.

$ARGUMENTS
