# CCSK Notes – Week 1 (Domains 1 & 5)

This file captures key ideas from CSA Security Guidance v5 that I applied in my Week 1 AWS lab work.

---

## Domain 1 – Cloud Computing Concepts and Architecture

**Key ideas I’m taking away**

- **Shared responsibility model**  
  - Cloud providers secure the underlying infrastructure and managed services.  
  - Customers are responsible for identity and access, resource configuration, data protection, and monitoring.

- **Abstraction and service models matter (IaaS / PaaS / SaaS)**  
  - The more managed the service, the smaller my configuration surface—but I never fully escape responsibility for identity, data, and access policies.  
  - In this lab, I’m operating mostly at the IaaS/PaaS level in AWS.

- **Account and tenancy boundaries are security boundaries**  
  - An AWS account is a strong isolation boundary.  
  - Over time, I’ll extend this lab toward a multi-account structure (e.g., security/logging account vs. workload accounts).

- **Security needs to be designed in, not bolted on**  
  - Identity, logging, and monitoring are “base layers,” not nice-to-haves added after workloads exist.  
  - Week 1 is intentionally about building that base before deploying any apps.

**How Domain 1 maps to my Week 1 lab**

- I treated the AWS account as **my responsibility zone**, and focused on:
  - Locking down the root account.
  - Creating a named admin IAM user.
  - Enabling CloudTrail, Config, GuardDuty, and Security Hub for traceability and visibility.
- I started with **security services before workloads**, consistent with the idea that cloud security controls should be part of the architecture, not an afterthought.
- I’m using this lab to practice the “customer responsibility” side of cloud security in a realistic way, not just as theory.

---

## Domain 5 – Identity and Access Management (IAM)

**Key ideas I’m taking away**

- **Root accounts are high-risk and should be rarely used**
  - Root has power that cannot be constrained by IAM policies.
  - Root credentials must be strongly protected and not used for daily operations.

- **Strong authentication for sensitive identities**
  - Multi-factor authentication (MFA) is a baseline control for root and admin identities.
  - Long-lived static credentials (especially for root) are a major risk.

- **Least privilege and role-based access**
  - Access should be granted based on roles and tasks, not convenience.
  - Over time, identity should move toward roles and federation instead of individual static users and keys.

- **Centralized management and monitoring of IAM**
  - IAM changes (users, roles, policies, keys) should be logged and monitored.
  - IAM design and monitoring are core to cloud security, not a side topic.

**How Domain 5 maps to my Week 1 lab**

- **Root user controls**
  - Enabled MFA on the AWS root account.
  - Ensured there are **no** root access keys.
  - Stopped using root for any day-to-day activity.

- **Admin IAM user**
  - Created a named admin IAM user with MFA for all console and administrative work.
  - This user is now my primary way to interact with the account, which aligns with CCSK’s guidance to avoid root for routine tasks.

- **Monitoring IAM-related activity**
  - Enabled a multi-Region CloudTrail trail logging management events, so IAM and account-level actions are captured.
  - Enabled AWS Config to record resource configuration, which supports monitoring and auditing of IAM-related resources over time.
  - Enabled GuardDuty and Security Hub, which consume CloudTrail/Config and surface suspicious behavior and misconfigurations, including IAM-related issues.

---

## How this lab applies CCSK so far (Week 1 summary)

- I implemented a **clear separation** between the high-risk root account and a named admin IAM user, with MFA enforced on both.
- I treated **logging and monitoring (CloudTrail, Config, GuardDuty, Security Hub)** as mandatory parts of my cloud security posture, consistent with CCSK’s view of customer responsibilities.
- I aligned my early work with CCSK’s emphasis on:
  - Identity as a primary control surface.
  - Strong authentication and protection of powerful identities.
  - Visibility into account activity and configuration as a prerequisite for any serious cloud security program.
