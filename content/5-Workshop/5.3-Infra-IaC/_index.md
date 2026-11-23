---
title: "5.3 AWS Infrastructure Provisioning via CloudFormation IaC"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### 1. CloudFormation Template Design (`infra/cloudformation/template.yaml`)
All V5 infrastructure is declared in `infra/cloudformation/template.yaml`, provisioning **15 core resources**, **4 conditional FinOps/Redshift resources** (`DeployRedshift=true/false`), and **16 Outputs**:

* **Resolving Dual Circular Dependencies via `!Sub`:**
  * *Cycle 1 (`S3RawBucket ↔ SqsMainQueuePolicy`):* `SqsMainQueuePolicy` constructs the bucket ARN via `!Sub`, while `S3RawBucket` declares `DependsOn: SqsMainQueuePolicy` so S3's initial `s3:TestEvent` succeeds on a single pass.
  * *Cycle 2 (`SqsMainQueue ↔ SqsDeadLetterQueue`):* `SqsDeadLetterQueue` builds the source queue ARN via `!Sub` inside `RedriveAllowPolicy: byQueue`, enforcing deterministic order: `DLQ → Main Queue → Queue Policy → S3 Bucket`.
* **5-Layer Amazon S3 Security (`S3RawBucket` + `S3RawBucketPolicy`):** `AES256` encryption, all 4 Public Access Block flags, HTTPS-only `Deny` policy (`aws:SecureTransport: "false"`), **Versioning enabled**, and Lifecycle rules (90-day `archive/` expiration, 30-day noncurrent version purge, orphan `DeleteMarker` & 7-day incomplete multipart cleanup).
* **3-Way Split Least-Privilege IAM (`*-${AWS::Region}`):**
  1. `RedshiftS3ReadOnlyRole`: Scoped to `aws:SourceAccount` and `raw/*` read-only.
  2. `AirflowRuntimePolicy`: Consume/ACK on `SqsMainQueue`, `GET/DELETE` on `raw/*`, and `GET/PUT` (**no `DELETE`**) on `archive/*`.
  3. `DlqOperatorPolicy`: Separated operator permissions for DLQ inspection and `StartMessageMoveTask` (DLQ Redrive).
* **4 CloudWatch Alarms (`TreatMissingData: notBreaching`) + `PipelineAlertTopicPolicy` + SNS:**
  1. `DlqMessagesVisibleAlarm`: `ApproximateNumberOfMessagesVisible >= 1` (1 min).
  2. `MainQueueOldestMessageAlarm`: `ApproximateAgeOfOldestMessage > 600s` (10-min SLA).
  3. `MainQueueInFlightAlarm`: `ApproximateNumberOfMessagesNotVisible >= 1` sustained 15 mins.
  4. `MainQueueBacklogAlarm`: `ApproximateNumberOfMessagesVisible >= 20` over 3 mins.

---

### 2. Step 1 — Static Lint & Change Set Dry-Run
```bash
# Static validation with cfn-lint and AWS CloudFormation Engine
cfn-lint infra/cloudformation/template.yaml
make cfn-validate

# Preview infrastructure changes safely with a Change Set
aws cloudformation deploy \
  --stack-name olist-v5-pipeline-stack \
  --template-file infra/cloudformation/template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --no-execute-changeset \
  --region us-east-1 \
  --parameter-overrides \
    ProjectPrefix=olist-v5 \
    EnvironmentName=dev \
    AlertEmail="your-email@example.com" \
    DeployRedshift=false
```

---

### 3. Step 2 — Deploy Stack & Inspect Outputs
The default `DeployRedshift=false` mode deploys all 15 core resources within AWS Always Free limits (reusing an existing Redshift Serverless workgroup). To also provision Redshift Serverless via IaC (`PubliclyAccessible=false`, `/32` CIDR, Secrets Manager password), pass `DeployRedshift=true` along with `VpcId` and `SubnetIds`.

```bash
make cfn-deploy ALERT_EMAIL="your-email@example.com"
make cfn-status
```

Once the stack reaches **`CREATE_COMPLETE`**, copy the 4 required Outputs (`S3RawBucketName`, `SqsMainQueueName`, `SqsMainQueueUrl`, `RedshiftS3ReadOnlyRoleArn`) into `.env`.

*(If any resource fails during creation or update, CloudFormation automatically triggers **Native Auto-Rollback** to revert cleanly).*

---

### 4. Step 3 — Confirm Amazon SNS Email Subscription
Open your `ALERT_EMAIL` inbox, locate the message from `AWS Notifications`, and click **Confirm subscription** to enable real-time CloudWatch → SNS notifications.

![Confirmed Amazon SNS Email Subscription](/fcaj-workshop-template/images/workshop/sns_v5_subscription_confirmed.png)
