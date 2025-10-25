# 🧱 Neuraj Organization Blueprint — v1.0  
### The Institutional Backbone of the Neuraj Ecosystem  

**Date:** October 2025  
**Prepared by:** Rajesh (Founder & Approver) · GPT-5 (Architect)  
**Scope:** Defines governance, contracts, and data frameworks that power Neuraj AI (Brain) and NeurOS Lite (Executor).  
**Review Protocol:** MyriadEye Round #1 — Organization Layer  
**Revision:** Claude Sonnet 4.5 validator edits (v1.1 candidate)

---

## 1️⃣ Purpose  

To establish a formal, auditable, and scalable foundation for the **Neuraj Organization**, ensuring that all intelligent and operational subsystems—present and future—operate under a single, transparent, and ethical governance model.  

This blueprint functions as the **Constitution** of the Neuraj ecosystem.  
Every derivative blueprint (Neuraj AI, NeurOS Lite, or Departmental Modules) must comply with it.  

---

## 2️⃣ Core Principles  

| Pillar | Description |
|--------|-------------|
| **Accountability** | Every AI action must have a traceable rationale, approval, and log. |
| **Auditability** | All systems maintain immutable, human-readable records. |
| **Adaptability** | Architecture must scale without refactoring core policies. |
| **Human-in-the-Loop** | Final authority always resides with the Approver. |
| **Zero-Cost-First** | Prefer open-source, free, and portable tech stacks. |
| **Transparency** | All reasoning, metrics, and decisions remain explainable. |
| **Resilience** | Failure or drift must trigger automatic rollback and review. |

---

## 3️⃣ Organizational Structure  

```
Rajesh (Human Approver)
│
├── 🧠 Neuraj AI (Brain)
│     ├─ Planning & Rationale
│     └─ L&D / Knowledge Loop
│
└── ⚙️ NeurOS Lite (Executor)
      ├─ Module Orchestration
      ├─ Task Execution
      └─ Metrics & Guardrails
```

**Neuraj Organization** = "Board of Directors"  
**Neuraj AI** = "Chief Strategy Officer"  
**NeurOS Lite** = "Chief Operations Officer"  

All subordinate modules (Scalping Pilot, Risk Manager, Profit Trailer, GoldMine, etc.) report via NeurOS.  

---

## 4️⃣ Governance Model (Maker — Checker — Approver)

| Role | Responsibility | Artifacts |
|------|----------------|------------|
| **Maker (LLM / Developer)** | Drafts or proposes changes, builds modules. | `proposal.md`, `patch.diff`, logs |
| **Checker (Validator AI / Peer)** | Reviews logic, compliance, and risks. | `review.yaml`, test reports |
| **Approver (Rajesh)** | Final decision authority; merges or vetoes. | `approval.md`, `audit_index.json` |

Governance cadence:
- **Daily:** Micro-reviews and metrics sync  
- **Weekly:** Departmental summaries  
- **Monthly:** Red-Team + Incident Drill  
- **Quarterly:** Policy revision and MyriadEye check  

<!-- EDIT START: Section 4 -->
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

### 4.6 Governance Audit Retention

- **Hot Period:** 24 months (full access via API and dashboards)
- **Warm Archive:** 25-84 months (searchable, slower retrieval)
- **Cold Archive:** 85+ months (tar + index, requires manual restore)
- **Destruction Policy:** No destruction for governance decisions; indefinite retention for legal/compliance
<!-- EDIT END -->

---

## 5️⃣ Contracts & Interfaces  

### 5.1 API Contract (`CONTRACT-1.0.0`)
- **Transport:** HTTPS / REST  
- **Auth:** HMAC-SHA256 (Stage) · mTLS + HMAC (Prod)  
- **Core Endpoints:**
  - `POST /v1/tasks/run` — Execute action  
  - `POST /v1/modules/build` — Scaffold new department  
  - `GET /v1/tasks/{id}/status` — Check progress  
  - `GET /v1/registry` — List available modules  
  - `GET /v1/health` — Liveness probe  

