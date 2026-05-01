# /review-ssis

Review the following SSIS package XML (.dtsx content) for production readiness.

## SSIS Package Review

### 🔴 Critical Issues
Package failure, data loss, or security exposure.
If none: _None found._

### 🟡 Warnings
Operational risk or deployment problems.
If none: _None found._

### 🟢 Suggestions
Best practice improvements.
If none: _None found._

### 📋 Summary
Verdict: SAFE TO DEPLOY / NEEDS CHANGES / NEEDS DISCUSSION

Check for: hardcoded server/db names, hardcoded credentials in ConnectionString or Password fields,
EncryptSensitiveWithUserKey protection level (bad for CI/CD),
unconnected error outputs on Data Flow tasks, missing OnError event handlers,
Sort transformation in data flow (move to SQL), Lookup full-cache on large tables,
package not using Project Deployment Model, 32/64-bit driver mismatch.

$ARGUMENTS
