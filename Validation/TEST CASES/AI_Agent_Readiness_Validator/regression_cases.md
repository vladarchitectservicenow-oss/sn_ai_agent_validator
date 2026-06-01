# Regression Test Cases — AI Agent Readiness Validator

## Purpose
Validate that existing functionality is not broken by new changes. Run on every code change.

---

## Regression Cases

### R01: Base Scan Completes Without Exception
- **Component:** AAScanner.scan()
- **Input:** Standard mock GlideRecord with all plugins active
- **Expected:** Returns object with all 5 category scores, no exceptions
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P0 — app is unusable

### R02: BYOK Table Missing — No Crash
- **Component:** BYOKValidator.validate()
- **Input:** sn_generative_ai_cfg_provider table absent
- **Expected:** Returns NOT_CONFIGURED with INFO severity
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P1 — crashes on vanilla PDIs

### R03: Cross-Scope GlideRecord Query Works
- **Component:** AAScanner.checkPlugins()
- **Input:** Mock sys_plugins with 3 active AI entries
- **Expected:** Counts all 3, returns correct plugin names
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P0 — zero results silently accepted

### R04: REST API Returns Well-Formed JSON
- **Component:** ReadinessAPI GET endpoint
- **Input:** Authenticated GET request
- **Expected:** 200 OK, valid JSON with required fields
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P1 — API consumers break

### R05: Score Range Bounded 0-100
- **Component:** ReadinessDashboard score calculation
- **Input:** Extreme edge case data (all zeros, all max)
- **Expected:** Score never < 0 or > 100
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P2 — display corruption

### R06: Historical Trend Query Works
- **Component:** GET /api/x_snaiarv/readiness/history
- **Input:** 5 mock scan records
- **Expected:** Returns all 5 sorted by date desc
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P2 — trend analysis broken

### R07: GlideAggregate Avoids getRowCount() Cap
- **Component:** AAScanner.checkDataFabric()
- **Input:** CMDB with 2500 CIs
- **Expected:** Count > 1000 (GlideAggregate bypasses cap)
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P1 — incorrect CMDB stats

### R08: Scheduled Job — Idempotent
- **Component:** ScanScheduler
- **Input:** Run twice within 30 seconds
- **Expected:** Second run returns 'already scanned recently', no duplicate record
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P2 — duplicate scans on DST

### R09: Plugin Detection — Multi-Strategy Fallback
- **Component:** AAScanner.checkPlugins()
- **Input:** Plugin not found by ID but found by name contains
- **Expected:** Still detected via fallback strategy
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P1 — plugin naming drift

### R10: Version Detection — Future Release
- **Component:** AAScanner.detectVersion()
- **Input:** Instance version = "Xanadu" (post-Australia)
- **Expected:** Recognized as >= Australia, compatibility_score = 10
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P2 — future-proofing

### R11: Permission Check — All Roles Present
- **Component:** AAScanner.checkPermissions()
- **Input:** User has admin, sn_ai.agent_admin, sn_ai.skill_author
- **Expected:** score = 15 (maximum), status = PASS
- **Last Run:** 2026-06-01 | **Status:** PASS
- **Risk if broken:** P1 — false negative for valid users

## Regression Run Frequency
- On every git push: full suite (R01-R11)
- Nightly: full suite
- Weekly: full suite + performance benchmark

## Test Data Seeding
All regression tests use deterministic mock data defined in `tests/mock_data.json`.
