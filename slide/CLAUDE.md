# CLAUDE.md — ESMOS Healthcare Go-Live Project Context

## Project Identity

- **Course**: IS214 Enterprise Solution Management (ESM), SMU, AY2025-26 Term 2
- **Group**: e08g08t05
- **Instructor**: Kiru
- **Project Title**: ESMOS Healthcare Go-Live: Odoo Operations + Moodle Training + Compliant Support
- **Framework**: ITIL 4.0 (NOT ITIL v3 — never use lifecycle terminology like "Service Strategy → Service Design → Service Transition → Service Operation → CSI")

## Team Roles

| Member   | Role                 |
|----------|----------------------|
| Yichen   | System Configurator  |
| Seann    | Support Manager      |
| Zachary  | Security Manager     |
| Sahanya  | Product Manager      |
| Shawmya  | Data Manager         |

## Timeline & Deliverables

| Week | Milestone | Weight | Focus |
|------|-----------|--------|-------|
| 4    | PM1 (Ungraded) | 0% | Service Strategy & Design |
| 7    | PM2 | 10% | Incident Management + Stakeholder Comms |
| 10   | Week 10 Proposal Presentation | Graded (part of PM3) | 5-min presentation + 5-min Q&A to CEO |
| 9-12 | PM3 | 15% | Overall Service Strategy, Operations, and Change |
| 12   | Final Presentation | 10% | Live demo of infrastructure + operations |

## Scenario

A healthcare client approved ESMOS Odoo IaaS deployment for production. They now require:
- **Odoo**: Meal plan inventory and operations
- **Moodle**: Mandatory compliance training for 500+ staff (~50 concurrent baseline, 100 concurrent designed capacity)
- **Compliant helpdesk**: Incidents, service requests, changes — all tracked
- **All systems**: Hosted in Asia Pacific for data residency compliance
- **Internal-only access**: No public access; simulate via IP restriction (e.g., SMU IP range)
- **Access control**: Odoo operational accounts granted ONLY after staff complete Moodle compliance training
- **Integration**: Process-level (human workflow via helpdesk), NOT automated SSO/API

## Architecture Decision: AKS (Option 4)

**Chosen**: Azure Kubernetes Service, Southeast Asia region, single cluster.

### Stack
```
Ingress Layer:    Nginx Ingress Controller (TLS termination, rate limiting)
Application Pods: Odoo Pod | Moodle Pod | Helpdesk (Odoo module)
Data Layer:       PostgreSQL | Azure Managed Disk (Persistent Volumes)
```

### Justification (use BECAUSE reasoning)
- HPA (Horizontal Pod Autoscaler) meets 100-concurrent-user capacity without permanent over-provisioning BECAUSE we scale Moodle pods during training windows and scale down otherwise
- Self-healing via pod restart policies drops MTTR from manual SSH intervention to seconds BECAUSE liveness/readiness probes detect and replace failed pods automatically
- Kubernetes RBAC + namespaces enforce least-privilege BECAUSE namespace-scoped roles prevent cross-service access
- AKS control plane is free; Standard_B2s node ~$30/month fits Azure for Students credits
- Single cluster in Southeast Asia satisfies data residency

### Known weaknesses to defend in Q&A
- **Cost**: ~$30+/month vs previous ~$4.32/month VM. Budget runway ~3 months on $100 credits. Must justify with Azure Calculator evidence.
- **Team skill gap**: All prior experience is VM + SSH + Azure DevOps bash pipelines. No demonstrated Kubernetes competency yet. If you can't demo it, don't propose it.
- **"Isn't AKS overkill?"**: ESA answer (not Technical SA): "Healthcare uptime commitments require self-healing and autoscaling that our 5-person team cannot staff manually for 24/7 recovery. AKS automates the recovery we cannot staff for."
- **Docker Compose comparison**: `docker compose up --build -d` achieves near-zero downtime too. AKS rolling updates are marginally better (new pod health-checked before old killed). Lead with self-healing and HPA, NOT rolling updates, as differentiators.

### ITIL guiding principle tension
"Start where you are" = team's maturity is at VM level. Jumping to K8s contradicts this unless you frame it as: "We are starting where we are (Azure DevOps pipelines) and extending to AKS because the business requirement for self-healing exceeds what our current VM approach can deliver."

## Operations Model

### Onboarding Workflow (Moodle → Odoo Access)

**BPMN Swimlanes**: New Staff | Support Manager | Department Manager | System Configurator

