# 📋 Proposals — Neuraj Organization Blueprint v1.0 Review
**Reviewer:** Claude Sonnet 4.5  
**Date:** October 25, 2025  
**Review Protocol:** MyriadEye Round #1

---

## Section 5.1: API Contract

### Observation
The API contract defines request/response schemas for success cases but lacks error response structure. This creates implementation ambiguity for error handling in both Neuraj AI and NeurOS.

### Recommendation
Add explicit error response schema with standard fields, HTTP status code mappings, and retry policies.

### Example Rewrite
```diff
     response_schema:
       task_id: "string"
       status: "enum: [queued, running, needs_approval, blocked, done, error]"
       artifacts: "array of {type, path_or_ref, sha256}"
       metrics_ref: "map of labels (e.g., prom_labels.task_id)"
       log_ref: "file/line reference (e.g., governance_log.json#L120)"
       message: "human summary string"
+    error_response_schema:
+      error_code: "string (e.g., INVALID_MODULE, GUARDRAIL_VIOLATION, RESOURCE_EXHAUSTED)"
+      message: "human-readable error description"
+      details: "object (context-specific error data)"
+      trace_id: "string (ULID for correlation)"
+      retry_after: "integer (seconds) | null"
+      http_status: "integer (400 bad request, 403 forbidden, 429 rate limited, 500 internal error)"
```

---

## Section 5.2: Data Contracts

### Observation
Table relationships are implied but foreign key constraints and cascade behaviors are not documented. This affects data integrity guarantees.

### Recommendation
Explicitly define all FK relationships with cascade rules.

### Example Rewrite
```diff
   data_contracts:
     postgres_tables:
       modules: ["id", "name", "version", "status", "owner", "created_at"]
       tasks: ["id", "module_id", "action", "params_json", "created_at", "started_at", "finished_at", "status", "requester", "approver"]
       artifacts: ["id", "task_id", "type", "path_or_ref", "sha256", "created_at"]
       governance_events: ["id", "actor", "kind", "message", "task_id", "created_at", "sha256"]
       kpis: ["id", "domain", "name", "value", "unit", "as_of", "source_sha"]
+    foreign_keys:
+      tasks.module_id -> modules.id (ON DELETE CASCADE)
+      artifacts.task_id -> tasks.id (ON DELETE CASCADE)
+      governance_events.task_id -> tasks.id (ON DELETE RESTRICT)
+    indexes:
+      tasks: ["status", "created_at DESC", "(module_id, status)"]
+      governance_events: ["actor", "created_at DESC", "task_id"]
```

---

## Section 6: Security & RBAC

### Observation (ORG-FND-003)
Governance logs are described as "immutable" but the technical mechanism ensuring immutability is not specified. This is a MUST-fix for audit integrity.

### Recommendation
Specify the immutability mechanism using append-only storage or cryptographic hash chains.

### Example Rewrite
```diff
   logging_audit:
     trace_id: "ULID/Snowflake per task"
-    immutable_logs: ["governance_log.json (append-only)", "daily_brief_YYYY-MM-DD.md"]
+    immutable_logs:
+      governance_log.json:
+        mechanism: "append-only file system flag (chattr +a on Linux)"
+        integrity: "SHA-256 hash chain: each entry includes hash of previous entry"
+        verification: "merkle tree root published to audit_index.json daily"
+      daily_brief:
+        mechanism: "write-once, signed with PGP key"
+        retention: "immutable for 12 months, then archive"
     clock_sync: "NTP required"
     evidence_pack: ["release_notes", "test_reports", "grafana_screens", "contract_versions", "sha_index"]
```

### Observation (ORG-FND-004)
HMAC and mTLS authentication are defined but session lifetime and token rotation policies are missing.

### Recommendation
Add explicit session and credential rotation policies.

### Example Rewrite
```diff
   authn_authz:
     human_app: ["DEV: local", "LATER: OAuth2"]
     service_to_service:
       stage: "HMAC-SHA256 (timestamp window ±120s)"
       prod_lite: "mTLS + HMAC"
+      token_policy:
+        hmac_ttl: "1 hour (tokens expire and must be refreshed)"
+        mtls_cert_rotation: "90 days (automated via cert-manager)"
+        session_timeout: "24 hours idle, 7 days absolute"
+        key_rotation: "quarterly for HMAC secrets, immediate on incident"
```

---

## Section 7: Data & Knowledge Architecture

### Observation
PostgreSQL retention is well-defined but vector store backup and recovery procedures are missing.

### Recommendation
Define vector store operational procedures to match PostgreSQL rigor.