<!-- EDIT START: Section 5.1 -->
**Request Schema:**
```yaml
task_id: "string | auto"
requester: "enum: [neuraj_ai]"
action: "enum: [run, build, test]"
module: "string (e.g., scalping_pilot, profit_trailer)"
params: "object (free-form, validated by module)"
governance:
  risk_level: "enum: [low, med, high]"
  approved_by: "string (human id)"
  notes: "string"
```

**Success Response Schema:**
```yaml
task_id: "string"
status: "enum: [queued, running, needs_approval, blocked, done, error]"
artifacts: "array of {type, path_or_ref, sha256}"
metrics_ref: "map of labels (e.g., prom_labels.task_id)"
log_ref: "file/line reference (e.g., governance_log.json#L120)"
message: "human summary string"
```

**Error Response Schema:**
```yaml
error_code: "string (e.g., INVALID_MODULE, GUARDRAIL_VIOLATION, RESOURCE_EXHAUSTED)"
message: "human-readable error description"
details: "object (context-specific error data)"
trace_id: "string (ULID for correlation)"
retry_after: "integer (seconds) | null"
http_status: "integer (400 bad request, 403 forbidden, 429 rate limited, 500 internal error)"
```

**Rate Limiting:**
```yaml
global: "30 req/min per actor (all endpoints combined)"
per_module:
  scalping_pilot: "20 req/min (high priority)"
  goldmine: "10 req/min (research, lower priority)"
  default: "15 req/min"
per_action:
  run: "20 req/min"
  build: "5 req/min (resource intensive)"
  test: "10 req/min"
burst: "Allow 2x rate for 10 seconds, then throttle"
backpressure: "429 response with Retry-After header"
```
<!-- EDIT END -->

### 5.2 Data Contracts (PostgreSQL + Vector Store)
| Table | Key Columns |
|--------|-------------|
| `modules` | id, name, version, status, owner, created_at |
| `tasks` | id, module_id, action, params_json, status, requester, approver, timestamps |
| `artifacts` | id, task_id, type, path, sha256 |
| `governance_events` | id, actor, kind, message, task_id, sha256, created_at |
| `kpis` | id, domain, name, value, unit, as_of, source_sha |

<!-- EDIT START: Section 5.2 -->
**Foreign Key Constraints:**
- `tasks.module_id → modules.id` (ON DELETE CASCADE)
- `artifacts.task_id → tasks.id` (ON DELETE CASCADE)
- `governance_events.task_id → tasks.id` (ON DELETE RESTRICT)

**Indexes:**
- `tasks`: `status`, `created_at DESC`, `(module_id, status)` composite
- `governance_events`: `actor`, `created_at DESC`, `task_id`
- `kpis`: `(domain, name, as_of DESC)` composite for fast latest-value queries
<!-- EDIT END -->

**Vector Store:** FAISS / Chroma for Neuraj AI knowledge recall  
Metadata → `{source_path, sha, section, timestamp}`  

<!-- EDIT START: Section 5.2 -->
**Vector Store Operations:**
- **Snapshot frequency:** Daily incremental, weekly full
- **Backup retention:** 7 daily, 4 weekly, 12 monthly
- **Recovery SLA:** <1 hour for latest snapshot
- **Failover:** Read-replica in different availability zone (STAGE+)
- **Consistency check:** Weekly SHA verification against source documents
<!-- EDIT END -->

---

## 6️⃣ Security & RBAC  

| Capability | Actors |
|-------------|---------|
| `read_policies` | Rajesh, Neuraj AI, NeurOS Lite |
| `propose_tasks` | Rajesh, Neuraj AI |
| `execute_tasks` | NeurOS Lite |
| `approve_high_risk` | Rajesh + Neuraj AI (co-sign) |
| `edit_policies` | Rajesh only |
| `kill_switch` | Rajesh + NeurOS Lite |

Guardrails:
- Market restart → 30 min cool-down + quiz  
- Daily loss cap → hard halt + manual unlock  
- Max trades/day limit  
- Credential changes → dual approval  

