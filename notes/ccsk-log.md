# CCSK Progress Log (Running)

## Week 11 (Feb 23–Mar 1, 2026)

### Completed / in progress
- **CCSK v5 Domain 2:** Completed
  - Domain 2 Introduction
  - Unit 1: Cloud Governance
  - Unit 2: Effective Cloud Governance
  - Unit 3: The Governance Hierarchy
  - Unit 4: Key Strategies & Concepts
  - Domain 2 Conclusion

### Takeaways (plain language)
- Cloud governance is how you prevent “everyone does security differently” — it sets the rules, responsibilities, and guardrails so cloud usage stays safe and consistent.
- Effective governance uses layers (policies/standards → processes → technical enforcement) so controls are not just docs; they’re implemented and measurable.
- The higher the risk or sensitivity (especially for data), the more you need clear ownership, least-privilege access, and monitoring that proves controls are actually working.

## Week 10 — 2026-02-02 to 2026-02-08

### CCSK v5 — Domain 2: Cloud Governance

**Status**
- Completed:
  - Unit 1: Cloud Governance
  - Unit 2: Effective Cloud Governance  [oai_citation:1‡training.cloudsecurityalliance.org](https://training.cloudsecurityalliance.org/certificate-of-cloud-security-knowledge-v5?utm_source=chatgpt.com)

**3 takeaways (plain language)**
- Governance is how you ensure cloud use stays aligned with business goals *and* doesn’t quietly expand risk (cost, exposure, compliance gaps).  [oai_citation:2‡Cloud Security Alliance](https://cloudsecurityalliance.org/artifacts/ccsk-v5-curriculum?utm_source=chatgpt.com)
- “Effective governance” is operational: clear ownership, repeatable guardrails, and measurable controls—not just policies in a doc.  [oai_citation:3‡Cloud Security Alliance](https://cloudsecurityalliance.org/artifacts/ccsk-v5-curriculum?utm_source=chatgpt.com)
- Control frameworks (like CSA’s Cloud Controls Matrix) help translate “we need governance” into concrete control areas you can assess and improve over time.  [oai_citation:4‡Cloud Security Alliance](https://cloudsecurityalliance.org/research/cloud-controls-matrix?utm_source=chatgpt.com)

## Week 9 — CCSK (Jan 19–Jan 25, 2026)

### Progress (completed / in progress only)
- [x] Unit 2: Cloud Computing Models
- [x] Unit 3: Reference & Architecture Models
- [x] Unit 4: Cloud Security Scope, Responsibilities, & Models
- [x] Domain 1 Conclusion

### Key takeaways (3)
- Service and deployment models (IaaS/PaaS/SaaS; public/private/hybrid/multi-cloud) define your control surface: you trade control and visibility for speed and managed responsibility.
- Reference and architecture models (including overlapping/XaaS patterns) help you reason about trust boundaries and shared services so you don’t accidentally create “flat” architectures that are hard to secure.
- Shared responsibility remains the anchor: providers secure the underlying cloud, while customers must secure configuration, identity/access, data protection, and monitoring — and should follow security frameworks/patterns with a simple, repeatable security process.

## Week 8 — CCSK (Jan 12–Jan 18, 2026)

### Progress (completed / in progress only)
- [x] Domain 1 Introduction
- [x] Unit 1: Introduction to Cloud Computing

### Key takeaways (3)
- Cloud computing is defined by traits like on-demand self-service, broad network access, resource pooling, rapid elasticity, and measured service.
- The benefits (speed + elasticity + pay-for-usage) only pay off if you design and govern for them (right-sizing, visibility, and intentional configuration).
- Shared responsibility is the anchor: providers secure the underlying cloud infrastructure, while customers are responsible for secure configuration and access controls in their cloud environment.

## Week 2 — CCSK (Domains 2)

### Progress (completed / in progress only)
- [x] Domain 2: Governance, Risk, and Compliance (takeaways + lab mapping)

### Key takeaways (3)
- Effective cloud governance requires clear policies, defined roles, and explicit decision-making ownership, not ad hoc security decisions.
- Risk management is a continuous cycle (identify, assess, treat, monitor), so controls and operations need regular review instead of one-time setup.
- Compliance outcomes depend on demonstrable controls and evidence; in practice, cloud logging, monitoring, and documentation are as important as policy language.

## Week 1 — CCSK (Domains 1 & 5)

### Progress (completed / in progress only)
- [x] Domain 1: Cloud Computing Concepts and Architecture (applied to Week 1 AWS setup)
- [x] Domain 5: Identity and Access Management (applied to root/admin hardening and monitoring)

### Key takeaways (3)
- Shared responsibility and tenancy boundaries are foundational: providers secure cloud infrastructure while customers own identity, configuration, data protection, and monitoring in their account boundary.
- IAM is the primary control surface early on: protect high-privilege identities (root/admin), enforce MFA, avoid routine root use, and move toward least-privilege access patterns.
- Security needs to be designed in from day one; enabling CloudTrail, Config, GuardDuty, and Security Hub before workloads creates the visibility and audit baseline needed for a credible security program.
