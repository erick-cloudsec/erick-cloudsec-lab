# Framework Mapping – Week 1

This lab is aligned to:

- **AWS Well-Architected – Security Pillar**
- **CIS AWS Foundations Benchmark**
- **CSA Security Guidance / CCSK domains (IAM & Monitoring)**
- **OWASP ASVS / SAMM** at the platform level

## AWS Well-Architected Security Pillar

- Strong identity foundation:
  - Root MFA enabled, no root access keys.
  - Admin IAM user with MFA for daily work.

- Maintain traceability:
  - CloudTrail multi-Region trail logging management events to S3.
  - AWS Config recording all supported resource types in the Region.

## CIS AWS Foundations – Week 1 highlights

- CloudTrail.1 – Multi-Region CloudTrail enabled with read/write management events.
- Config.1 – AWS Config enabled using a service-linked role.
- IAM.4 – No root user access keys.
- IAM.9 – MFA enabled for the root user.

## CSA CCSK – Early alignment

- IAM domain:
  - Strong separation of root and daily admin usage.
- Monitoring/logging domain:
  - Centralized logging via CloudTrail + Config as a basis for detection and audit.

## OWASP ASVS / SAMM (Platform View)

- ASVS: Foundational logging and monitoring support for future workloads.
- SAMM: Early “Secure Architecture” practice – defining a baseline AWS security architecture before workloads.
