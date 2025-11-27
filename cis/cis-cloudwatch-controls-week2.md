# CIS CloudWatch Controls - Week 2

## CloudWatch.2 - Unauthorized API call metric filter

- Metric filter: `unauthorized_api_calls_metric`
  - Log group: `aws-cloudtrail-logs-erick-cloudsec-lab`
  - Pattern: {($.errorCode="UnauthorizedOperation") || ($.errorCode="AccessDenied")}
  - Metric: CISBenchmark / UnauthorizedAPICalls

### CloudWatch.2 - Unauthorized API calls (status update)

- Implemented a CloudWatch Logs metric filter on `aws-cloudtrail-logs-erick-cloudsec-lab`:
  - Filter name: `unauthorized_api_calls_metric`
  - Pattern: {($.errorCode="UnauthorizedOperation") || ($.errorCode="AccessDenied")}
  - Metric: `CISBenchmark / UnauthorizedAPICalls` (value = 1 per matching event)
- Created a CloudWatch alarm:
  - Name: `Alarm-Unauthorized-API-Calls`
  - Condition: Sum of `UnauthorizedAPICalls` >= 1 over a 5-minute period (1 evaluation period)
  - Action: Sends notification to SNS topic `lab-security-alerts` (email)
- Current alarm state: `INSUFFICIENT_DATA` immediately after creation, which is expected until CloudWatch has at least one full evaluation period of metric data (or a matching unauthorized API event).
- Next step: Either let the alarm naturally move to `OK` as it sees empty periods, or generate a controlled AccessDenied/UnauthorizedOperation event to confirm it transitions to `ALARM` and sends an email notification.

#### Tuning – Alarm-Unauthorized-API-Calls

- Initial behavior:
  - Alarm fired on benign AccessDenied events from AWS service-linked roles:
    - AWSServiceRoleForConfig calling Inspector2 (GetDelegatedAdminAccount)
    - AWSServiceRoleForResourceExplorer calling Fraud Detector (GetExternalModels, etc.).
- Updated metric filter:
  - Still matches errorCode = UnauthorizedOperation or AccessDenied.
  - Now excludes events where:
    - userIdentity.sessionIssuer.userName = AWSServiceRoleForConfig
    - userIdentity.sessionIssuer.userName = AWSServiceRoleForResourceExplorer
- Rationale:
  - These events are expected AWS service plumbing in this lab and do not represent attacker behavior
    or misconfigured human/automation principals.

### CloudWatch.3 - Console sign-in without MFA

- Log group: `aws-cloudtrail-logs-erick-cloudsec-lab`
- Metric filter: `console_signin_no_mfa_metric`
  - Pattern: {($.eventName="ConsoleLogin") && ($.additionalEventData.MFAUsed!="Yes") && ($.responseElements.ConsoleLogin="Success")}
  - Metric: CISBenchmark / ConsoleSigninNoMFA
- Alarm: `Alarm-ConsoleSignin-NoMFA`
  - Condition: Sum >= 1 over 5 minutes (1 evaluation period)
  - Action: SNS topic `lab-security-alerts` (email)
- Evidence:
  - `cloudwatch-console-no-mfa-metric.png`
  - `cloudwatch-console-no-mfa-alarm.png`