```
Start → Staff submits "Odoo Access Request" via helpdesk guest portal
  → Support Manager receives, logs service request ticket
  → Support Manager validates: legitimate new hire? [Gateway]
    → No: Reject, notify staff, End
    → Yes: Assign Moodle compliance course, notify staff
  → Staff completes training on Moodle
  → Staff submits completion proof via ticket update
  → Support Manager verifies completion in Moodle records [Gateway]
    → Incomplete: Return with specific modules required
    → Complete: Approve Odoo account creation
  → System Configurator creates Odoo account (least-privilege role)
  → System Configurator confirms on ticket
  → Staff receives credentials + onboarding docs
  → End
```

**KNOWN GAP — needs fixing**: No error handling or timeout. Add:
- BPMN timer event: No completion within 14 days → escalate to Department Manager
- Backup assignee if Support Manager unavailable
- Dispute path if completion proof is contested

**SLA targets**:
- Moodle access: < 4 hours after request
- Odoo access: < 24 hours after training completion
- Overall service request: < 24 hours

### Incident & Change Handling

- P1 incident: 15 min acknowledge, 2 hour resolve
- RASCI escalation: Support Manager triages (R), System Configurator resolves (R), Security Manager consulted (C) for security events
- **Client notification**: ONLY for Major/Critical incidents with significant business impact. Informational events (backup success, routine restarts) stay internal. Over-informing creates noise per RASCI principles.
- All non-emergency changes go through RFC process: documented in helpdesk, reviewed by Change Authority, rollback plan before execution

### Event Classification (apply correctly)
- **Informational**: Backup success, user login/logout, system startup → logged internally, no client notification
- **Warning**: CPU at 85%, low disk space, high memory → investigate, preventive action, internal team only
- **Exception**: Server crash, network outage, security breach → trigger incident management, client informed if service-impacting

## Quality (Kano Model)

### Dissatisfiers (Basic / Threshold — unspoken, must-have)
- System down during training window
- Data stored outside Asia Pacific
- Unauthorized staff accessing meal/patient-adjacent data
- Moodle can't handle 50 concurrent users (baseline minimum)

### Satisfiers (Desired / Performance — spoken, paid for)
- Incident response ≤ 2 hours during business hours
- Clear onboarding workflow with audit trail
- Moodle handles 100 concurrent users with < 3s page load (2× headroom above 50 baseline)

### Delighters (Excited — unspoken, wow factor)
- Automated backup verification reports sent to **internal ops team** (NOT client) via monitoring dashboard
- Proactive monitoring alerts before users notice degradation
- Self-service dashboard showing training completion rates
- Self-healing infrastructure (pod auto-restart) — users never notice the blip

**Note on delighters**: In IS214, delighters map to non-functional requirements (NFRs) — reliability, performance, availability, security — not visible UI features. Backend improvements count when stakeholders experience the result (fewer outages, faster response).

## CTQ (Critical to Quality) — Quantitative

Structure: Need → Driver → Measurable CTQ

| Need | Driver | CTQ |
|------|--------|-----|
| Reliable training platform | Availability during rollout | 99.5% uptime during Mon-Fri 9am-6pm SGT |
| Responsive training | Page load under load | P95 page load ≤ 3s at 100 concurrent sessions |
| Fast incident resolution | Response time | 95% of P2+ incidents acknowledged within 2 hours |
| Operational trust | Backup reliability | Daily backup verified, RPO ≤ 24 hours |
| Secure access | Onboarding compliance | 100% of Odoo accounts have verified Moodle completion |

## SLA

| Metric | Target | Measurement |
|--------|--------|-------------|
| Service Availability | 99.5% during business hours (Mon-Fri 8am-8pm SGT) | Azure Monitor / Better Stack synthetic monitoring |
| P1 Incident Response | 15 min ack, 2 hr resolve | Helpdesk ticket timestamps |
| Service Request Fulfillment | Moodle 4hrs, Odoo 24hrs post-completion | Helpdesk ticket timestamps |
| Backup RPO | 24 hours | Daily automated snapshot verification |
| Backup RTO | 4 hours | Documented rebuild runbook |

**Note**: Do NOT promise 24/7 99.5% — that allows only ~3.6 hours downtime/month and team has no on-call rotation. Scope to business hours.

## Non-Functional Change (PM3 — two RFCs)

### RFC 1: WAF + Nginx Reverse Proxy (Infrastructure-level, does NOT impact workflow)

