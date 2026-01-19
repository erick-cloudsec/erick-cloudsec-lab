# AWS Lab Log

## Week 8 — SimuLearn Foundations (Jan 12–Jan 18, 2026)

### Completed
- AWS SimuLearn: Cloud First Steps — Completed (see evidence)
- AWS SimuLearn: Cloud Computing Essentials — Completed (see evidence)

### What I did (high level)
- Worked through two simulated customer scenarios and built the proposed solutions using structured, step-by-step lab guidance in the AWS Management Console (SimuLearn format).

### Key services / concepts touched
- Reliability & availability basics: redundancy across Availability Zones (AZs)
- Compute: EC2 (virtual servers)
- Storage / hosting: S3 static website hosting
- Access control: S3 bucket policy for public read (`s3:GetObject`) + exposure implications

### Evidence (files / links)
- Cloud First Steps:
  - evidence/aws-skillbuilder-solutions-architect/CloudFirstSteps/CloudFirstSteps-SimuLearn.png
  - evidence/aws-skillbuilder-solutions-architect/CloudFirstSteps/CloudFirstSteps-Lab.png
  - evidence/aws-skillbuilder-solutions-architect/CloudFirstSteps/cloud-first-steps-certificate.pdf
- Cloud Computing Essentials:
  - evidence/aws-skillbuilder-solutions-architect/CloudComputingEssentials/CloudComputingEssentials-Lab.png
  - evidence/aws-skillbuilder-solutions-architect/CloudComputingEssentials/cloud-computing-essentials-certificate.pdf

### What I learned (3–5 bullets)
- Making an S3 static website publicly reachable requires enabling static website hosting and explicitly allowing public read for site objects (e.g., `s3:GetObject`) via a bucket policy.
- Biggest risk with S3 static hosting is accidental over-exposure (policy scope too broad / wrong prefix / sensitive objects in the same bucket).
- Multi-AZ deployment is a foundational availability pattern; it reduces downtime risk from a single-AZ failure/outage.
- Shared Responsibility is the anchor: AWS secures the underlying cloud infrastructure; I’m responsible for service configuration and access controls (like bucket policies).

### Interview-ready bullets (Week 8 retention)
- Identity: S3 bucket policy (public read)
  Risk: unintended public exposure of objects/data
  Control: least-privilege bucket policy (only `s3:GetObject`, scoped to intended content); keep sensitive objects separate
  Verify: test anonymous access only for intended paths; review bucket policy / public exposure settings
  Evidence: screenshots + certificate (see Evidence section)

- Identity: EC2 workload placement across AZs
  Risk: single-AZ outage causes downtime
  Control: deploy redundant compute across multiple AZs
  Verify: validate service remains reachable during failure scenario
  Evidence: screenshots + certificate (see Evidence section)

### Next up (per learning plan order)
- AWS SimuLearn: Networking Concepts

## 2026-01-02 – Unauthorized API calls alarm (CIS CloudWatch.2)

### What I set up (in plain language)
I set up monitoring so that when AWS records a **permission-denied API call** in CloudTrail, CloudWatch will **count it**, **raise an alarm**, and **notify me** (via SNS). This aligns with CIS CloudWatch.2 (“Ensure a log metric filter and alarm exist for unauthorized API calls”).

### Why I set it up
Unauthorized API calls can indicate:
- a harmless misconfiguration (least privilege too tight), or
- early signs of malicious probing / credential misuse.

### What I configured (specifics)
**Log source**
- CloudTrail management events routed to CloudWatch Logs log group: `aws-cloudtrail-logs-erick-cloudsec-lab` (us-east-1)

**Metric filter → custom metric**
- Namespace: `CISBenchmark`
- Metric name: `UnauthorizedAPICalls`
- Metric value: `1`
- Filter pattern:
  { ( $.errorCode = "*UnauthorizedOperation*" || $.errorCode = "AccessDenied*" )
    && ( $.userIdentity.sessionContext.sessionIssuer.userName != "AWSServiceRoleForConfig" )
    && ( $.userIdentity.sessionContext.sessionIssuer.userName != "AWSServiceRoleForResourceExplorer" ) }

Notes:
- Used wildcards to catch variants like `Client.UnauthorizedOperation` as well as `AccessDenied*`.
- Excluded common “expected noise” service roles for Config and Resource Explorer.

**Alarm**
- Name: `Alarm-Unauthorized-API-Calls`
- Condition: `UnauthorizedAPICalls >= 1` for 1 datapoint within 5 minutes
- Action: SNS topic `lab-security-alerts`
- Expected behavior: alarms can show `INSUFFICIENT_DATA` until the first matching datapoints exist.