### Example Rewrite
```diff
   **Vector Store:** FAISS / Chroma for Neuraj AI knowledge recall  
   Metadata → `{source_path, sha, section, timestamp}`  
+  
+  **Vector Store Operations:**
+  - Snapshot frequency: Daily incremental, weekly full
+  - Backup retention: 7 daily, 4 weekly, 12 monthly
+  - Recovery SLA: <1 hour for latest snapshot
+  - Failover: Read-replica in different availability zone (STAGE+)
+  - Consistency check: Weekly SHA verification against source documents
```

---

## Section 8: Observability & Ops

### Observation (ORG-FND-006)
Metrics and dashboards are comprehensive but alert routing and escalation are undefined.

### Recommendation
Add alert policy with notification channels and escalation timers.

### Example Rewrite
```diff
   **Incident Severity & Response**
   | Level | Action |
   |-------|---------|
   | P1 | Kill-switch, rotate keys, rollback image, alert, RCA < 24 h |
   | P2 | Throttle + canary rollback |
   | P3 | Log + schedule next window |
+
+  **Alert Policies & Escalation**
+  | Alert Type | Initial Notify | Escalate After | Channel |
+  |------------|---------------|----------------|---------|
+  | Guardrail violation | Rajesh + on-call | Immediate | Telegram + SMS |
+  | API error rate >5% | Rajesh | 5 minutes | Telegram |
+  | SLO breach | Rajesh | 15 minutes | Telegram + email |
+  | Task timeout | NeurOS log | 30 minutes | Email |
+  
+  On-call rotation: Single-person (Rajesh) initially, expand as team grows
```

### Observation (ORG-FND-007)
Distributed tracing (trace_id) mentioned but implementation not specified.

### Recommendation
Define tracing system and propagation mechanism.

### Example Rewrite
```diff
   logging_audit:
-    trace_id: "ULID/Snowflake per task"
+    trace_id:
+      format: "ULID per task (sortable, 128-bit unique)"
+      propagation: "W3C Trace Context headers (traceparent, tracestate)"
+      backend: "Jaeger (self-hosted) or Grafana Tempo (cloud-friendly)"
+      retention: "14 days hot traces, 90 days sampled"
+      sampling: "100% for errors and high-risk tasks, 10% for routine tasks"
```

---

## Section 9: Versioning & Compatibility

### Observation (ORG-FND-008)
Compatibility matrix exists but breaking change detection and migration orchestration are missing. This is a MUST-fix.

### Recommendation
Add breaking change protocol with detection rules and migration steps.

### Example Addition
```markdown
### 9.4 Breaking Change Protocol

**Detection Rules:**
- Contract changes triggering major version bump:
  - Removing endpoints or fields
  - Changing field types or semantics
  - Modifying authentication schemes
  - Altering error codes or status enums

**Migration Process:**
1. **Proposal Phase:** Maker documents breaking change with migration guide
2. **Review Phase:** Checker validates backward compatibility impact
3. **Approval Phase:** Approver signs off with rollback criteria
4. **Coordination Phase:**
   - Announce deprecation 90 days in advance
   - Update integration_matrix.yaml with new compatibility ranges
   - Deploy new contract version alongside old (dual-support window)
5. **Execution Phase:**
   - Deploy NeurOS first (supports both old and new contracts)
   - Deploy Neuraj AI with adapter for new contract
   - Monitor error rates and fallback to old version if needed
6. **Cleanup Phase:**
   - After 90 days, deprecate old contract version
   - Remove dual-support code
   - Update audit logs with final migration status

**Rollback Criteria:**
- Error rate >5% within 24h of deployment
- Guardrail violations detected
- Task success rate <95%
```

### Observation (ORG-FND-019)
Contract deprecation timeline not specified.

### Recommendation
Define deprecation policy with support windows.

### Example Addition
```markdown
### 9.5 Deprecation Policy

| Contract Version | Support Duration | Deprecation Notice | End-of-Life Process |
|------------------|------------------|--------------------|--------------------|
| Current | Indefinite | N/A | N/A |
| Previous (N-1) | 12 months | 3 months advance | Graceful degradation, then hard cutoff |
| Legacy (N-2+) | Best-effort | Immediate | No guaranteed support, migrate urgently |

**Deprecation Communication:**
1. Update documentation with DEPRECATED badge
2. Add runtime warnings to API responses
3. Telegram notification to all stakeholders
4. Grafana dashboard showing usage of deprecated versions
```

---

## Section 10: Workflows

### Observation (ORG-FND-009)
Task lifecycle states are clear but timeout handling and stale task cleanup are undefined.

### Recommendation
Define task timeout policy with max execution times.

