# AWS Lab Log

## 2025-12-10 – Unauthorized API calls detection (CIS CloudWatch.2)

### What I set up (in plain language)

In my **erick-cloudsec** lab AWS account, I refined an existing CloudWatch alarm so it properly implements the CIS AWS Foundations Benchmark control for **unauthorized API calls** (CloudWatch.2).

CloudTrail is already logging management API activity. When CloudTrail records API calls that fail with `UnauthorizedOperation` or `AccessDenied` (excluding a couple of noisy AWS service roles), those events are counted by a custom metric. I updated the alarm on that metric so it raises a clear, documented alert when there are too many unauthorized calls in a short period.

This control is meant to catch both:
- Misconfigurations (for example, an app or Lambda role that’s missing permissions and keeps failing), and  
- Potential attacker behavior (for example, someone probing which APIs they can call in order to escalate privileges).

On **2025-12-10**, I treated this as a “refine an existing detection” exercise: confirm the config, align it with CIS CloudWatch.2, and write down how to respond when it fires.

---

### Asset I’m protecting

- The proper and authorized use of AWS APIs and resources in the **erick-cloudsec** lab account:
  - IAM users and roles  
  - The APIs they are supposed (and not supposed) to call  
  - Early warning when something keeps trying APIs it isn’t allowed to use

---

### Threat I care about here

Patterns where there is a **burst of unauthorized API calls**, such as:

- A misconfigured application, Lambda, or automation role repeatedly calling an API it doesn’t have permission for.
- A human or script (possibly an attacker) doing recon or privilege escalation probing, trying different APIs and getting back `UnauthorizedOperation` or `AccessDenied` many times in a short window.

These are called out in CIS AWS Foundations and AWS Security Hub’s CIS checks as important to detect, because they often show up before successful misuse of access.

---

### Detection / control details (the actual config)

#### Log source

- CloudTrail is configured in this account to capture management events.
- CloudTrail delivers these events to a CloudWatch Logs log group, which is used as the source for metric filters and alarms.

#### Metric filter (CIS-style, unchanged)

- Filter name: `unauthorized_api_calls_metric`
- Filter pattern:

    { ( $.errorCode = "UnauthorizedOperation" || $.errorCode = "AccessDenied" ) &&
      ( $.userIdentity.sessionIssuer.userName != "AWSServiceRoleForConfig" ) &&
      ( $.userIdentity.sessionIssuer.userName != "AWSServiceRoleForResourceExplorer" ) }

- What this pattern does:
  - Matches CloudTrail events where an API call failed because:
    - The caller is not authorized to perform the operation (`UnauthorizedOperation`), or
    - The call was denied in general (`AccessDenied`).
  - Filters out two known noisy AWS service roles:
    - `AWSServiceRoleForConfig`
    - `AWSServiceRoleForResourceExplorer`
- Metric namespace: `CISBenchmark`
- Metric name: `UnauthorizedAPICalls`
- Metric value: `1` per matching log event

So the metric `CISBenchmark / UnauthorizedAPICalls` is a count of unauthorized API failures over each CloudWatch period.

#### CloudWatch alarm (refined on 2025-12-10)

On 2025-12-10, I reviewed and updated the existing alarm on this metric to make sure it is clearly documented and aligned to the CIS control.

- Alarm name: `Alarm-Unauthorized-API-Calls`
- Type: Metric alarm
- Description (updated):  
  Implements CIS AWS Foundations CloudWatch.2 by raising an alarm when CloudTrail logs show unauthorized API calls (`UnauthorizedOperation` or `AccessDenied`) in the erick-cloudsec lab account.
- Metric: `CISBenchmark / UnauthorizedAPICalls`
- Statistic: `Sum`
- Period: `5` minutes
- Threshold: Alarm when `UnauthorizedAPICalls >= 5` for `1` datapoint within `5` minutes
- Datapoints to alarm: `1 out of 1`
- Missing data treatment: `Treat missing data as missing`
- Region / account: `us-east-1 / 912590423014`
- Alarm ARN:  
  `arn:aws:cloudwatch:us-east-1:912590423014:alarm:Alarm-Unauthorized-API-Calls`
- Actions:
  - Actions are enabled.
  - Alarm sends a notification via an SNS topic (email) configured for security alerts in this account.

Note: This alarm originally existed (last state update shown as 2025-11-27), but on 2025-12-10 I treated it as a “refine an existing detection” exercise—confirming the metric, threshold, and description, and documenting the control narrative below.

---

### How I expect to respond when this fires

If `Alarm-Unauthorized-API-Calls` goes into ALARM:

1. Review the alarm and recent events  
   - Open the alarm in CloudWatch → Alarms.  
   - From the alarm, view the metric graph and drill down to the underlying CloudTrail events in the CloudWatch Logs log group.

2. Inspect the matching CloudTrail entries  
   - Which IAM principal (user or role) generated the unauthorized calls?  
   - Which APIs and resources were they trying to access?  
   - What is the source IP, user agent, and time window?

3. Classify the cause  
   - If it looks like a misconfiguration (for example, new deployment with missing permissions, wrong role attached):  
     - Fix the IAM policy or application configuration.  
     - Re-run the workload and confirm that unauthorized calls drop back to normal levels.  
   - If it looks suspicious (unexpected principal, unusual IP, odd API surface or volume):  
     - Revoke or rotate access keys or credentials for the principal.  
     - Tighten the principal’s IAM policies to least privilege.  
     - Check for related findings in GuardDuty, Security Hub, or other logs.  
     - Consider temporarily disabling the principal or limiting access while investigating.

4. Tune instead of disable  
   - If the alarm fires too frequently in this lab environment:  
     - Increase the threshold (for example, require more than 5 events), or  
     - Increase the evaluation period (for example, 10–15 minutes) so only meaningful spikes cause alarms.  
   - Avoid disabling the alarm unless it is replaced by a better control.

---

### How this maps to frameworks and attacker behavior

- CIS AWS Foundations Benchmark  
  - Directly maps to **CloudWatch.2 – Ensure a log metric filter and alarm exist for unauthorized API calls**.
- AWS Security Hub  
  - Aligns with the Security Hub CIS checks that expect CloudTrail → CloudWatch Logs → metric filter + alarm for unauthorized API activity.
- CSA Top Threats / attacker techniques  
  - Helps detect:
    - IAM misconfigurations and misuse of access (for example, broken roles or apps).  
    - Early signs of attackers or malicious scripts probing which permissions they have by repeatedly calling unauthorized APIs (recon / privilege escalation).

---

### Future improvements

- Express this metric filter and alarm in Terraform or CloudFormation so:
  - Every lab or production account can enable this control consistently.
  - Changes can be reviewed as code and drift can be detected.
- Add automation around the alarm:
  - An EventBridge rule or Lambda that enriches events or tags high-risk principals.
  - Integration with a ticketing system or on-call channel so incidents are tracked and not just emailed.
