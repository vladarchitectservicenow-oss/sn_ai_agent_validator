# Test Suite SOP — AI Agent Readiness Validator

## Test Environment
- **Runtime:** Node.js moc
k (GlideRecord, GlideDateTime, GlideAggregate, gs)  
- **Coverage target:** > 90% per Script Include
- **Test file:** `tests/test_aascanner.js`, `tests/test_byokvalidator.js`, `tests/test_aascanner_e2e.js`

---

## Test Scenarios

### T01: Full Scan — All Categories Pass
**Preconditions:** All plugins active, BYOK configured, CMDB clean, roles granted  
**Input:** AAScanner.scan()  
**Expected Output:** total_score = 100, all categories PASS, no recommendations  
**Assertions:**
- `result.total_score === 100`
- `result.categories.ai_plugins.status === 'PASS'`
- `result.recommendations.length === 0`

### T02: BYOK Table Missing — Graceful Degradation
**Preconditions:** sn_generative_ai_cfg_provider table does not exist  
**Input:** BYOKValidator.validate()  
**Expected Output:** status = 'NOT_CONFIGURED', severity = 'INFO', score = 0  
**Assertions:**
- `result.status === 'NOT_CONFIGURED'`
- `result.severity === 'INFO'`
- `result.score === 0`
- No exception thrown, no crash

### T03: Missing AI Plugins — Score 0
**Preconditions:** No AI-related plugins active  
**Input:** AAScanner.checkPlugins()  
**Expected Output:** score = 0, status = 'FAIL', recommendations list includes install steps  
**Assertions:**
- `result.score === 0`
- `result.status === 'FAIL'`
- `result.recommendations.length >= 1`

### T04: CMDB Governance Scan — Orphans Detected
**Preconditions:** cmdb_ci has orphan records (no parent, no relationships)  
**Input:** AAScanner.checkDataFabric()  
**Expected Output:** score < 20 (out of 20), orphan_count > 0, status = 'WARN'  
**Assertions:**
- `result.score < 20`
- `result.details.orphan_count > 0`
- `result.status === 'WARN'`

### T05: Permission Gap — Missing sn_ai.skill_author
**Preconditions:** User has admin but not sn_ai.skill_author  
**Input:** AAScanner.checkPermissions()  
**Expected Output:** score < 15, missing_roles includes 'sn_ai.skill_author'  
**Assertions:**
- `result.score < 15`
- `result.details.missing_roles.includes('sn_ai.skill_author')`

### T06: REST API — GET Readiness (Cached)
**Preconditions:** Previous scan exists in x_snaiarv_readiness_scan  
**Input:** GET /api/x_snaiarv/readiness  
**Expected Output:** 200 OK, returns latest scan, scan_duration_ms present  
**Assertions:**
- `response.status === 200`
- `json.total_score >= 0 && json.total_score <= 100`
- `json.categories` has all 5 categories
- `json.scan_duration_ms > 0`

### T07: REST API — POST Trigger Scan (Async)
**Preconditions:** Authenticated request, no lock conflict  
**Input:** POST /api/x_snaiarv/readiness/scan  
**Expected Output:** 202 Accepted, returns job_id, status = 'queued'  
**Assertions:**
- `response.status === 202`
- `json.job_id` is non-empty string
- `json.status === 'queued'`

### T08: Concurrent Scan — Lock Conflict
**Preconditions:** Scan already in progress (last scan < 5 seconds ago)  
**Input:** POST /api/x_snaiarv/readiness/scan  
**Expected Output:** 409 Conflict, message = 'Scan already in progress'  
**Assertions:**
- `response.status === 409`
- `json.error === 'Scan already in progress'`

### T09: Instance Version Detection — Australia
**Preconditions:** Instance version string = "Australia"  
**Input:** AAScanner.detectVersion()  
**Expected Output:** version = 'Australia', compatibility_score = 10  
**Assertions:**
- `result.version === 'Australia'`
- `result.compatibility_score === 10`

### T10: Instance Version — Vancouver (Below Minimum)
**Preconditions:** Instance version string = "Vancouver"  
**Input:** AAScanner.detectVersion()  
**Expected Output:** version = 'Vancouver', compatibility_score = 0, status = 'FAIL'  
**Assertions:**
- `result.compatibility_score === 0`
- `result.status === 'FAIL'`

### T11: REST API — Unauthenticated Request
**Preconditions:** No Basic Auth header  
**Input:** GET /api/x_snaiarv/readiness  
**Expected Output:** 401 Unauthorized  
**Assertions:**
- `response.status === 401`

### T12: Scheduled Job — Weekly Scan Persists
**Preconditions:** Clean instance, no prior scans  
**Input:** Scheduled job executes scan  
**Expected Output:** 1 record in x_snaiarv_readiness_scan, scan_date within last minute  
**Assertions:**
- Table row count = 1
- scan_date is valid GlideDateTime
- total_score > 0

### T13: Negative Case — Empty CMDB
**Preconditions:** cmdb_ci has zero records  
**Input:** AAScanner.checkDataFabric()  
**Expected Output:** score = 20 (no orphans, no duplicates — perfect), status = 'PASS'  
**Assertions:**
- `result.score === 20`
- `result.status === 'PASS'`
- `result.details.orphan_count === 0`

### T14: Negative Case — Corrupt Readiness Table
**Preconditions:** x_snaiarv_readiness_scan contains row with NULL total_score  
**Input:** GET /api/x_snaiarv/readiness  
**Expected Output:** 200 OK, total_score defaults to 0 (not null, not crash)  
**Assertions:**
- `response.status === 200`
- `json.total_score === 0`

## Test Execution Order
1. Unit: T01 → T05 → T09 → T10 → T13 (AAScanner isolated)
2. Unit: T02 → T03 (BYOKValidator + plugin check)
3. Integration: T12 (Scheduled Job → Table persistence)
4. API: T06 → T07 → T08 → T11 → T14
5. E2E: Full pipeline — scan → persist → API read

## Failure Protocol
- Any RED test: fix source, re-run full suite
- 3 consecutive RED on same test: escalate to fallback model
- All GREEN: proceed to git commit + push