<!-- EDIT START: Section 6 -->
### 6.7 Authentication & Session Management

**Service-to-Service:**
- **STAGE:** HMAC-SHA256 (timestamp window ±120s)
- **PROD:** mTLS + HMAC (defense in depth)

**Token & Session Policy:**
- **HMAC token TTL:** 1 hour (tokens expire and must be refreshed)
- **mTLS cert rotation:** 90 days (automated via cert-manager)
- **Session timeout:** 24 hours idle, 7 days absolute
- **Key rotation:** Quarterly for HMAC secrets, immediate on incident

**Human App Authentication:**
- **DEV:** Local (no auth)
- **LATER:** OAuth2 with MFA for production access

### 6.8 Audit Log Immutability

**Governance Log (`governance_log.json`):**
- **Mechanism:** Append-only file system flag (`chattr +a` on Linux)
- **Integrity:** SHA-256 hash chain — each entry includes hash of previous entry
- **Verification:** Merkle tree root published to `audit_index.json` daily
- **Tampering detection:** Weekly automated verification of hash chain

**Daily Briefs:**
- **Mechanism:** Write-once, signed with PGP key
- **Retention:** Immutable for 12 months, then archive
- **Verification:** PGP signature validation on read

### 6.9 Encryption Standards

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

### 6.10 Circuit Breaker Pattern

To improve resilience against downstream failures:

| Service | Failure Threshold | Timeout | Half-Open Retry |
|---------|------------------|---------|-----------------|
| NeurOS API | 5 failures in 60s | 30s | 1 request after 60s |
| PostgreSQL | 3 failures in 30s | 10s | 1 query after 30s |
| Prometheus | 5 failures in 60s | 15s | 1 request after 45s |
| Vector Store | 3 failures in 30s | 20s | 1 query after 30s |

**Behavior:**
- **Closed:** Normal operation
- **Open:** Fail fast, return cached data or error
- **Half-Open:** Test with single request, reopen if success, else stay open
<!-- EDIT END -->

---

## 7️⃣ Data & Knowledge Architecture  

| Layer | Description | Engine |
|--------|--------------|---------|
| **Truth Layer** | Canonical state — tasks, artifacts, KPIs, governance | PostgreSQL |
| **Performance Layer** | Metrics & SLO tracking | Prometheus + Grafana |
| **Recall Layer** | Semantic memory with SHA provenance | Vector Store (FAISS/Chroma) |

Retention:  
`hot: 60 days · warm: 12 months · archive: tar + index`  

Access:
- **Neuraj AI:** read-only views or NOS API  
- **NeurOS:** controlled read/write per module ownership  

---

## 8️⃣ Observability & Ops  

- **Metrics:** `nos_task_success_total`, `nos_task_error_total`, `guardrail_violations_total`, latency buckets  
- **Dashboards:** `org_overview.json`, `executor_health.json`, `guardrails.json`, `kpis_overview.json`  
- **Availability SLO:** ≥ 99.5 % (7-day)  
- **Error Budget:** ≤ 2 % failures  
- **Latency P95:** ≤ 3 s (read) · ≤ 30 s (build)

**Canary & Rollback**
```yaml
canary_percent: 10
auto_rollback_triggers:
  - error_rate_threshold_breach
  - guardrail_violation_detected
  - latency_p95_breach
```

**Incident Severity & Response**
| Level | Action |
|-------|---------|
| P1 | Kill-switch, rotate keys, rollback image, alert, RCA < 24 h |
| P2 | Throttle + canary rollback |
| P3 | Log + schedule next window |

<!-- EDIT START: Section 8 -->
### 8.8 Alert Policies & Escalation

| Alert Type | Initial Notify | Escalate After | Channel |
|------------|---------------|----------------|---------|
| Guardrail violation | Rajesh + on-call | Immediate | Telegram + SMS |
| API error rate >5% | Rajesh | 5 minutes | Telegram |
| SLO breach | Rajesh | 15 minutes | Telegram + email |
| Task timeout | NeurOS log | 30 minutes | Email |

