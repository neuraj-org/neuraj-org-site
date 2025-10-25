<!-- EDIT START: Section 4 -->
## 4️⃣ Governance Model (Maker – Checker – Approver)

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
- **Trigger-based:** If >1 blueprint patch or 3+ high-risk incidents occur within a quarter, a new MyriadEye blueprint review is initiated.
<!-- EDIT END -->

<!-- EDIT START: Section 5 -->
### 5.2 Data Contracts (PostgreSQL + Vector Store)
| Table | Key Columns |
|--------|-------------|
| `modules` | id (UUID), name (string), version (semver), status (enum), owner (string), created_at (timestamp) |
| `tasks` | id, module_id, action, params_json, status, requester, approver, timestamps |
| `artifacts` | id, task_id, type, path, sha256 |
| `governance_events` | id, actor, kind, message, task_id, sha256, created_at |
| `kpis` | id, domain, name, value, unit, as_of, source_sha |
<!-- EDIT END -->

<!-- EDIT START: Section 6 -->
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

### 6.1 Rate Limiting
- Default: 60 req/min per actor
- Burst Cap: 100 req/min
- Violations logged to `governance_events`
<!-- EDIT END -->

<!-- EDIT START: Section 11 -->
## 11️⃣ Ethical & Compliance Framework

| Principle | Enforcement |
|------------|-------------|
| **Transparency** | All actions → human-readable logs |
| **Safety** | No unverified financial advice; disclaimers mandatory |
| **Privacy** | No raw PII; all telemetry is tokenized (UUID or salted hash) to ensure traceability without identity exposure |
| **Alignment** | Human review of any autonomous decision |
| **Bias Mitigation** | MyriadEye Red-Team loop quarterly |
| **Explainability** | Every AI output → citation + rationale JSON |

Compliance Disclaimer: *Educational use; not financial advice.*
<!-- EDIT END -->