proposals.md
---
## Top 3 Strengths

1.  **Clear Separation of Powers:** The "Board, CSO, COO" analogy (Org, AI, OS) is exceptionally clear and creates a robust, auditable hierarchy. This design inherently promotes accountability.
2.  **Pragmatic "Zero-Cost-First" Principle:** This principle anchors the architecture in realistic, maintainable, and portable open-source technologies (Postgres, Prometheus, FAISS), ensuring long-term feasibility.
3.  **Strong Ops & Observability:** Section 8 is comprehensive, defining clear SLOs, error budgets, rollback triggers, and incident response levels. This makes the system testable and manageable from day one.

## Top 3 Risks

1.  **Data Contract Incompleteness:** The `tasks` table schema (Section 5.2) is missing critical timestamp columns. As all Ops/SLOs (Section 8) depend on these for latency and audit, this is a MUST-fix integration blocker.
2.  **Ambiguous Human-in-the-Loop Authority:** The `approve_high_risk` capability (Section 6) giving Neuraj AI "co-sign" status is a security and governance risk. It conflicts with the core principle of final human authority and must be clarified to be a *review* step, not an *approval* step.
3.  **Missing Governance Workflows:** The blueprint omits two key processes: 1) Who performs `risk_tier_eval` (Section 10.2) and 2) How the 'Recall Layer' (Vector Store) is updated when policies change (Section 7). These gaps break the end-to-end governance loop.

---

## [Section 4. Governance Model]

-   **Observation:** The "Checker" role is ambiguously defined as "Validator AI / Peer".
-   **Recommendation:** Clarify this to remove the "Peer" ambiguity.
-   **Example Rewrite:**

```diff
| Role | Responsibility | Artifacts |
|------|----------------|------------|
| **Maker (LLM / Developer)** | Drafts or proposes changes, builds modules. | `proposal.md`, `patch.diff`, logs |
| **Checker (Validator AI)** | Reviews logic, compliance, and risks. | `review.yaml`, test reports |
| **Approver (Rajesh)** | Final decision authority; merges or vetoes. | `approval.md`, `audit_index.json` |

---

## [Section 5. Contracts & Interfaces]

- Observation (5.1): The HMAC-SHA256 definition is missing the timestamp/nonce detail, which is critical for security.
- Recommendation (5.1): Add the timestamp detail from the Integration_Context.yaml.
- Example Rewrite (5.1):
    - **Auth:** HMAC-SHA256 (Stage) · mTLS + HMAC (Prod)  
    + **Auth:** HMAC-SHA256 (Stage, w/ timestamp nonce) · mTLS + HMAC (Prod)

-   Observation (5.2): The tasks table is missing critical columns for auditing and SLO tracking. This is a MUST-FIX.
-   Recommendation (5.2): Update the tasks table definition to match the more complete version in Integration_Context.yaml.
-   Example Rewrite (5.2):
        | `tasks` | id, module_id, action, params_json, status, requester, approver, timestamps |
        + | `tasks` | id, module_id, action, params_json, status, requester, approver, created_at, started_at, finished_at |

## [Section 6. Security & RBAC]

-   Observation (1): The approve_high_risk capability for "Rajesh + Neuraj AI (co-sign)" is ambiguous and dilutes the "Human-in-the-Loop" principle.
-   Recommendation (1): Rephrase to make it a mandatory review step, where Rajesh retains sole approval authority.

-   Observation (2): The autonomous trigger conditions for the NeurOS kill_switch are undefined.
-   Recommendation (2): Add a footnote clarifying the autonomous trigger.

-   Example Rewrite:
    | Capability | Actors |
    |-------------|---------|
    | `read_policies` | Rajesh, Neuraj AI, NeurOS Lite |
    | `propose_tasks` | Rajesh, Neuraj AI |
    | `execute_tasks` | NeurOS Lite |
    |- `approve_high_risk` | Rajesh + Neuraj AI (co-sign) |
    |+ `approve_high_risk` | Rajesh (requires Neuraj AI review) |
    | `edit_policies` | Rajesh only |
    | `kill_switch` | Rajesh + NeurOS Lite [1] |

    + [1] NeurOS Lite is authorized to autonomously trigger the kill-switch *only* if a P1-level guardrail (e.g., Daily Loss Cap) is breached.

## [Section 7. Data & Knowledge Architecture]

- Observation: The process for updating the Recall Layer (Vector Store) is missing.
- Recommendation: Add a new subsection defining this crucial knowledge loop.
- Example Rewrite (add new 7.1):

    ## 7️⃣ Data & Knowledge Architecture  
    | Layer | Description | Engine |
    |--------|--------------|---------|
    | **Truth Layer** | Canonical state — tasks, artifacts, KPIs, governance | PostgreSQL |
    | **Performance Layer** | Metrics & SLO tracking | Prometheus + Grafana |
    | **Recall Layer** | Semantic memory with SHA provenance | Vector Store (FAISS/Chroma) |

    + ### 7.1 Recall Layer Update Workflow
    + 
    + The Recall Layer must remain in sync with the Truth Layer.
    + - **Trigger:** Any `governance_event` of kind `policy_approved` or `doc_merged`.
    + - **Action:** A NeurOS system task is triggered to:
    +   1. Parse the new/updated artifact (e.g., `*.md`, `*.yaml`).
    +   2. Generate embeddings for each section.
    +   3. Write to the Vector Store with new metadata (SHA, source_path, timestamp).
    + - **Audit:** This update task is logged in the `tasks` table like any other system action.

    Retention:  
    ...

## [Section 8. Observability & Ops]

- Observation: The P1 Incident Response to "rotate keys" is a very aggressive automated action that could cause more harm.
- Recommendation: Separate immediate containment (kill-switch) from the more delicate recovery step (key rotation), making the latter a human-driven follow-up.
- Example Rewrite:
    | Level | Action |
    |-------|---------|
    |- P1 | Kill-switch, rotate keys, rollback image, alert, RCA < 24 h |
    |+ P1 | Kill-switch, rollback image, alert, schedule key rotation, RCA < 24 h |
    | P2 | Throttle + canary rollback |
    | P3 | Log + schedule next window |

## [Section 10. Workflows]

- Observation: The actor/process for risk_tier_eval in the Risky Action Flow is undefined.
- Recommendation: Explicitly assign this responsibility.
- Example Rewrite (10.2):
### 10.2 Risky Action Flow  
    - 1. Request → risk_tier_eval  
    + 1. Request → Neuraj AI proposes `risk_tier` in task proposal.
    + 2. NeurOS Lite validates `risk_tier` against module-specific rules.
    - 2. If high → needs_approval + quiz  
    + 3. If `risk_tier` is high → needs_approval + quiz  
    - 3. On approval → execute  
    + 4. On approval → execute  
    - 4. On reject → cancel + audit_event  
    + 5. On reject → cancel + audit_event