**On-call Rotation:** Single-person (Rajesh) initially, expand as team grows

### 8.9 Distributed Tracing

**Trace ID:**
- **Format:** ULID per task (sortable, 128-bit unique)
- **Propagation:** W3C Trace Context headers (`traceparent`, `tracestate`)
- **Backend:** Jaeger (self-hosted) or Grafana Tempo (cloud-friendly)
- **Retention:** 14 days hot traces, 90 days sampled
- **Sampling:** 100% for errors and high-risk tasks, 10% for routine tasks

### 8.10 Cost Observability

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
- Tag all resources with `module_id` for cost allocation
- Monthly cost report generated automatically
- Zero-cost validation: Flag any service exceeding free tier
<!-- EDIT END -->

---

## 9️⃣ Versioning & Compatibility  

| Component | Versioning | Compatible Range |
|------------|-------------|------------------|
| CONTRACT | v1.0.0 | fixed |
| Neuraj AI | ≥ 1.0 < 2.0 |
| NeurOS Lite | ≥ 1.0 < 2.0 |
| Modules | MOD.<name> MAJOR.MINOR.PATCH |

Integration matrix stored in `governance/integration_matrix.yaml`.

<!-- EDIT START: Section 9 -->
### 9.4 Breaking Change Protocol

**Detection Rules:**
Contract changes triggering major version bump:
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
   - Update `integration_matrix.yaml` with new compatibility ranges
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

### 9.5 Deprecation Policy

| Contract Version | Support Duration | Deprecation Notice | End-of-Life Process |
|------------------|------------------|--------------------|--------------------|
| Current | Indefinite | N/A | N/A |
| Previous (N-1) | 12 months | 3 months advance | Graceful degradation, then hard cutoff |
| Legacy (N-2+) | Best-effort | Immediate | No guaranteed support, migrate urgently |

**Deprecation Communication:**
1. Update documentation with `DEPRECATED` badge
2. Add runtime warnings to API responses
3. Telegram notification to all stakeholders
4. Grafana dashboard showing usage of deprecated versions
<!-- EDIT END -->

---

## 🔟 Workflows  

### 10.1 Task Lifecycle  
States: `queued → running → needs_approval → blocked → done → error`  
All transitions logged with `trace_id`.  

### 10.2 Risky Action Flow  
1. Request → risk_tier_eval  
2. If high → needs_approval + mandatory_quiz  
   - **Quiz source:** Last 3 incident RCA summaries + current market conditions
   - **Format:** 5 multiple-choice questions
   - **Passing score:** ≥4 correct (80%)
   - **Failure handling:** Block action, log attempt, notify approver
   - **Retry:** Max 2 retries within 1 hour, then 24h cooldown
   - **Quiz questions:** Rotate weekly from question bank
3. On approval → execute  
4. On reject → cancel + audit_event  

### 10.3 Daily Brief  
07:30 → NOS collects KPIs → renders markdown → Telegram notify → Grafana link  

<!-- EDIT START: Section 10 -->
### 10.4 Task Timeout & Cleanup Policy

**Timeouts by Action Type:**
| Action | Max Runtime | Timeout Behavior |
|--------|-------------|------------------|
| run | 5 minutes | Fail with TIMEOUT error |
| build | 30 minutes | Fail with TIMEOUT error |
| test | 15 minutes | Fail with TIMEOUT error |

**Stale Task Cleanup:**
- Tasks in `queued` >24h → Marked as ERROR with reason "EXPIRED"
- Tasks in `running` beyond timeout → Killed, marked as ERROR "TIMEOUT"
- Tasks in `needs_approval` >7 days → Marked as CANCELLED "NO_RESPONSE"
- Cleanup cron runs daily at 02:00 UTC
- Cleanup events logged to `governance_events`

**Timeout Extensions:**
- High-risk build tasks may request extension (max +30 min, requires approval)
- Extension requests logged and count against monthly quota (10/month)
<!-- EDIT END -->

---

## 11️⃣ Ethical & Compliance Framework  

