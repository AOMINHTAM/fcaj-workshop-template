---
title: "5.6 Resource Clean-up to $0.00"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### Why Clean-Up Order Matters
Because `S3RawBucket` enables **`VersioningConfiguration: Enabled`**, a plain `aws s3 rm --recursive` only inserts *DeleteMarkers* without removing underlying *Object Versions*, causing `delete-stack` to fail with `BucketNotEmpty`. Follow the 4 steps below to tear down 100% of AWS resources cleanly to **`$0.00`**.

---

### Step 1 — Stop Local Apache Airflow Triggerer
Immediately halt SQS Long Polling requests:
```bash
astro dev stop
```

---

### Step 2 — Purge S3 Object Versions & Delete CloudFormation Stack in 1 Command
```bash
make cfn-delete
```
`make cfn-delete` automates the full teardown sequence:
1. Executes `scripts/empty_versioned_bucket.py` (paginated purge of all `Versions` and `DeleteMarkers` in batches of 500, handling buckets with > 1,000 versions).
2. Runs `aws cloudformation delete-stack --stack-name olist-v5-pipeline-stack --region us-east-1`.
3. Waits for `stack-delete-complete`, removing the S3 Bucket, SQS Main Queue, SQS DLQ, 4 CloudWatch Alarms, SNS Topic, and Stack IAM Roles/Policies.

---

### Step 3 — Delete Standalone Redshift Serverless (if created outside the Stack via `DeployRedshift=false`)
```bash
# 1. Delete the Workgroup first
aws redshift-serverless delete-workgroup \
  --workgroup-name ecommerce-workgroup \
  --region us-east-1

# 2. After Workgroup deletion finishes, delete the Namespace (without keeping a final snapshot)
aws redshift-serverless delete-namespace \
  --namespace-name ecommerce-namespace \
  --region us-east-1
```

---

### Step 4 — Verify Zero Lingering Resources
```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE DELETE_FAILED \
  --region us-east-1 \
  --query "StackSummaries[].{Name:StackName,Status:StackStatus}" \
  --output table
```
Double-check **AWS Billing & Cost Management** and **AWS Budgets** to confirm zero ongoing charges.
