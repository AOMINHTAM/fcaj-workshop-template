---
title: "Week 2 Worklog"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Master **AWS Identity and Access Management (IAM)** architecture: Users, User Groups, Policies, IAM Roles, and temporary credential delegation (`sts:AssumeRole`).
* Attend on-site office duty at the FCAJ office and provide technical support for community event preparation.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :--- |
| **Mon (Day 2)** | - Study **AWS IAM** fundamentals <br> - Understand Root account vs. IAM Users, IAM Groups, and the **Principle of Least Privilege** | 05/10/2026 | 05/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| **Tue (Day 3)** | - **Hands-on IAM Setup:** <br>&emsp; + Create IAM User Group `FCAJ-Admin` with `AdministratorAccess` policy <br>&emsp; + Provision IAM User `aotam_admin` for daily engineering tasks and lock away Root credentials | 06/10/2026 | 06/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html> |
| **Wed (Day 4)** | - Enable IAM user access to the Billing and Cost Management console <br> - Create **IAM Role (`AdminRole`)** configured with a Trust Policy supporting `sts:AssumeRole` | 07/10/2026 | 07/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html> |
| **Thu (Day 5)** | - **FCAJ Office Attendance:** Work on-site at the office, assisting the organizing team with network checks, AV equipment setup, and event logistics | 08/10/2026 | 08/10/2026 | FCAJ Office Guidelines |
| **Fri (Day 6)** | - Practice the **Switch Role** workflow on the AWS Management Console <br> - Verify temporary security credential issuance via AWS STS | 09/10/2026 | 09/10/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 2 Achievements:
* Completed on-site office attendance and contributed to technical logistics for FCAJ community events.
* Enforced AWS identity security best practices by eliminating daily Root usage and centralizing permissions via `FCAJ-Admin` and `aotam_admin`.
* Mastered IAM Role delegation (`sts:AssumeRole`), providing a solid security foundation for cross-service S3-to-Redshift authentication in later weeks.