### How I verified it works (safe test)
1) Assumed a role with **no identity-based permissions**: `ErickDenyTestRole`
2) Triggered a harmless, read-only denied call: `ec2:DescribeInstances` via AWS CLI in CloudShell
3) Observed:
   - CloudTrail event generated with an unauthorized error
   - Metric filter matched and emitted a datapoint
   - Alarm transitioned from `INSUFFICIENT_DATA` → `ALARM`
   - SNS action executed successfully

### Golden CloudTrail event (proof details)
- Time (UTC): `2026-01-02T03:51:58Z`
- Actor: `AssumedRole` — `arn:aws:sts::912590423014:assumed-role/ErickDenyTestRole/denytest`
- Issuer role: `arn:aws:iam::912590423014:role/ErickDenyTestRole` (MFA: true)
- API: `ec2.amazonaws.com : DescribeInstances` (readOnly: true)
- Result: `errorCode = Client.UnauthorizedOperation`
- Source: `3.89.163.64` | AWS CLI via CloudShell (userAgent shows `exec-env/CloudShell`)
- Correlation: `eventID=f7a276de-9ab0-4463-baf8-171c3f2b014f`, `requestID=1efeb0a5-1baf-4270-b6e9-9a09354b1a50`

### Evidence (screenshots)
Stored in: `Week7/`
- Metric filter configuration (pattern + metric namespace/name/value)
- Alarm configuration + graph (showing ALARM) + Alarm History (showing `INSUFFICIENT_DATA → ALARM` and SNS action success)
- CloudTrail event details (expanded view showing `userIdentity`, `eventName`, `errorCode`, `sourceIPAddress`)

### Tiny runbook (3 steps)
1) **Identify** actor + scope: `userIdentity.*`, `eventSource`, `eventName`, `sourceIPAddress`, time window.
2) **Validate**: misconfig vs suspicious probing (repeat denied calls, unusual API mix, new source IP/user agent).
3) **Contain + fix**: if suspicious, cut off the credential path (restrict role assumption/credentials), tighten IAM policy, document root cause + prevention.

### Interview bullets (identity → risk → control → verification → evidence)
- **Identity:** Used CloudTrail identity context to attribute denied API activity to a specific assumed-role session.
- **Risk:** Unauthorized API calls can signal mis-scoped permissions or credential misuse; monitoring reduces time-to-detect.
- **Control:** Implemented CIS CloudWatch.2 using CloudTrail → CloudWatch Logs → metric filter → alarm → SNS.
- **Verification:** Safely triggered a denied call via a no-permissions role and confirmed datapoint emission + alarm/SNS execution.
- **Evidence:** Screenshots of the filter, alarm history/state, and the CloudTrail event showing `Client.UnauthorizedOperation`.

## 2025-12-26 – Root hardware MFA hardening (CIS IAM.6)

### What I set up (in plain language)

In my erick-cloudsec lab AWS account, I remediated a Critical failing CIS/Security Hub posture control: IAM.6 – Hardware MFA should be enabled for the root user.

This control isn’t just “root has MFA.” It specifically checks that the account is enabled to sign in with **root credentials using hardware-backed MFA**, and it fails if **virtual MFA devices are permitted** for root sign-in.

I started from Security Hub CSPM → CIS AWS Foundations Benchmark (v5.x) and used the failing control as my “work item.” I already had MFA on root, but the control was failing because a **virtual MFA device** was still permitted.

After switching root MFA to hardware-backed methods only (passkeys) and removing the virtual MFA device, the IAM.6 check now shows PASSED (and the finding workflow status is RESOLVED, which Security Hub applies automatically when a control finding becomes PASSED).

Note: the top-level “Control status” banner can lag the latest finding update because Security Hub rolls up control status based on findings from the previous 24 hours; the Checks row is the best proof of the most recent evaluation.

———

### Asset I’m protecting

- AWS account “break-glass” access (root credentials):
  - Preventing unauthorized takeover of the most privileged identity
  - Ensuring root access requires hardware-backed MFA
  - Avoiding a single-point-of-failure by keeping multiple MFA devices registered

———

### Threat I care about here

- Account takeover: if root credentials are compromised (password theft, phishing, credential stuffing), an attacker can gain unrestricted control of the account.
- Persistence and cleanup risk: an attacker with root can create backdoor identities, weaken security controls, and make incident response harder.
- Operational risk: losing access to the only MFA device can cause access interruption during a true emergency (“break-glass” moment).

———

### Detection / control details (the actual config)

