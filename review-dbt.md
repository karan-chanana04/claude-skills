# /review-dbt

Review the following dbt model SQL and schema.yml. If schema.yml is missing, generate it.

## dbt Model Review

### 🔴 Critical Issues
Broken pipelines, incorrect data, or failed dbt runs.
If none: _None found._

### 🟡 Warnings
Test coverage gaps, materialisation risks, missing documentation.
If none: _None found._

### 🟢 Suggestions
Best practice improvements.
If none: _None found._

### 📋 Generated / Corrected schema.yml
Provide a complete schema.yml block using dbt v1.6+ syntax (data_tests:).

### 📋 Summary
Verdict: SAFE TO MERGE / NEEDS CHANGES / NEEDS DISCUSSION

Check for: ref() vs hardcoded tables, source() for raw tables, SELECT *,
incremental model unique_key, materialisation appropriateness,
unique + not_null tests on PK, accepted_values on status columns,
model and column descriptions, naming conventions (stg_ / int_ / fct_ / dim_).

$ARGUMENTS