### Example Addition
```markdown
### 10.4 Task Timeout & Cleanup Policy

**Timeouts by Action Type:**
| Action | Max Runtime | Timeout Behavior |
|--------|-------------|------------------|
| run | 5 minutes | Fail with TIMEOUT error |
| build | 30 minutes | Fail with TIMEOUT error |
| test | 15 minutes | Fail with TIMEOUT error |

**Stale Task Cleanup:**
- Tasks in 'queued' >24h → Marked as ERROR with reason "EXPIRED"
- Tasks in 'running' beyond timeout → Killed, marked as ERROR "TIMEOUT"
- Tasks in 'needs_approval' >7 days → Marked as CANCELLED "NO_RESPONSE"
- Cleanup cron runs daily at 02:00 UTC
- Cleanup events logged to governance_events

**Timeout Extensions:**
- High-risk build tasks may request extension (max +30 min, requires approval)
- Extension requests logged and count against monthly quota (10/month)
```

### Observation (ORG-FND-014)
Quiz mechanism for risky actions lacks implementation details.

### Recommendation
Define quiz policy with content, scoring, and failure handling.

### Example Rewrite
```diff
   ### 10.2 Risky Action Flow  
   1. Request → risk_tier_eval  
-  2. If high → needs_approval + quiz  
+  2. If high → needs_approval + mandatory_quiz
+     - Quiz source: Last 3 incident RCA summaries + current market conditions
+     - Format: 5 multiple-choice questions
+     - Passing score: ≥4 correct (80%)
+     - Failure handling: Block action, log attempt, notify approver
+     - Retry: Max 2 retries within 1 hour, then 24h cooldown
+     - Quiz questions rotate weekly from question bank
   3. On approval → execute  
   4. On reject → cancel + audit_event  
```

---

## Section 11: Ethical & Compliance Framework

### Observation (ORG-FND-010)
Privacy principle exists but data subject rights (GDPR-style) are not addressed.

### Recommendation
Add data subject rights section even for minimal PII systems.

### Example Addition
```markdown
### 11.7 Data Subject Rights

While the Neuraj Organization minimizes PII storage, the following rights are supported:

| Right | Implementation |
|-------|---------------|
| **Right to Access** | User can request export of all governance_events and task logs associated with their user_id via support ticket (response within 30 days) |
| **Right to Deletion** | User can request deletion of personal identifiers from logs (anonymization to "user_<hash>") within 30 days |
| **Right to Portability** | User can export task history and artifacts in JSON format via API endpoint |
| **Right to Rectification** | User can request correction of inaccurate metadata in governance logs |

**Compliance Framework:**
- Applicable regulations: GDPR (if EU users), CCPA (if CA users)
- Data Protection Officer: Rajesh (interim)
- Privacy policy published at neuraj.org/privacy
- Breach notification: <72h to affected users and regulators
```

### Observation (ORG-FND-017)
Public communication strategy lacks incident disclosure policy.

### Recommendation
Add incident disclosure guidelines.

### Example Addition
```markdown
### 12.4 Incident Disclosure Policy

**What to Disclose:**
- Security incidents affecting user data or system integrity
- Prolonged service outages (>4 hours)
- Major contract version incompatibilities causing failures

**What NOT to Disclose:**
- Vulnerability details before patch deployment
- Internal investigation details
- User-specific information

**Timeline:**
1. Incident detected → Internal alert (immediate)
2. Containment achieved → Status page update (within 2h)
3. Root cause identified → Public blog post (within 72h)
4. Remediation complete → Final report with lessons learned (within 14 days)

**Communication Channels:**
- Status page: status.neuraj.org (to be implemented)
- Twitter: @Neuraj (brief updates)
- Blog: neuraj.org/blog (detailed reports)
- Email: Direct notification to affected users
```

---

## Section 4: Governance Model

### Observation (ORG-FND-011)
Maker-Checker-Approver flow is clear but conflict resolution is missing.

### Recommendation
Add conflict resolution and fallback procedures.

### Example Addition
```markdown
### 4.5 Conflict Resolution

**Scenario 1: Checker Rejects, Maker Disagrees**
1. Maker provides counter-evidence in response to findings.yaml
2. Second Checker review (different model in MyriadEye or peer human)
3. Approver makes final binding decision with documented rationale
4. If pattern of disagreement, escalate to quarterly policy review

**Scenario 2: Approver Unavailable**
- High-risk actions: Block until Approver returns (max 24h, then auto-reject)
- Medium-risk actions: Delegate to pre-designated Deputy Approver
- Low-risk actions: Auto-approve after 48h if no veto

**Scenario 3: Deadlock (2+ Reviewers Disagree)**
- Escalate to Approver with summary of positions
- Approver may request third-party expert review
- Final decision documented in audit_index.json with rationale

**Fallback Approver Designation:**
- Primary: Rajesh
- Deputy: [To be designated when team grows]
- Emergency: Neuraj AI can suggest but not approve high-risk actions
```