#### Detection source (posture finding)

- Service: AWS Security Hub CSPM
- Standard: CIS AWS Foundations Benchmark (v5.x)
- Control: IAM.6
- Region / account: us-east-1 / 912590423014

Security Hub generates and updates control findings on a schedule and summarizes control status based on findings in the prior 24 hours.

#### Remediation (Root MFA configuration)

On the root user “My security credentials” page:

- Confirmed hardware-backed MFA is registered for root:
  - Passkeys / security keys (Touch ID / FIDO2)
- Removed the virtual MFA device previously registered for root (authenticator/TOTP), because IAM.6 fails if virtual MFA is permitted.
- Kept two hardware-backed methods registered (resilience), since AWS supports up to 8 MFA devices per root user.

#### Verification

- AWS Config:
  - Managed rule: `root-account-hardware-mfa-enabled`
  - Current evaluation: COMPLIANT
- Security Hub CSPM:
  - IAM.6 control Checks row shows PASSED and the finding workflow is RESOLVED

———

### How I expect to respond when this fails (or regresses)

If IAM.6 goes back to FAILED:

1. Confirm what MFA device types are registered under root “My security credentials.”
2. Ensure only hardware-backed options are permitted for root sign-in (passkeys/security keys/hardware device) and that no virtual MFA devices are present for root.
3. Re-check AWS Config rule `root-account-hardware-mfa-enabled` evaluation, then refresh Security Hub after the next update window.

———

### How this maps to frameworks and interview narrative

- CIS AWS Foundations Benchmark (v5.x)
  - Directly addresses IAM.6 expectations for hardening root access with hardware-backed MFA.
- AWS Security Hub CSPM
  - Demonstrates a posture-driven workflow: identify a critical gap → remediate in IAM → verify via AWS Config + Security Hub finding state.
- “Architect story” (risk narrative)
  - “I treated root as break-glass and enforced phishing-resistant MFA. I used CSPM to find the gap, removed virtual MFA permission for root, and verified compliance through Config and Security Hub.”

———

### Evidence captured

- Screenshot: Root “My security credentials” → MFA devices list showing only “Passkeys and security keys”
- Screenshot: AWS Config rule `root-account-hardware-mfa-enabled` showing COMPLIANT
- Screenshot: Security Hub CSPM → IAM.6 Checks row showing PASSED / RESOLVED

———

### References (for my notes)

- CIS benchmark in Security Hub (control list): https://docs.aws.amazon.com/securityhub/latest/userguide/cis-aws-foundations-benchmark.html
- IAM controls / IAM.6 definition: https://docs.aws.amazon.com/securityhub/latest/userguide/iam-controls.html
- AWS Config rule: root-account-hardware-mfa-enabled: https://docs.aws.amazon.com/config/latest/developerguide/root-account-hardware-mfa-enabled.html
- Enable passkey/security key for root: https://docs.aws.amazon.com/IAM/latest/UserGuide/enable-fido-mfa-for-root.html
- Control status roll-up behavior: https://docs.aws.amazon.com/securityhub/latest/userguide/controls-overall-status.html
- Control details (Checks row / past 24 hours): https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-control-details.html

Sources: Security Hub CIS benchmark control list and IAM controls (IAM.6), AWS Config managed rule behavior, root passkey/security key setup steps, and Security Hub control-status update cadence.  ￼

## 2025-12-21 – AWS Config recording foundation (CIS Config.1)

### What I set up (in plain language)

In my **erick-cloudsec** lab AWS account, I remediated a **Critical** failing CIS/Security Hub posture control: **Config.1 – AWS Config should be enabled and use the service-linked role for resource recording**.

I started from **Security Hub CSPM → CIS AWS Foundations Benchmark v5.0.0** and used the failing control as my “work item.” The key issue wasn’t just “Config is on” — it was that AWS Config must:
- be enabled in the current Region,
- use the **AWS Config service-linked role** (`AWSServiceRoleForConfig`), and
- record the resource types required by the Security Hub controls enabled in that Region (including IAM global resource types like users/roles/policies/groups).  
(That’s exactly what Config.1 checks.)  
(Ref: Security Hub CSPM Config controls docs.)  [oai_citation:0‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/config-controls.html?utm_source=chatgpt.com)

After updating the AWS Config recorder settings, the **Config.1 check** now shows **PASSED** (and the finding workflow status is **RESOLVED**, which Security Hub sets automatically when a control finding becomes PASSED).  [oai_citation:1‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/controls-overall-status.html?utm_source=chatgpt.com)

