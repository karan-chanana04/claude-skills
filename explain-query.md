# /explain-query

Explain the following SQL query in plain English and flag any risks.

## Query Explanation

### 📖 What This Query Does
2–4 plain English sentences. Business purpose, no technical jargon.

### 🔍 Step-by-Step Breakdown
Walk through logical steps: source data → filters → joins → aggregations → output.
If CTEs present, explain each one's role first.
If window functions present, explain partition and frame in plain English.

### ⚠️ Risks & Gotchas
NULL edge cases, date range issues, duplicate row risk, performance concerns,
hardcoded assumptions. Flag CROSS JOIN, FULL OUTER JOIN, SELF JOIN explicitly.

### 📊 Output Shape
Grain (one row per what?), column list, row count considerations.

### 💡 Suggested Improvements
Up to 3 concrete suggestions. If none needed: _Query looks clean._

$ARGUMENTS