---

## Section 5.1: API Contract (Rate Limiting)

### Observation (ORG-FND-012)
Global rate limit exists but hierarchical limits (per-module, per-action) are missing.

### Recommendation
Define multi-tier rate limiting to prevent resource starvation.

### Example Rewrite
```diff
       rate_limits:
-        neuros_api: "30 req/min per actor"
+        neuros_api:
+          global: "30 req/min per actor (all endpoints combined)"
+          per_module:
+            scalping_pilot: "20 req/min (high priority)"
+            goldmine: "10 req/min (research, lower priority)"
+            default: "15 req/min"
+          per_action:
+            run: "20 req/min"
+            build: "5 req/min (resource intensive)"
+            test: "10 req/min"
+          burst: "Allow 2x rate for 10 seconds, then throttle"
+          backpressure: "429 response with Retry-After header"
```

---

## General Improvements

### Observation (ORG-FND-013)
Given zero-cost-first principle, tracking actual costs would validate adherence.

### Recommendation
Add cost observability section.

### Example Addition
```markdown
### 8.9 Cost Observability

**Cost Tracking Metrics:**
- `neuraj_api_calls_total{provider, model}` — LLM API call counts
- `neuraj_compute_seconds{service, environment}` — Total compute time
- `neuraj_storage_bytes{layer, type}` — Storage consumption (DB, vector, archive)
- `neuraj_network_egress_bytes{destination}` — Network transfer costs

**Cost Budgets & Alerts:**
| Resource | Monthly Budget | Alert Threshold |
|----------|---------------|-----------------|
| LLM API calls | 10,000 calls | 80% (8,000) |
| Compute (DEV) | 100 hours | 80% |
| Storage | 50 GB | 80% |
| Network egress | 10 GB | 80% |

**Cost Attribution:**
- Tag all resources with module_id for cost allocation
- Monthly cost report generated automatically
- Zero-cost validation: Flag any service exceeding free tier
```

### Observation (ORG-FND-016)
Encryption requirements implied but not explicit.

### Recommendation
Explicitly state encryption standards.

### Example Addition
```markdown
### 6.7 Encryption Standards

**Data in Transit:**
- All HTTP connections: TLS 1.3 (minimum TLS 1.2)
- Certificate pinning for service-to-service (STAGE+)
- Cipher suites: ECDHE-RSA-AES256-GCM-SHA384 or stronger

**Data at Rest:**
- PostgreSQL: AES-256 encryption (pg_crypto or filesystem-level)
- Vector store: AES-256 encryption on disk
- Backup archives: GPG encryption before upload to cold storage
- Audit logs: Signed with PGP, optionally encrypted

**Key Management:**
- Secrets stored in HashiCorp Vault (self-hosted) or encrypted .env files
- Key rotation: Quarterly for symmetric keys, annually for asymmetric keys
- Key escrow: Backup keys stored offline in secure location
```

---

## Top 3 Strengths

1. **Exceptional Governance Model** — The Maker-Checker-Approver framework with clear cadences and audit trails sets a gold standard for AI system governance. The MyriadEye protocol itself demonstrates institutional-grade thinking.

2. **Comprehensive Integration Architecture** — The separation of concerns between Org (rules), AI (reasoning), and OS (execution) with explicit contracts and dependency maps is masterful. This enables independent evolution while maintaining coherence.

3. **Practical Operationalization** — The choice of proven, zero-cost technologies (PostgreSQL, Prometheus, FAISS) combined with realistic SLOs and canary deployment shows deep understanding of production constraints. This blueprint can be implemented immediately.

---

## Top 3 Risks

1. **Audit Immutability Mechanism Undefined** (ORG-FND-003, MUST) — Without a specified technical mechanism for immutable logs, audit integrity is not guaranteed. This is the highest priority fix as it affects legal and operational defensibility.

2. **Breaking Change Orchestration Missing** (ORG-FND-008, MUST) — As the ecosystem grows, uncoordinated contract upgrades could cause system-wide failures. The lack of a formal migration protocol is a scaling risk that will manifest with the first major version bump.

3. **Security Token Lifecycle Gaps** (ORG-FND-004, SHOULD) — Production deployments without explicit session timeout, token TTL, and rotation policies create attack surface. While HMAC and mTLS are strong, missing lifecycle policies weaken the security posture over time.

---

**End of Proposals Document**