| Principle | Enforcement |
|------------|-------------|
| **Transparency** | All actions → human-readable logs |
| **Safety** | No unverified financial advice; disclaimers mandatory |
| **Privacy** | No PII storage; anonymized records only |
| **Alignment** | Human review of any autonomous decision |
| **Bias Mitigation** | MyriadEye Red-Team loop quarterly |
| **Explainability** | Every AI output → citation + rationale JSON |

Compliance Disclaimer: *Educational use; not financial advice.*

<!-- EDIT START: Section 11 -->
### 11.7 Data Subject Rights

While the Neuraj Organization minimizes PII storage, the following rights are supported:

| Right | Implementation |
|-------|---------------|
| **Right to Access** | User can request export of all `governance_events` and task logs associated with their `user_id` via support ticket (response within 30 days) |
| **Right to Deletion** | User can request deletion of personal identifiers from logs (anonymization to `user_<hash>`) within 30 days |
| **Right to Portability** | User can export task history and artifacts in JSON format via API endpoint |
| **Right to Rectification** | User can request correction of inaccurate metadata in governance logs |

**Compliance Framework:**
- Applicable regulations: GDPR (if EU users), CCPA (if CA users)
- Data Protection Officer: Rajesh (interim)
- Privacy policy published at `neuraj.org/privacy`
- Breach notification: <72h to affected users and regulators
<!-- EDIT END -->

---

## 12️⃣ Public Identity  

- **Website:** `neuraj.org` — static mission & values  
- **Hosting:** Contabo VPS · DNS: Google Domains  
- **Social:**  
  - Twitter: `@Neuraj` (`#NeurajNotes`, AI-voice tone)  
  - LinkedIn: `Neuraj Organization` (Professional updates)  

<!-- EDIT START: Section 12 -->
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
- Status page: `status.neuraj.org` (to be implemented)
- Twitter: `@Neuraj` (brief updates)
- Blog: `neuraj.org/blog` (detailed reports)
- Email: Direct notification to affected users
<!-- EDIT END -->

---

## 13️⃣ Success Criteria  

- MyriadEye Review ≥ 4.0 average rubric score  
- No MUST-level issues open  
- All approved patches merged → `v1.0.R1`  
- Red-Team tests: critical = PASS  
- Integration Matrix P1/P2 validated  

---

## 14️⃣ Guiding Ethos  

> "We are not building a product.  
> We are building a civilization of intelligent systems that serve responsibly."

---

<!-- EDIT START: Appendix -->
## 15️⃣ Glossary (Expanded)

| Term | Definition |
|------|------------|
| **Brain** | Neuraj AI — reasoning/orchestration layer |
| **Executor** | NeurOS Lite — operations/guardrails layer |
| **Contract** | Shared API & data schemas defined by Org |
| **Modules** | Departments executed by NeurOS (e.g., Scalping Pilot) |
| **MyriadEye** | Parallel multi-LLM review protocol for governance |
| **RAG** | Retrieval-Augmented Generation with provenance citations |
| **ULID** | Universally Unique Lexicographically Sortable Identifier (128-bit, timestamp-prefixed) |
| **HMAC** | Hash-based Message Authentication Code — cryptographic authentication method |
| **mTLS** | Mutual TLS — both client and server authenticate with certificates |
| **WORM** | Write Once Read Many — immutable storage mechanism |
| **Circuit Breaker** | Design pattern to prevent cascading failures by failing fast when downstream service is unavailable |
| **Canary Deployment** | Gradual rollout strategy (e.g., 10% traffic) to detect issues before full deployment |
| **P95 Latency** | 95th percentile latency — 95% of requests complete within this time |
| **Merkle Tree** | Hash tree structure for efficient verification of data integrity |
| **PGP** | Pretty Good Privacy — encryption and signing system for data and communications |
<!-- EDIT END -->

---

**End of Blueprint**  
*Filed under `/Neuraj_Organization/docs/blueprints/Neuraj_Organization_Blueprint_v1.1.md`*  
*Revision by: Claude Sonnet 4.5 (MyriadEye Validator)*
