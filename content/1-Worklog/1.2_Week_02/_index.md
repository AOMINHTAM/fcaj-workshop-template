---
title: "Week 2: IAM Security Governance & Event Technical Support"
weight: 2
---

### 🇬🇧 Week 02 Worklog

**1. Tasks Completed:**
- **Office Attendance & Community Contribution:** Attended the office on Thursday (01/10/2026) and assisted the organizing team in setting up network connectivity, AV equipment, and logistics for the upcoming Friday event (02/10/2026).
- **AWS Identity & Access Management (IAM):**
  - Created the dedicated IAM User Group **`FCAJ-Admin`** attached with the managed `AdministratorAccess` policy.
  - Provisioned the IAM User `aotam_admin` for regular administration, eliminating persistent Root user sessions.
  - Activated IAM user access to Billing and Cost Management console.
  - Created the IAM Role `AdminRole` configured with a Trust Policy supporting cross-account / identity delegation via `sts:AssumeRole`.
  - Executed the **Switch Role** workflow directly from the AWS Console navigation bar to validate temporary credential issuance under the Principle of Least Privilege.

**2. Key Deliverables & Results:**
- Logged 01 office attendance session with on-site event support contributions.
- Mastered core IAM building blocks: contrasting long-term human credentials (IAM Users) against temporary service delegation tokens (IAM Roles).
- Verified seamless role-switching operations on the AWS Console.

**3. References:**
- FCAJ Lab Guide: Access Management with AWS Identity and Access Management (AWS IAM).
- AWS Documentation: Security Best Practices in IAM & Role Switching.