**ITIL Lifecycle Loop**:
1. PM2 Incident → web defacement attack compromised production Odoo
2. Problem Management → RCA identified zero application-layer defenses (this is a CONTRIBUTING FACTOR, not root cause — root cause was unauthorized account with Settings-level privileges)
3. Change Management → RFC to implement WAF as preventive control
4. Continual Improvement → monitor blocked threats, validate control effectiveness

**What it includes**:
- Static asset caching (Nginx serves CSS/JS/images from memory)
- Gzip compression (~60-70% transfer size reduction)
- Rate limiting on login endpoints (brute-force protection)
- ModSecurity with OWASP CRS (SQL injection, XSS, directory traversal detection)

**Demo plan**: Before/after load test + attack simulation. Rollback via `kubectl rollout undo`. Show outcomes, not config steps.

### RFC 2: MFA Enforcement (Process-level, DOES impact workflow)

**Source**: Week 6 homework — listed as permanent fix with Medium risk (user friction), 2 weeks expected duration including change management.

**Workflow impact**: Adds step between "Odoo account created" and "staff can access Odoo" — staff must set up TOTP via authenticator app before account activates. This modifies the onboarding BPMN diagram.

**Why it's a separate RFC**: It's a normal change requiring CAB approval BECAUSE it introduces user friction and changes the access workflow. Infrastructure changes (WAF) and process changes (MFA) should not be bundled — different risk profiles, different rollback procedures.

### Other workflow-impacting change candidates (stretch)
- Dual-approval gate: Department Manager must also sign off before Odoo account creation (separation of duties)
- Periodic access recertification: Quarterly review, accounts not re-confirmed in 14 days suspended (Continual Improvement)

## Compliance & Security

- **Data residency**: All workloads in Southeast Asia (Azure region)
- **Internal-only access**: NSG rules deny all inbound except SSH from known IPs + HTTPS on ingress ports. Simulate via IP restriction to SMU range.
- **RBAC**: Azure RBAC for infrastructure, Kubernetes RBAC + namespaces for pod-level, Odoo role-based for application-level
- **Least privilege**: Namespace isolation means compromised Moodle pod can't touch Odoo resources
- **HTTPS**: TLS termination at Nginx Ingress Controller; internal pod-to-pod traffic HTTP within cluster network
- **Backups**: Daily PostgreSQL snapshots to Azure Managed Disk, verified via monitoring
- **Monitoring**: Azure Monitor + Better Stack for synthetic health checks + alerting
- **Audit**: All helpdesk tickets tracked to closure, all changes documented via RFC

## ITIL 4.0 Framing

### Service Value System (SVS) — the big picture
- **Guiding Principles**: Start where you are, progress iteratively, keep it simple, focus on value, think and work holistically, optimize and automate, collaborate and promote visibility
- **Governance**: RASCI matrix for escalation, Change Authority for RFCs, CTO approval for scope changes
- **Service Value Chain**: Six activities (below)
- **Practices**: Incident Management, Problem Management, Change Enablement, Service Request Management, Access Management, Monitoring and Event Management, Service Level Management
- **Continual Improvement**: PIR after incidents, monitoring WAF effectiveness, quarterly access recertification

### Service Value Chain (SVC) — the operating model
| Activity | How it maps |
|----------|-------------|
| Plan | SDD defines architecture, SLAs, compliance posture, cost controls |
| Engage | Healthcare CTO relationship, feedback loops, SLA negotiation |
| Design & Transition | AKS architecture, Docker images, Nginx config, RFC process |
| Obtain/Build | Azure provisioning, container image builds, CI/CD pipeline |
| Deliver & Support | Helpdesk operations, incident response, monitoring |
| Improve | PIR findings, WAF implementation, MFA proposal, access recertification |

### Four Dimensions (don't forget these)
| Dimension | Mapping |
|-----------|---------|
| Organizations & People | 5 team roles, RASCI, escalation paths |
| Information & Technology | AKS, Odoo, Moodle, PostgreSQL, Azure Monitor, Nginx |
| Partners & Suppliers | Azure (IaaS SLA dependency), Odoo SA (software), OWASP CRS (open-source security) |
| Value Streams & Processes | Onboarding workflow, incident flow, change RFC process |

## Key Kiru-isms (course-specific patterns that affect grading)

