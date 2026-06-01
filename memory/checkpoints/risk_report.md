# Risk Report — AI Agent Readiness Validator

## Risk Assessment Matrix

| ID | Risk | Severity | Likelihood | Impact | Status | Mitigation |
|----|------|----------|-----------|--------|--------|------------|
| P0-01 | BYOK provider table missing on vanilla PDI | Low | High | Medium | MITIGATED | Return NOT_CONFIGURED with INFO severity, not FAIL |
| P0-02 | Cross-scope GlideRecord returns empty without error | Medium | Medium | High | MITIGATED | Pre-scan ACL check + explicit access grant docs |
| P0-03 | Token/credential leak via REST response | Critical | Low | Critical | MITIGATED | Never serialize provider credentials; mask all secrets |
| P1-01 | sn_generative_ai_cfg_provider missing on pre-Australia instances | Medium | High | Medium | MITIGATED | Graceful table-not-found → INFO status, proceed |
| P1-02 | REST endpoint auth bypass (public readiness report) | Medium | Low | High | MITIGATED | Require authenticated user with admin or sn_ai scope |
| P1-03 | GlideRecord.getRowCount() cap at 1000 for large CMDBs | High | Medium | Low | MITIGATED | Use GlideAggregate for counts > 1000 |
| P1-04 | Scheduled job timeout on slow instances | Medium | Low | High | MITIGATED | 30s gs.sleep() cap; batch scan across multiple jobs |
| P2-01 | Instance version detection failure on new releases | Low | Medium | Low | MITIGATED | Regex fallback: match "Vancouver|Utah|Zurich|Australia" prefix |
| P2-02 | Plugin detection via sys_plugins unreliable (naming drift) | Low | Medium | Medium | MITIGATED | Multiple fallback queries: plugin ID, name contains, table exists |
| P2-03 | JSON category_scores field grows unbounded over time | Low | Medium | Low | MITIGATED | Max 100 scan records; LRU eviction; optional auto-cleanup |
| P2-04 | ReadinessDashboard UI fails on NextExperience iframes | Low | Medium | Low | OPEN | Fall back to raw JSON API for dashboard data; UI is bonus |
| P2-05 | Score threshold system properties not created at install | Low | High | Medium | MITIGATED | Default thresholds in code; properties override if present |
| P3-01 | Scheduled job double-fire on Daylight Savings | Low | Low | Medium | OPEN | Use GlideDateTime diff check; skip if < 20h since last scan |
| P3-02 | Concurrent scan API calls race on table writes | Low | Low | Medium | MITIGATED | GlideRecord lock via sys_mod_count check before insert |
| P3-03 | Plugin name localization (non-English instances) | Low | Low | Low | OPEN | Match plugin ID only; name is display-only hint |
| P3-04 | CMDB class hierarchy depth changes break field detection | Low | Low | Low | OPEN | Test against top-10 CMDB classes only |
| P3-05 | README License header vs LICENSE file contradiction | Medium | Low | High | MITIGATED | Pre-commit validation hook verifies both agree |

## Severity Legend
- **P0 (Critical):** Blocker — app cannot ship with this unresolved
- **P1 (High):** Must fix before production deployment
- **P2 (Medium):** Fix in next release cycle
- **P3 (Low):** Known edge case, documented, accepted

## Risk Register Updates
This report is versioned with the product. Update on:
- Platform release changes (e.g., Australia → Xanadu)
- New plugin/API dependencies added
- Customer-reported field issues
- Scheduled monthly risk review
