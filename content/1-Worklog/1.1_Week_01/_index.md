---
title: "Week 1 Worklog"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:
* Attend the **Workforce Bootcamp - First Cloud AI Journey (FCAJ)** Kick-off event, understand program regulations, deliverables, and align with the **Data & Analytics** track.
* Set up an AWS Free Tier account, resolve identity verification blockers, and complete the **Explore AWS** tasks to unlock **$200 in AWS Credits**.
* Establish **FinOps** cost guardrails using **AWS Budgets** with 4 automated alert thresholds.

### Tasks Completed This Week:
* **FCAJ Bootcamp Kick-off:** Attended the opening ceremony, reviewed internship rules, project evaluation criteria, and the 12-week hands-on technical roadmap.
* **AWS Account Provisioning & Troubleshooting:** Registered a personal AWS Free Tier account; proactively opened an AWS Support ticket (Case ID: `179040404500475`) to resolve automated SMS/call verification failures and prepared a backup account to maintain progress.
* **Completed Explore AWS Tasks ($200 in Credits Earned):**
  * *Task 1 (Amazon EC2):* Launched a `t2/t3.micro` EC2 instance with key pair `first-kp.pem` and safely terminated it (+$20).
  * *Task 2 (Amazon Bedrock):* Explored the Bedrock Text Playground with Claude Haiku and Amazon Titan models (+$20).
  * *Task 3 (AWS Budgets):* Established an initial cloud cost control budget (+$20).
  * *Task 4 (AWS Lambda):* Deployed an HTTP blueprint serverless function (`http-function-url-tutorial`) and safely deleted it (+$20).
  * *Task 5 (Amazon RDS):* Created an Aurora PostgreSQL database and practiced a 2-tier resource clean-up process covering both DB Instance and DB Cluster (+$20).
* **FinOps Cloud Cost Governance:** Configured a master **AWS Budgets** alert with a **$200** limit across 4 progressive email notification tiers (`12.5% - $25`, `25% - $50`, `50% - $100`, `75% - $150`) to eliminate accidental billing risks.

### Week 1 Achievements:
* Proactively handled account verification issues via AWS Support without delaying hands-on progress.
* Completed all 5 **Explore AWS** tasks and verified the full **$200 Credit** balance in the Billing Console.
* Deployed a **4-tier AWS Budgets FinOps guardrail** protecting the account against unexpected charges.
* Established strict **multi-layer resource clean-up habits** across EC2, Lambda, and RDS Aurora clusters.

### References:
* <https://hcm-rules.awsfcaj.com/>
* <https://cloudjourney.awsstudygroup.com/>
* <https://aws.amazon.com/free/>
* <https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html>
