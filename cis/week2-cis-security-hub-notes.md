# Week 2 - CIS Security Hub Notes

### CloudTrail → CloudWatch Logs integration

- CloudTrail trail: erick-cloudsec-trail (multi-Region, management events enabled).
- CloudWatch Logs log group: `aws-cloudtrail-logs-erick-cloudsec-lab`
- IAM role for CloudTrail→CloudWatch Logs:
  - `arn:aws:iam::912590423014:role/service-role/CloudTrailRoleForCloudWatchLogs_erick-cloudsec-trail`
- Purpose: Provide a log stream for CIS CloudWatch controls (CloudWatch.2 and CloudWatch.3) evaluated by AWS Security Hub.
