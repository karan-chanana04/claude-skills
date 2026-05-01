# /gen-dbt-tests

Generate a complete, production-ready dbt schema.yml with tests and descriptions
for the following dbt model SQL or column list.

Rules:
- Use dbt v1.6+ syntax: data_tests: (not tests:)
- Infer primary key from _id, _key, _sk, _pk suffixes or column named 'id'
- Apply unique + not_null to primary key
- Apply not_null to columns with _id, _date, _at, amount, status in name
- Apply accepted_values to status/type/flag/is_*/has_* columns (infer values)
- Apply relationships test where column matches <other_model>_id pattern
- Mark columns with email/name/phone/ssn/dob as pii: true in meta
- Add meta: {owner: "data-engineering"} to model

After the YAML, output a Test Coverage Summary table and a Missing Information section.

$ARGUMENTS
