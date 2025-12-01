# Weekly Retro Template – Cloud/AppSec Lab

Use this at the end of each week, in a 15-minute block.

1. What did I ship this week?
2. What moved me closer to Cloud/AppSec Architect interviews?
3. What felt heavy or draining?
4. What will I do less of next week?
5. One win to celebrate.
6. Top 1–2 priorities for next week.

## Week 2 (Mon Nov 24 – Sun Nov 30, 2025)

**Q1. What did I actually get done?**
- Implemented CIS-style CloudWatch.2 and CloudWatch.3 controls in the lab:
  - Metric filters + alarms for Unauthorized API calls and Console sign-in without MFA.
  - Wired both alarms to the `lab-security-alerts` SNS topic.
- Investigated multiple `AccessDenied` events using CloudTrail CSV export and CloudWatch Logs Insights:
  - Found that many alerts were from AWS service-linked roles (Config, Resource Explorer 2) calling Inspector2, Macie, and Fraud Detector discovery APIs.
- Tuned the `UnauthorizedAPICalls` metric filter:
  - Still matches `UnauthorizedOperation` / `AccessDenied`, but now excludes events from:
    - `AWSServiceRoleForConfig`
    - `AWSServiceRoleForResourceExplorer`
- OWASP:
  - Explored OWASP Cheat Sheet Series issues with the `HELP_WANTED` label and confirmed many were already taken or blocked.
  - Posted in OWASP `#contribute` channel describing my background and asking for a scoped 4–6 hour starter task.
  - Chose **OWASP Nest** as my first contribution project, focusing on cleaning up API documentation.
  - Spent ~1 hour getting a local Nest environment running and viewing the OpenAPI version of the APIs (instead of reading Django Ninja code directly).
- CCSK:
  - Read CSA Security Guidance v5, Domain 2 (Governance, Risk, and Compliance).
  - Added Domain 2 notes to my CCSK notes file.

**Q2. What moved me closer to Cloud/AppSec Architect interviews?**
- Built and documented realistic CloudWatch detections (CloudWatch.2 and CloudWatch.3) that map cleanly to CIS-style requirements and AWS best practices.
- Practiced “real” detection engineering:
  - Used CloudTrail and Logs Insights to understand why alarms fired.
  - Distinguished benign AWS service plumbing from potentially interesting unauthorized activity.
- Captured tuning decisions (what I excluded and why) in the repo, which reads like the beginning of an actual detection-engineering portfolio piece.
- Took concrete steps toward an **OWASP contribution portfolio artifact**:
  - Selected a specific project (OWASP Nest).
  - Got the local app and OpenAPI docs view working, so future work is about analysis + documentation, not basic setup.

**Q3. What felt heavy or draining?**
- CCSK Domain 2 reading:
  - The governance / risk / compliance material felt more “PMBOK-like” and idealized than how things work in practice.
  - Intellectually fine, but not as engaging as hands-on, applied work in the lab.
- OWASP Cheat Sheet issue triage:
  - Took time and attention with little actionable work before I pivoted to OWASP Nest.

**Q4. What did I learn or notice about myself?**
- I actually enjoy the “real-world” architecture work:
  - Debugging noisy alarms with CloudTrail, Logs Insights, and Security Hub felt energizing, not draining.
  - Investigating AccessDenied events and tuning detections made me feel closer to how a real architect works.
- Governance/GRC content (like CCSK Domain 2) is useful context, but it doesn’t light me up the way hands-on detection and architecture work does.
- When a contribution path stalls (Cheat Sheets), I’m willing to pivot to a better fit (OWASP Nest API docs) instead of forcing progress on the wrong thing.

**Q5. Did I stay roughly within the 6–8 hour target?**
- Yes / Mostly:
  - CloudWatch + investigation + tuning took the bulk of the time.
  - OWASP work was split between exploration and concrete Nest setup.
  - CCSK reading was focused (Domain 2 only) and didn’t blow up the week.

**Q6. Top 1–2 adjustments for Week 3**
- Default to **CloudWatch Logs Insights** earlier when troubleshooting alarms instead of wrestling with CloudTrail Event history.
- Treat OWASP Nest as the primary contribution track:
  - Next steps: identify which Nest endpoints/docs to clean up first, map current behavior from OpenAPI, and draft improved API documentation.
  - Only check other OWASP opportunities opportunistically, so I don’t dilute focus.

## Week 1 Retro (2025-11-17 to 2025-11-23)

**1. What did I ship this week?**

- Stood up a new AWS personal lab account.  
- Locked down root (MFA on, no root access keys) and created an admin IAM user with MFA.  
- Configured a multi-Region CloudTrail trail to S3.  
- Enabled AWS Config (all resource types, continuous recording).  
- Enabled GuardDuty and Security Hub with CIS AWS Foundations.  
- Created the public GitHub repo `erick-cloudsec-lab` with docs and Week 1 evidence screenshots.  
- Downloaded CSA Security Guidance v5 and the CCSK exam prep kit and read Domains 1 and 5.  
- Completed and passed the “AWS Security Fundamentals (Second Edition)” course.

**2. What moved me closer to Cloud/AppSec Architect interviews?**

- I have an AWS lab started where I can continue to experiment and learn.  
- I now have a working AWS security baseline I can demo and talk through.  
- I have a public GitHub repo that shows real hands-on work, not just theory.  
- CCSK Domains 1 & 5 plus AWS Security Fundamentals gave me the language to connect what I built to cloud security concepts.

**3. What felt heavy or draining?**

- Trying to keep both the “what” and the “how to” specifics of AWS straight, since a lot of it is new to me.  
- I’m hoping that with more practice reps it will become increasingly second nature.

**4. What will I do less of next week?**

- Fewer long, late-night AWS sessions when I’m already fried.  
- Less trying to configure multiple new services in one sitting; more small, focused blocks.

**5. One win to celebrate**

- I went from no lab to a real AWS security baseline plus a public repo plus passing AWS Security Fundamentals in a single week.

**6. Top 1–2 priorities for next week**

- Review top Security Hub CIS findings and pick 1–2 controls to improve in the lab.  
- Start exploring possible OWASP projects I could realistically contribute to.

## Additional notes

- The metrics and alarms steps made sense at a high-level, but I’m not 100% comfortable with the finer details yet.  
- Learned that Security Hub may not have controls for some of the CloudWatch items I configured, which makes me wonder why they were removed in CIS v5.
