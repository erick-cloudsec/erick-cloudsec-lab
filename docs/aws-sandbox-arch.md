# AWS Sandbox Architecture (Week 1)

Scope: Personal AWS security lab in a single account, primary Region us-east-1.

Current components:
- Root user protected with MFA, no root access keys.
- Admin IAM user with MFA for everyday use.
- AWS CloudTrail multi-Region trail logging management events to S3.
- AWS Config recording all supported resource types in the Region.
- Amazon GuardDuty enabled for threat detection.
- AWS Security Hub enabled with CIS AWS Foundations Benchmark.

Planned next steps:
- Add multi-account structure for security/logging vs workloads.
- Deploy a 3-tier reference app and secure it.
- Build a DevSecOps pipeline and K8s security controls.