Note: the top-level “Control status” banner can lag the latest finding update; the **Checks** row is the best proof of the current evaluation.  [oai_citation:2‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-control-details.html?utm_source=chatgpt.com)

---

### Asset I’m protecting

- **Governance and visibility** for the AWS account:
  - A trustworthy configuration inventory
  - Change history for resources
  - The ability for posture controls (CIS / AWS FSBP) to evaluate resource configurations consistently

AWS Config is a foundational service for audit/compliance and ongoing configuration monitoring.  [oai_citation:3‡Amazon Web Services, Inc.](https://aws.amazon.com/blogs/mt/aws-config-best-practices/?utm_source=chatgpt.com)

---

### Threat I care about here

- **Blind spots**: if AWS Config isn’t recording the right resource types, security posture tooling can’t reliably detect drift or misconfiguration.
- **Weak auditability**: limited or missing configuration history makes incident response and compliance evidence harder.
- **Control dependency failure**: many Security Hub controls rely on AWS Config recording to evaluate resources.

---

### Detection / control details (the actual config)

#### Detection source (posture finding)

- Service: **AWS Security Hub CSPM**
- Standards enabled:
  - **AWS Foundational Security Best Practices v1.0.0**
  - **CIS AWS Foundations Benchmark v5.0.0**
- Control: **Config.1**
- Region / account: `us-east-1 / 912590423014`

Security Hub generates/updates control findings on a schedule (typically every 12–24 hours) and lists active findings for the past 24 hours on the control’s **Checks** tab.  [oai_citation:4‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-control-details.html?utm_source=chatgpt.com)

#### Remediation (AWS Config)

In **AWS Config → Settings** (us-east-1), I ensured the account has an active recorder that satisfies Config.1:

- **Customer managed recorder**: enabled (Recording is on)
- **IAM role**: **AWS Config service-linked role** (`AWSServiceRoleForConfig`)
  - Service-linked roles are predefined by AWS and include the permissions AWS Config requires.  [oai_citation:5‡AWS Documentation](https://docs.aws.amazon.com/config/latest/developerguide/using-service-linked-roles.html?utm_source=chatgpt.com)
- **Recording scope**: records the resource types required for enabled Security Hub controls in this Region (including IAM global resource types such as users/roles/policies/groups).  [oai_citation:6‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/config-controls.html?utm_source=chatgpt.com)
- **Delivery**: S3 bucket configured for AWS Config delivery (already present in this lab)

#### Verification

- In the **Config.1 control view**, the **Checks** row shows:
  - Compliance: **PASSED**
  - Workflow: **RESOLVED** (set automatically by Security Hub when compliance becomes PASSED)  [oai_citation:7‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/controls-overall-status.html?utm_source=chatgpt.com)

---

### How I expect to respond when this fails (or regresses)

If Config.1 goes back to FAILED:

1. Open the Config.1 control and inspect the latest finding JSON (reason codes and “missing required resource types” are usually explicit).
2. In AWS Config, verify:
   - recorder is on in the Region,
   - service-linked role is in use,
   - required resource types are being recorded (especially IAM global types if relevant controls are enabled).
3. Re-check Security Hub after the next update window (control findings refresh on a schedule).  [oai_citation:8‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/automation-rules.html?utm_source=chatgpt.com)

---

### How this maps to frameworks and interview narrative

- CIS AWS Foundations Benchmark v5.0.0
  - Directly addresses **Config.1** expectations for posture evidence and configuration recording.
- AWS Security Hub CSPM
  - Demonstrates using posture tooling to identify a high-severity gap, remediate it, and verify via the control finding state.  [oai_citation:9‡AWS Documentation](https://docs.aws.amazon.com/securityhub/latest/userguide/config-controls.html?utm_source=chatgpt.com)
- “Architect story” (risk narrative)
  - “I fixed a foundational control that posture management depends on: reliable configuration recording with the correct role and required resource scope.”

---

### Evidence captured

- Screenshot: Security Hub CSPM → Config.1 **Checks row showing PASSED / RESOLVED**
- Screenshot: AWS Config recorder settings showing:
  - Recording is on
  - Service-linked role (`AWSServiceRoleForConfig`) selected
  - Recording scope includes required resource types

---

### Future improvements

- Enable/validate AWS Config recording across additional Regions (CIS recommends “all Regions”).
- Express AWS Config recorder setup as IaC (Terraform/CloudFormation) so the control is repeatable and reviewable as code.
- Add a small “posture runbook” checklist: when a CSPM control fails, capture the reason code, remediate, and keep screenshots + a short narrative for evidence.

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
