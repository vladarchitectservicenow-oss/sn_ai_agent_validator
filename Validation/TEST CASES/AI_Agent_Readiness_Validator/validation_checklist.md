# Validation Checklist — AI Agent Readiness Validator

## Pre-Commit Validation (Run Before Every Git Push)

### Code Quality
- [ ] All JavaScript files pass `eslint` (no console.log, no var)
- [ ] All file headers have `Copyright (c) 2026 Vladimir Kapustin. SPDX-License-Identifier: AGPL-3.0-only`
- [ ] No hardcoded credentials in any file
- [ ] No `eval()` in production code (Script Includes run in server-side runtime)
- [ ] All method names use camelCase
- [ ] No unresolved TODO markers in src/

### Tests (G1 Gate)
- [ ] `node tests/test_aascanner.js` → ALL PASS
- [ ] `node tests/test_byokvalidator.js` → ALL PASS
- [ ] `node tests/test_aascanner_e2e.js` → ALL PASS
- [ ] Coverage report shows > 90% per Script Include
- [ ] Test execution log saved to tests/execution_history/
- [ ] No tests skipped without explicit SKIP: <reason> comment

### Documentation (G2 Gate)
- [ ] README.md ≥ 2000 words
- [ ] README contains Mermaid architecture diagram
- [ ] README contains ROI analysis section
- [ ] README contains Troubleshooting section
- [ ] README License header matches LICENSE file (both AGPL-3.0)
- [ ] No duplicate sections in README (grep '^## ' README.md | sort | uniq -d = empty)

### Licensing (G3 Gate)
- [ ] LICENSE file is full AGPL-3.0 text (624+ lines)
- [ ] `head -1 LICENSE` = `GNU AFFERO GENERAL PUBLIC LICENSE`
- [ ] All src/ files have SPDX-License-Identifier
- [ ] Copyright owner = "Vladimir Kapustin" (not abbreviated)

### Git (G4 Gate)
- [ ] `.gitignore` exists and excludes `__pycache__/`, `*.pyc`, `reports/`, `venv/`
- [ ] `git diff --cached --stat` shows all expected files staged
- [ ] Commit message is conventional format: `feat: [product] Phase 1-2 docs + validation suite`
- [ ] Remote push verified via GitHub API (branch exists, HEAD matches local)

### Security (G5 Gate)
- [ ] No credentials in source code (grep for password|token|secret|api_key)
- [ ] REST endpoints require authentication
- [ ] No sensitive data in README examples

### Integrity (G6 Gate)
- [ ] Phase 1 docs: architecture_summary.md ≥ 40 lines
- [ ] Phase 1 docs: dependency_report.md ≥ 30 lines
- [ ] Phase 1 docs: risk_report.md ≥ 5 P0/P1/P2/P3 entries
- [ ] Phase 1 docs: execution_plan.md ≥ 30 lines
- [ ] Phase 2 docs: test_suite_SOP.md ≥ 10 scenarios (T01-TXX format)
- [ ] Phase 2 docs: regression_cases.md ≥ 8 cases (R01-RXX format)
- [ ] Phase 2 docs: edge_cases.md ≥ 8 cases
- [ ] Phase 2 docs: validation_checklist.md (this file)

## Post-Push Verification
- [ ] DONE.marker exists in repo root
- [ ] GitHub API confirms LICENSE SPDX = 'AGPL-3.0-only'
- [ ] GitHub API confirms README word count ≥ 2000
- [ ] All Phase 1+2 files accessible via raw.githubusercontent.com

## Sign-off
- **Date:** 2026-06-01
- **Author:** Vladimir Kapustin
- **Status:** PHASE 2 COMPLETE
