# Architecture Summary — AI Agent Readiness Validator

## Product Overview
The **AI Agent Readiness Validator** is a ServiceNow Zurich → Australia scoped application that performs comprehensive pre-flight checks on PDI and production instances to determine whether the environment is ready for AI Agent Studio, NowAssist skills, and Generative AI Controller deployment.

## Release Alignment
- **Target Release:** Australia (ServiceNow Xanadu 2025)  
- **Build Framework:** Zurich (2024) — backward-compatible to Vancouver

## Core Components

| Component | Type | Location | Purpose |
|-----------|------|----------|---------|
| AAScanner | Script Include | sys_script_include | Instance-wide readiness scan engine |
| BYOKValidator | Script Include | sys_script_include | BYOK provider connectivity + compliance checks |
| ReadinessDashboard | UI Page | sys_ui_page | Visual readiness score with drill-down |
| ReadinessAPI | REST Endpoint | sys_ws_definition | GET /api/x_snaiarv/readiness — JSON readiness report |
| ScanScheduler | Scheduled Job | sysauto_script | Weekly auto-scan + trend tracking |
| ReadinessTable | Custom Table | sys_db_object (x_snaiarv_readiness_scan) | Scan result storage |
| ScoreConfig | System Property | sys_properties | Configurable scoring thresholds |

## Data Model

### Table: x_snaiarv_readiness_scan
| Field | Type | Purpose |
|-------|------|---------|
| sys_id | GUID | Primary key |
| scan_date | GlideDateTime | Timestamp of scan execution |
| total_score | Integer (0-100) | Overall readiness score |
| category_scores | JSON | Per-category breakdown |
| plugin_status | JSON | Active AI plugins + versions |
| byok_status | JSON | BYOK provider config status |
| data_fabric_score | Integer (0-100) | CMDB governance score |
| permission_gaps | JSON | Missing roles/ACLs |
| recommendations | JSON | Actionable fix steps |
| instance_version | String | Rome/Vancouver/Utah/Zurich/Australia |
| scan_duration_ms | Integer | Performance metric |

### Scoring Categories
| Category | Weight | Checks |
|----------|--------|--------|
| AI Plugins | 30% | AI Agent Studio, Generative AI Controller, NowAssist plugin activation |
| BYOK Configuration | 25% | Azure OpenAI / Bedrock / Vertex AI / watsonx connectivity |
| Data Fabric | 20% | CMDB governance score (orphans, duplicates, missing fields) |
| Permissions | 15% | Required AI roles: sn_ai.agent_admin, sn_ai.skill_author |
| Release Compatibility | 10% | Instance version ≥ Utah (minimum for AI Agent features) |

## Data Flow

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│ Scheduled   │    │  AAScanner   │    │ x_snaiarv_      │
│ Job /       │───▶│  .scan()     │───▶│ readiness_scan  │
│ REST API    │    │              │    │ (persist)       │
└─────────────┘    └──────┬───────┘    └────────┬────────┘
                          │                     │
                   ┌──────▼───────┐    ┌────────▼────────┐
                   │ BYOKValidator│    │ ReadinessDashboard│
                   │ .validate()  │    │ (UI Page)        │
                   └──────────────┘    └─────────────────┘
```

## API Contract

### GET /api/x_snaiarv/readiness
**Response (200):**
```json
{
  "instance_version": "Zurich",
  "total_score": 85,
  "categories": {
    "ai_plugins": {"score": 30, "max": 30, "status": "PASS"},
    "byok": {"score": 20, "max": 25, "status": "WARN", "issues": ["Azure OpenAI not configured"]},
    "data_fabric": {"score": 18, "max": 20, "status": "PASS"},
    "permissions": {"score": 10, "max": 15, "status": "FAIL", "issues": ["Missing: sn_ai.skill_author"]},
    "release": {"score": 7, "max": 10, "status": "PASS"}
  },
  "recommendations": [...],
  "scan_duration_ms": 342,
  "scan_date": "2026-06-01T14:22:00Z"
}
```

### POST /api/x_snaiarv/readiness/scan
Triggers a new scan. Returns 202 Accepted with job ID.

## Performance Benchmarks
| Metric | Target | Max |
|--------|--------|-----|
| Scan duration | < 500ms | 2000ms |
| GlideRecord queries | < 15 | 25 |
| Memory footprint | < 10MB | 20MB |
| API response time | < 300ms | 1000ms |

## Test Strategy
- **Unit:** Mock GlideRecord + BYOK REST calls — target > 90% coverage
- **Integration:** Test against seeded PDI with known plugin states
- **E2E:** Full scan → persist → API read → dashboard render pipeline
- **Performance:** Load test with 10 concurrent scans

## Deployment Pre-requisites
- ServiceNow Utah or later
- sn_ai plugin (com.glide.automation.ai_agent) — auto-detected, not required
- REST API access role: admin or sn_ai.agent_admin
- App scope: x_snaiarv