1. **BECAUSE reasoning**: Every decision must be justified with "X BECAUSE Y". Not "we chose AKS" but "we chose AKS BECAUSE..."
2. **Don't jump to root cause during identification**: During incident identification phase, describe what happened, not why. Root cause comes during Problem Management.
3. **Value is co-created, not delivered**: The client co-creates value by defining compliance requirements, participating in the access workflow, providing feedback. Service provision ≠ value creation.
4. **Monitor service quality, not just infrastructure**: LTA DataMall case study — system met uptime SLAs but failed customers because data quality wasn't monitored. Monitor what matters to users.
5. **ITIL v4, not v3**: GenAI often mixes them. v4 = SVS + SVC activities (non-sequential). v3 = sequential lifecycle. Never use v3 terminology.
6. **Spoken and unspoken requirements**: The project has both. Dissatisfiers are unspoken expectations. Missing them is worse than missing delighters.
7. **ESA mindset over Technical SA**: Business alignment > platform configuration. Justify from business need, not tech capability.

## Existing Infrastructure (from PM1/PM2)

- **Staging**: 20.244.119.185, user: is214
- **Production DNS**: `http://e08g08t05-prod.southeastasia.cloudapp.azure.com:8069`
- **Azure DevOps**: CI/CD pipelines with SSH service connections (ssh-odoo17, ssh-odoo17-prod)
- **Pipeline secrets**: STAGINGUSERPASSWORD, PRODUSERPASSWORD
- **Past incident**: Agatha Harkness (thanos@esmos.meals.sg) compromised account with Settings-level privileges → web defacement of ESMOS storefront
- **Incident tickets**: INC-001 through INC-005 sequence
- **Load testing**: Ramp-up tests at 20/50/100 VUs on Odoo (staging). No Moodle load testing done yet.

## Week 10 Presentation Structure (5 min + 5 min Q&A)

Suggested slides (graded):
1. Service overview & business context (1 slide)
2. Service strategy & ITIL 4 framing (1 slide) — SVC emphasis on Plan, anticipate later stages
3. Deployment approach & justification (1 slide) — AKS, Asia-Pacific compliance, business justification
4. Change strategy — forward-looking (1 slide) — conceptual, no implementation yet. RFC type, business value, risks, demo plan
5. Operations & support model (1 slide) — helpdesk, incident/change handling, success criteria
6. Cost, risk, and security considerations (1 slide)

**Key instruction**: This is a proposal to the CEO. Strategy and design, not implementation details. You can change the proposal after Week 10 — treat deviations as change management (communicate to CEO why, what business impact, get approval).

## Week 12 Final Presentation

- Live demo of project infrastructure and operations (technical walkthrough)
- Optional bonus: live demo of incident recovery
- All videos embedded in SharePoint document
- SDD delivered as SharePoint site (not just a Word doc)

## Tools & Constraints

- **Allowed**: Azure (primary), AWS Learner Lab or Alibaba Cloud (fallback), local hosting (more work), personal Azure free tier (requires credit card)
- **NOT allowed**: Paid tools. Free tier of freemium tools and open-source only.
- **Documentation**: SharePoint ONLY (enterprise compliance requirement — not Notion)
- **Load testing**: Required for 50 concurrent users. Tool choice is team's decision.
- **Moodle**: Open-source LMS. Use prebuilt Azure VMs or Docker images for quick deployment.
- **Helpdesk**: Must be compliant with data residency (self-hosted/IaaS in Asia Pacific, NOT SaaS)
- **Odoo Helpdesk guest portal**: Allowed — guest/portal submissions don't violate the "Odoo access only after training" requirement (that applies to operational user accounts)

## Open Risks & Gaps

1. **No Moodle load testing done** — CTQ claims 100 concurrent users but no evidence yet. Need Moodle-specific load test with PHP config tuning (pm.max_children, memory_limit). If HPA scales Moodle beyond 1 replica, need session sharing strategy (sticky sessions or Redis).
2. **AKS cost vs budget** — Must produce Azure Calculator screenshot showing monthly burn rate fits within credits for remaining project timeline.
3. **99.5% SLA measurement method undefined** — Synthetic monitoring (Better Stack) vs Azure Monitor Container Insights vs K8s liveness probe success rate give different numbers. Pick one, document it.
4. **Planned maintenance not excluded from SLA** — AKS node upgrades cause brief disruptions. Define maintenance windows or exclude from availability calculation.
5. **Non-functional change section ambiguity** — "Forward-looking — conceptual" vs listing specific implementation details. Decide clearly: is it a proposal (RFC document only) or are you implementing it (demo required)?
6. **Onboarding workflow has no exception paths** — Happy path only. Need timeout, backup assignee, dispute resolution.
7. **MFA on Odoo** — Haven't confirmed Odoo 17 supports TOTP natively or if it requires a module. Verify before committing.
