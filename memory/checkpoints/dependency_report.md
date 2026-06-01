# Dependency Report — AI Agent Readiness Validator

## Scope
**Application Scope:** x_snaiarv  
**Base Instance:** Zurich (ServiceNow 2024)  
**Minimum Platform Version:** Utah

## Plugin Dependencies

| Plugin ID | Plugin Name | Required | Fallback Behavior |
|-----------|------------|----------|-------------------|
| com.glide.automation.ai_agent | AI Agent Studio | No | Skip AI Agent checks, score 0/30 |
| com.glide.automation.genai_ctrl | Generative AI Controller | No | Skip GenAI checks, note in report |
| com.glide.ngbsm.control_tower | Control Tower | No | Data Fabric score from CMDB only |

## Table Dependencies

| Table | Scope | Read/Write | Purpose |
|-------|-------|-----------|---------|
| sys_plugins | Global | Read | Check active AI plugins |
| sn_generative_ai_cfg_provider | Global | Read | BYOK provider config |
| cmdb_ci | Global | Read | CMDB governance scan |
| sys_user_role | Global | Read | Verify AI roles |
| sys_user_has_role | Global | Read | User-to-role mappings |
| sys_properties | Global | Read/Write | System property thresholds |
| x_snaiarv_readiness_scan | x_snaiarv | Read/Write | Scan results storage |
| x_snaiarv_scan_category | x_snaiarv | Read/Write | Per-category detail |

## Role Dependencies

| Role | Required | Purpose |
|------|----------|---------|
| admin | Yes | Full read access across scopes |
| sn_ai.agent_admin | No | AI Agent admin functions (detected) |
| sn_ai.skill_author | No | NowAssist skill authoring (detected) |
| snc_platform_role | Yes | Platform API access |

## Cross-Scope Access Grants

| Source Scope | Target Scope | Table | Access |
|-------------|-------------|-------|--------|
| x_snaiarv | Global | sys_plugins | Read |
| x_snaiarv | Global | sn_generative_ai_cfg_provider | Read |
| x_snaiarv | Global | cmdb_ci | Read |
| x_snaiarv | Global | sys_user_role | Read |
| x_snaiarv | Global | sys_user_has_role | Read |
| x_snaiarv | Global | sys_properties | Read |

## REST API Dependencies

| Endpoint | Method | Purpose | Auth |
|----------|--------|---------|------|
| /api/x_snaiarv/readiness | GET | Readiness report | Basic/RBAC |
| /api/x_snaiarv/readiness/scan | POST | Trigger scan | Basic/RBAC |
| /api/x_snaiarv/readiness/history | GET | Historical trends | Basic/RBAC |

## External Dependencies
**None.** This is a fully self-contained scoped app. No external system calls. BYOK provider validation is done by checking local configuration completeness, not by probing external endpoints.

## Installation Order
1. App scope registration (x_snaiarv)
2. Tables: x_snaiarv_readiness_scan (parent), x_snaiarv_scan_category (child)
3. Script Includes: AAScanner, BYOKValidator
4. REST Endpoints: readiness, readiness/scan, readiness/history
5. UI Page: ReadinessDashboard
6. Scheduled Job: Weekly scan
7. System Properties: score thresholds
8. ACLs: cross-scope read grants
9. Roles: admin, sn_ai.agent_admin (if exists)
