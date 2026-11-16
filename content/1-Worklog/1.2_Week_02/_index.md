---
title: "Week 2 Worklog"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Master identity and access management architecture on AWS (**AWS IAM**): Users, User Groups, Policies, IAM Roles, and temporary delegation via `sts:AssumeRole`.
* Attend on-site office hours at FCAJ and support technical preparations for community events.

### Tasks Completed This Week:
* **AWS IAM Security Foundations:** Studied the separation between Root account, IAM Users, IAM User Groups, Managed vs. Inline Policies, and the **Principle of Least Privilege**.
* **Enterprise-grade Identity Setup:**
  * Created the IAM User Group `FCAJ-Admin` attached with the `AdministratorAccess` managed policy.
  * Created a dedicated IAM User `aotam_admin` for day-to-day cloud engineering work, safely locking away the Root account.
  * Enabled secure Billing Console access for non-root administrative IAM users.
* **IAM Role & STS Delegation:** Created IAM Role `AdminRole` with a Trust Policy permitting `sts:AssumeRole`; practiced the **Switch Role** workflow directly in the AWS Console and verified Temporary Security Credentials generation via AWS STS.
* **On-site Office Support:** Reported to the FCAJ office to collaborate with program coordinators, inspecting network connectivity, audio-visual presentation equipment, and technical logistics for upcoming community workshops.

### Week 2 Achievements:
* Successfully fulfilled on-site office duty and contributed technical support to FCAJ community operations.
* Established an AWS security baseline: complete isolation of Root User, centralized access control via `FCAJ-Admin` group and `aotam_admin` user.
* Gained a solid grasp of long-term credentials (IAM Users) vs. short-term STS tokens (IAM Roles), laying the foundation for cross-service authorization between S3 and Redshift.

### References:
* <https://cloudjourney.awsstudygroup.com/>
* <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html>
* <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html>
