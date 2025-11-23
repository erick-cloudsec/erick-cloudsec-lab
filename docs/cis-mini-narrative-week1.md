# CIS AWS Foundations – Week 1 Control Narrative

## Scope

Personal AWS account used as a Cloud/AppSec lab (single account, primary Region us-east-1).

## Objective

Establish a basic AWS security baseline aligned with key CIS AWS Foundations controls:
- Logging and monitoring of account activity.
- Protecting the root account.
- Enabling configuration recording for future posture checks.

## Implementation (Week 1)

- Root user:
  - MFA enabled for the root account.
  - No root access keys present.

- Identity:
  - Created an admin IAM user with MFA for daily console work.
  - Stopped using root for day-to-day tasks.

- Logging:
  - Configured a multi-Region AWS CloudTrail trail to log read and write management events.
  - Delivered CloudTrail logs to a dedicated S3 bucket.

- Configuration tracking:
  - Enabled AWS Config in the primary Region.
  - Recording all supported resource types with continuous recording.
  - Using the AWS Config service-linked role and a dedicated S3 bucket.

- Detection:
  - Enabled Amazon GuardDuty in the primary Region.

- Posture management:
  - Enabled AWS Security Hub.
  - Enabled the CIS AWS Foundations Benchmark standard.

## Evidence (stored in `evidence/week1/`)

- Screenshot – Root user security page showing MFA enabled.
- Screenshot – IAM user list showing the admin IAM user with MFA.
- Screenshot – CloudTrail Trails page showing multi-Region trail.
- Screenshot – AWS Config Settings showing recorder status = Recording.
- Screenshot – GuardDuty dashboard showing service enabled.
- Screenshot – Security Hub security standards page showing CIS AWS Foundations enabled.

## Planned Improvements

- Enable CloudTrail log file validation and harden S3 log buckets.
- Add additional CIS-related controls as resources are created (for S3, EC2, VPC, RDS, etc.).
- Integrate Security Hub and GuardDuty findings into a simple incident response workflow.
