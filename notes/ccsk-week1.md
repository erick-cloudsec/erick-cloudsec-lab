# CCSK Notes – Week 1 (Domains 1 & 5)

## Domain 1 – Cloud Computing Concepts and Architecture (high level notes)

- Shared responsibility model:
  - Cloud provider secures the infrastructure.
  - Customer is responsible for IAM, data protection, and configuration.
- This lab work focuses heavily on the **customer responsibilities**:
  - IAM setup (root vs. IAM user).
  - Logging and monitoring (CloudTrail, Config, GuardDuty, Security Hub).

## Domain 5 – Identity and Access Management

- Avoid using the root user for daily tasks:
  - Root is only for account-level tasks and emergency actions.
- Use IAM users/roles with least privilege:
  - Created an admin IAM user with MFA for console work.
- Strong authentication:
  - MFA on root user.
  - MFA on admin IAM user.
- Key idea: long-lived credentials (like root keys) are high risk and should be avoided.

## Mapping CCSK ideas to this lab

- IAM domain:
  - Strong separation between root and daily admin work.
  - MFA as a baseline control.

- Monitoring domain:
  - CloudTrail + AWS Config + GuardDuty + Security Hub provide observability into identity and configuration actions.
