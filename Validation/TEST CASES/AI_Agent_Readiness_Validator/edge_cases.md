# Edge Cases — AI Agent Readiness Validator

## Documented Edge Cases

---

### E01: Empty Instance — No Data
**Scenario:** Completely fresh PDI with zero customizations, no plugins activated, empty CMDB  
**Expected Behavior:** Scanner completes successfully. All scores = 0 except CMDB (20/20 — no orphans possible with zero records). Total score = 20. Recommendations = max verbosity (install plugins, configure BYOK, etc.)  
**Status:** VERIFIED (mock)  
**Risk:** Low — expected behavior on fresh instances

### E02: Corrupted JSON in category_scores
**Scenario:** x_snaiarv_readiness_scan contains malformed JSON in category_scores field  
**Expected Behavior:** API returns 200 with category_scores = {} and a warning in recommendations: "Corrupt scan data detected"  
**Status:** DOCUMENTED  
**Risk:** Low — requires manual DB corruption

### E03: Partial Plugin Activation
**Scenario:** AI Agent Studio plugin is installed but status = 'inactive'  
**Expected Behavior:** Scanner reports score = 0 for ai_plugins (plugin exists but not active). Recommendation: "Activate AI Agent Studio plugin (com.glide.automation.ai_agent)"  
**Status:** DOCUMENTED  
**Risk:** Medium — common on PDIs during upgrade

### E04: Concurrent Read During Scan Write
**Scenario:** GET /api/x_snaiarv/readiness is called while a scan is writing to the table  
**Expected Behavior:** Returns the previous completed scan (cache hit). Does NOT return partial in-progress scan.  
**Status:** DOCUMENTED  
**Risk:** Medium — race condition on busy instances

### E05: Unicode Characters in Instance Name
**Scenario:** Instance name contains non-ASCII characters (e.g., "東京-PROD")  
**Expected Behavior:** String stored correctly in instance_version field. No JSON encoding errors.  
**Status:** DOCUMENTED  
**Risk:** Low — standard GlideRecord handles Unicode

### E06: Very Large CMDB (>100,000 CIs)
**Scenario:** Production instance with 150,000+ CIs  
**Expected Behavior:** GlideAggregate completes in < 2 seconds. Full scan < 5 seconds total. Results accurate (compared to GlideRecord manual count).  
**Status:** DOCUMENTED — performance target  
**Risk:** Medium — needs real production testing

### E07: BYOK Provider with Partial Credentials
**Scenario:** Azure OpenAI configured but API key field is empty  
**Expected Behavior:** Scanner reports status = 'WARN', details = "Azure OpenAI: missing API key"  
**Status:** DOCUMENTED  
**Risk:** Medium — common configuration gap

### E08: sys_plugins Table — Unexpected Column Names
**Scenario:** ServiceNow changes sys_plugins column names between releases (id → plugin_id)  
**Expected Behavior:** Scanner uses getValue('plugin_id') then falls back to getValue('id') if null. Both paths work.  
**Status:** DOCUMENTED  
**Risk:** Medium — platform drift

### E09: System Property Thresholds — Invalid Values
**Scenario:** x_snaiarv.min_ai_plugins is set to "foo" (non-integer)  
**Expected Behavior:** Scanner defaults to built-in threshold. Logs gs.warn("Invalid system property: x_snaiarv.min_ai_plugins = 'foo'")  
**Status:** DOCUMENTED  
**Risk:** Low — requires manual property corruption

### E10: REST API — Large Response Payload
**Scenario:** Historical trends endpoint returns 1000+ scan records in JSON  
**Expected Behavior:** Response size < 1MB. If > 100 records, paginate automatically (next_cursor in response).  
**Status:** DOCUMENTED  
**Risk:** Low — 1000 scans = ~20 years of weekly scans

### E11: Table sys_mod_count Overflow
**Scenario:** Scan record updated 2,147,483,647 times (Int32 max)  
**Expected Behavior:** Not realistically reachable. Scanner uses date-based lock instead.  
**Status:** DOCUMENTED  
**Risk:** Zero — theoretical only

## Edge Case Test Priority
- High: E03 (partial plugin), E07 (partial BYOK) — common in production
- Medium: E04 (concurrent), E06 (large CMDB), E08 (column drift)
- Low: E01, E05, E09, E10, E11 — rare or theoretical
