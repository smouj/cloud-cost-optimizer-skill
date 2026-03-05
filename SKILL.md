---
name: "cloud-cost-optimizer"
version: "1.2.0"
description: "Analyzes multi-cloud spending patterns, identifies underutilized resources, reserved capacity opportunities, and provides actionable savings recommendations for AWS, Azure, and GCP"
author: "SMOUJBOT"
tags: ["cloud", "cost", "optimization", "billing", "savings", "AWS", "Azure", "GCP", "finops"]
dependencies:
  - "awscli>=2.15.0"
  - "azure-cli>=2.55.0"
  - "gcloud-sdk>=467.0.0"
  - "jq>=1.6"
  - "python3>=3.9"
  - "pandas>=2.0.0"
  - "matplotlib>=3.7.0"
required_env:
  - "AWS_ACCESS_KEY_ID"
  - "AWS_SECRET_ACCESS_KEY"
  - "AZURE_SUBSCRIPTION_ID"
  - "GCP_PROJECT_ID"
  - "COST_THRESHOLD_DAYS=30"
platforms: ["linux", "darwin"]
timeout: 300
---

# Cloud Cost Optimizer

## PURPOSE

Provides intelligent cost analysis and optimization recommendations across AWS, Azure, and GCP infrastructure. Identifies specific waste patterns and delivers quantified savings opportunities with implementation commands.

**Real use cases:**

1. **Weekly cost anomaly detection** - Scan last 7 days of billing data, flag services with >20% cost spikes vs previous period, identify root causes (e.g., unattached EBS volumes, idle load balancers, overprovisioned VMs)
2. **Reserved capacity planning** - Analyze historical usage of EC2 instances, RDS databases, and Azure VMs to recommend 1-year or 3-year reservation purchases with precise coverage percentages
3. **Resource right-sizing** - Evaluate CPU/memory utilization of running instances, recommend downsizing opportunities with confidence intervals (e.g., "t3.large → t3.medium would save $142/month with 95% performance retention")
4. **Storage tier optimization** - Identify S3/Blob/Cloud Storage objects with access patterns suitable for infrequent access or archive tiers, calculate storage class transition savings
5. **Orphaned resource cleanup** - Detect unattached volumes, stale snapshots, unused Elastic IPs, empty S3 buckets, idle Cloud SQL instances with age and cost impact metrics
6. **Tag compliance enforcement** - Scan resources missing required cost allocation tags (Environment, Owner, Project), generate reports for governance teams
7. **Budget forecasting** - Project 30-day spend based on current run-rate, alert when exceedances predicted with confidence intervals

## SCOPE

### Core Commands

**analyze-spend**
```bash
kilo cloud-cost-optimizer analyze-spend \
  --platform <aws|azure|gcp|all> \
  --days <integer> \
  --output <json|csv|html> \
  --threshold <percentage>
```
Real example: `kilo cloud-cost-optimizer analyze-spend --platform all --days 30 --output html --threshold 15`

**idle-detector**
```bash
kilo cloud-cost-optimizer idle-detector \
  --resource-type <ec2|ebs|elastic-ip|load-balancer|azure-vm|gcp-disk> \
  --idle-threshold-hours <integer> \
  --region <region-name>
```
Real example: `kilo cloud-cost-optimizer idle-detector --resource-type ec2 --idle-threshold-hours 72 --region us-east-1`

**reservation-advisor**
```bash
kilo cloud-cost-optimizer reservation-advisor \
  --platform <aws|azure|gcp> \
  --term <1-year|3-year> \
  --payment-option <all-upfront|partial-upfront|no-upfront> \
  --confidence <0.7-0.99>
```
Real example: `kilo cloud-cost-optimizer reservation-advisor --platform aws --term 3-year --payment-option partial-upfront --confidence 0.85`

**right-size-recommendations**
```bash
kilo cloud-cost-optimizer right-size-recommendations \
  --platform <aws|azure|gcp> \
  --metric <cpu|memory|network|all> \
  --utilization-threshold <0.1-0.5> \
  --instance-family <specific-family>
```
Real example: `kilo cloud-cost-optimizer right-size-recommendations --platform aws --metric cpu --utilization-threshold 0.2`

**storage-analyzer**
```bash
kilo cloud-cost-optimizer storage-analyzer \
  --service <s3|azure-blob|gcp-storage> \
  --bucket <bucket-name> \
  --days-access <integer> \
  --transition-to <standard|ia|archive>
```
Real example: `kilo cloud-cost-optimizer storage-analyzer --service s3 --bucket my-app-logs --days-access 90 --transition-to ia`

**tag-compliance**
```bash
kilo cloud-cost-optimizer tag-compliance \
  --required-tags <comma-separated> \
  --platform <aws|azure|gcp> \
  --resource-types <comma-separated>
```
Real example: `kilo cloud-cost-optimizer tag-compliance --required-tags Environment,Owner,Project --platform aws --resource-types ec2,s3,lambda`

**forecast-spend**
```bash
kilo cloud-cost-optimizer forecast-spend \
  --platform <aws|azure|gcp> \
  --days <30|60|90> \
  --include-commitments <true|false>
```
Real example: `kilo cloud-cost-optimizer forecast-spend --platform aws --days 30 --include-commitments true`

## WORK PROCESS

### 1. Authentication & Configuration
```bash
# Set environment variables (loaded from ~/.openclaw/config/ext/custom-overrides.json)
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"
export AZURE_SUBSCRIPTION_ID="12345678-1234-1234-1234-123456789012"
export GCP_PROJECT_ID="my-project-123456"
export COST_THRESHOLD_DAYS=30
```

### 2. Run Analysis (Real command sequence)

**Multi-cloud spending analysis:**
```bash
kilo cloud-cost-optimizer analyze-spend \
  --platform all \
  --days 30 \
  --output json \
  --threshold 15 > cost-analysis-$(date +%Y%m%d).json
```

**Idle EC2 detection:**
```bash
kilo cloud-cost-optimizer idle-detector \
  --resource-type ec2 \
  --idle-threshold-hours 168 \
  --region us-east-1 \
  --output csv > idle-ec2-us-east-1-$(date +%Y%m%d).csv
```

**Reservation recommendations:**
```bash
kilo cloud-cost-optimizer reservation-advisor \
  --platform aws \
  --term 3-year \
  --payment-option partial-upfront \
  --confidence 0.85 > reservations-recommendations-$(date +%Y%m%d).txt
```

### 3. Implementation Workflow

After receiving recommendations:
1. Review high-impact items (> $100/month potential savings)
2. Test non-destructive actions in staging environments first
3. Schedule changes during maintenance windows
4. Document every change with JIRA ticket reference
5. Monitor for 7 days post-implementation

### 4. Verification Steps

**Before changes:**
```bash
# Capture baseline
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --start-time $(date -d '7 days ago' -Iseconds) --end-time $(date -Iseconds) --period 86400 --statistics Average
az monitor metrics list --resource <resource-id> --metric "Percentage CPU" --interval PT1H
gcloud monitoring time-series list --filter='metric.type="compute.googleapis.com/instance/cpu/utilization"' --project=$GCP_PROJECT_ID
```

**After changes:**
```bash
# Wait 24-48 hours, then compare costs
kilo cloud-cost-optimizer analyze-spend --platform aws --days 2 --output json | jq '.total_savings_expected'
```

## GOLDEN RULES

1. **Never delete resources without 7-day idle confirmation** - Use `--dry-run` flag first, verify metrics manually, implement deletion schedule with 48-hour grace period
2. **Never modify production resources during business hours** - Restrict changes to 02:00-06:00 local timezone, require change management ticket approval
3. **Always preserve audit trails** - Tag every optimization action with `CostOptimized=true` and `OptimizationDate=YYYY-MM-DD`, create AWS Config rules to prevent deletion of tagged resources
4. **Reservations require capacity reservation** - Before purchasing 3-year reservations, verify `aws ec2 describe-reserved-instances-offerings` has capacity in target region, Azure `az vm reservation list` shows availability
5. **Right-sizing requires performance testing** - Load test target instance for 1 hour minimum in staging, compare p99 latency and throughput before production cutover
6. **Storage transitions are irreversible** - Archive to Glacier/Archive tiers has 90-day minimum retention, calculate egress costs before transition
7. **Budget alerts must be set** - After optimization, create/update AWS Budgets with 80%, 100%, 120% thresholds, integrate with Slack/PagerDuty webhooks
8. **Cross-cloud coordination required** - If optimizing AWS and Azure simultaneously, ensure application can handle mixed instance types and region failovers
9. **Reserved instance flexibility check** - Verify `aws ec2 describe-reserved-instances --filters Name=scope,Values=Availability Zone` matches actual deployment zones before purchase
10. **Compliance validation mandatory** - After tag cleanup, run `aws resourcegroupstaggingapi get-resources` to ensure no resources orphaned without required tags

## EXAMPLES

**Example 1: Detect unattached EBS volumes**
```bash
$ kilo cloud-cost-optimizer idle-detector --resource-type ebs --idle-threshold-hours 24 --region us-west-2
Found 47 unattached EBS volumes in us-west-2:
  - vol-0a1b2c3d4e5f6g7h8 (100 GiB, gp3) - $10.30/month, unattached for 15 days
  - vol-1a2b3c4d5e6f7g8h9 (500 GiB, io2) - $75.00/month, unattached for 89 days
Total potential savings: $2,467/month

# Implementation command (review first):
aws ec2 delete-volume --volume-id vol-0a1b2c3d4e5f6g7h8
```

**Example 2: Reserved instance purchase recommendation**
```bash
$ kilo cloud-cost-optimizer reservation-advisor --platform aws --term 1-year --payment-option all-upfront
Based on 90-day usage analysis:

Instance: m5.large (2 vCPU, 8 GiB)
  Current on-demand spend: $1,872/year
  1-year All Upfront RI: $1,125 (40% savings)
  Savings: $747/year
  Risk: Low (95% utilization consistency)
  
Purchase command:
aws ec2 purchase-reserved-instances-offering \
  --reserved-instances-offering-id 1234abcd-12ab-34cd-56ef-1234567890ab \
  --instance-count 3 \
  --dry-run
```

**Example 3: EC2 right-sizing analysis**
```bash
$ kilo cloud-cost-optimizer right-size-recommendations --platform aws --metric cpu --utilization-threshold 0.15
Instance: i-0a1b2c3d4e5f6g7h8 (m5.xlarge)
  Avg CPU: 8% (max 12%)
  Avg Memory: 4.2 GiB / 16 GiB (26%)
  Current cost: $196/month
  Recommended: m5.large ($98/month)
  Savings: $98/month
  Confidence: 97% (based on 30-day metrics)
  
Downgrade command:
aws ec2 modify-instance-attribute --instance i-0a1b2c3d4e5f6g7h8 --instance-type m5.large
```

**Example 4: S3 storage class transition**
```bash
$ kilo cloud-cost-optimizer storage-analyzer --service s3 --bucket prod-backups --days-access 180
Bucket: prod-backups (2.4 TB total)
  Objects not accessed in 180+ days: 1.8 TB (75%)
  Current cost: $55/month (Standard)
  After transition to Intelligent-Tiering: $33/month
  Savings: $22/month ($264/year)
  
Batch transition command (dry-run first):
aws s3 cp s3://prod-backups s3://prod-backups --storage-class INTELLIGENT_TIERING --recursive --dryrun
```

**Example 5: Multi-cloud cost anomaly detection**
```bash
$ kilo cloud-cost-optimizer analyze-spend --platform all --days 7 --threshold 25
AWS: EC2 costs increased 34% ($2,100 → $2,814)
  Root cause: New m5.8xlarge instance launched 2024-01-15
  Action: Verify instance necessity, right-size or terminate

Azure: Storage costs increased 67% ($450 → $752)
  Root cause: Premium SSD provisioned for dev environment
  Action: Convert to Standard SSD

GCP: Compute Engine costs stable, BigQuery costs increased 22%
  Root cause: Unoptimized queries in analytics pipeline
  Action: Implement partitioned tables, add WHERE clauses
  
Total unexpected spend: $1,216
```

## ROLLBACK COMMANDS

**If idle resource deletion was incorrect:**
```bash
# AWS - Restore deleted EBS volume (if snapshot exists)
aws ec2 create-volume --snapshot-id snap-0a1b2c3d4e5f6g7h8 --availability-zone us-east-1a

# Azure - Recreate deleted VM from latest snapshot
az vm create --resource-group my-group --name my-vm --attach-os-disk my-vm-osdisk --os-type linux

# GCP - Restore deleted persistent disk
gcloud compute disks create my-disk --source-snapshot my-snapshot --zone us-central1-a
```

**If right-sizing caused performance issues:**
```bash
# AWS - Revert instance type
aws ec2 modify-instance-attribute --instance i-0a1b2c3d4e5f6g7h8 --instance-type m5.xlarge

# Azure - Change VM size back
az vm resize --resource-group my-group --name my-vm --size Standard_D4s_v3

# GCP - Update machine type
gcloud compute instances set-machine-type my-instance --machine-type n1-standard-4 --zone us-central1-a
```

**If reserved instance purchase was wrong:**
```bash
# AWS - Cancel RI within 5 years (prorated refund)
aws ec2 modify-reserved-instances --reserved-instances-ids 1234abcd-5678-... --instance-count 1

# Azure - Exchange RI (if within 5 years)
az vm reservation exchange --reservation-order-id 11111111-1111-1111-1111-111111111111 --reservation-id 22222222-2222-2222-2222-222222222222

# GCP - Cancel commitment (minimum 1 year)
gcloud compute commitments remove-iam-policy-binding my-commitment --member='user:admin@example.com' --role='roles/compute.viewer'
```

**If storage transition errors occurred:**
```bash
# Restore S3 objects from Standard-IA to Standard (within 30 days)
aws s3 copy s3://my-bucket/key s3://my-bucket/key --storage-class STANDARD --metadata-directive COPY

# Restore Azure blob from Archive (requires rehydration, 1-15 hours)
az storage blob rehydrate --container-name mycontainer --name myblob --tier Hot

# Restore GCP Nearline/Coldline to Standard (7-day minimum retention)
gsutil rewrite -s standard gs://my-bucket/gs://my-bucket/**
```

**Complete rollback (undo all optimizations in last 24 hours):**
```bash
# Generate rollback script from audit log
kilo cloud-cost-optimizer generate-rollback --since "24 hours ago" --platform all --output rollback-$(date +%Y%m%d-%H%M%S).sh

# Execute rollback with confirmation
bash rollback-$(date +%Y%m%d-%H%M%S).sh --dry-run
bash rollback-$(date +%Y%m%d-%H%M%S).sh --execute
```

**Restore from backup before optimization:**
```bash
# If pre-optimization backup created
kilo cloud-cost-optimizer restore-state --backup-file /backups/cloud-state-$(date -d 'yesterday' +%Y%m%d).json --region all
```

## TROUBLESHOOTING

**Issue: "InvalidClientTokenId" from AWS CLI**
- Check: `aws sts get-caller-identity`
- Fix: Verify `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are correct, not expired, have required IAM permissions (`ce:GetCostAndUsage`, `ec2:Describe*`, `s3:ListAllMyBuckets`)

**Issue: "SubscriptionNotRegistered" from Azure CLI**
- Check: `az account show`
- Fix: Ensure `AZURE_SUBSCRIPTION_ID` matches active subscription, run `az account set --subscription $AZURE_SUBSCRIPTION_ID`, register required providers: `az provider register --namespace Microsoft.CostManagement`

**Issue: "Permission denied" on GCP**
- Check: `gcloud auth list`
- Fix: Authenticate with `gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS`, ensure service account has roles: `roles/billing.viewer`, `roles/compute.viewer`, `roles/storage.objectViewer`

**Issue: No data returned for analysis**
- Check: Billing export to S3/Storage bucket is configured, Cost and Usage Report (CUR) is active, data is at least 24 hours old
- Fix: Enable CUR in AWS Billing console, wait 24-48 hours for initial delivery, verify bucket permissions

**Issue: High false-positive rate on idle detection**
- Check: Verify `--idle-threshold-hours` matches business cycle (weekends vs weekdays), validate CloudWatch metrics have sufficient granularity
- Fix: Use `--metric-average-period 300` for 5-minute granularity, cross-reference with application logs before termination

## VERIFICATION CHECKLIST

After any optimization cycle, verify:
- [ ] All deleted resources confirmed unattached >72 hours
- [ ] Right-sized instances passed load test in staging
- [ ] Reserved instance coverage >70% of steady-state workload
- [ ] Storage transitions have lifecycle policies for cleanup
- [ ] Budget alerts configured at 80%, 100%, 120% thresholds
- [ ] Weekly cost report scheduled via cron: `0 6 * * 1 kilo cloud-cost-optimizer analyze-spend --platform all --days 7 --output html > /reports/cost-$(date +\%Y\%m\%d).html`
- [ ] Slack notifications enabled with webhook URL
- [ ] All changes documented in change management system
- [ ] Rollback scripts tested in sandbox environment

## DEPENDENCY INSTALLATION

```bash
# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# GCloud SDK
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-467.0.0-linux-x86_64.tar.gz
tar -xf google-cloud-cli-*.tar.gz
./google-cloud-sdk/install.sh

# Python dependencies
pip3 install --user pandas matplotlib
```

## SECURITY REQUIREMENTS

- Use IAM roles with least privilege, never root credentials
- Rotate API keys every 90 days maximum
- Store credentials in AWS Secrets Manager / Azure Key Vault / GCP Secret Manager, not in shell history
- Enable MFA deletion on S3 buckets containing cost exports
- Encrypt all cost reports at rest with KMS keys
- Restrict skill execution to dedicated cost optimization service account with read-only billing permissions except for specific write actions

## OUTPUT FORMATS

**JSON:**
```json
{
  "timestamp": "2024-01-20T14:30:00Z",
  "platform": "aws",
  "total_monthly_savings": 4523.67,
  "recommendations": [
    {
      "type": "idle_resource",
      "resource_id": "vol-0a1b2c3d4e5f6g7h8",
      "region": "us-east-1",
      "monthly_cost": 10.30,
      "action": "delete",
      "confidence": "high",
      "justification": "Unattached for 15 days, no attachment history"
    }
  ]
}
```

**CSV:**
```csv
type,resource_id,region,monthly_cost,action,confidence
idle_resource,vol-0a1b2c3d4e5f6g7h8,us-east-1,10.30,delete,high
rightsizing,i-0a1b2c3d4e5f6g7h8,us-west-2,196.00,m5.large,medium
reservation,m5.large,us-east-1,747.00,1-year-all-upfront,high
```

**HTML (with embedded charts):** Interactive dashboard with savings by category, time series trends, implementation progress tracker

## AUTOMATION INTEGRATION

**GitHub Actions workflow:**
```yaml
name: Weekly Cost Optimization
on:
  schedule:
    - cron: '0 6 * * 1'
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Cloud Cost Optimizer
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          kilo cloud-cost-optimizer analyze-spend \
            --platform all \
            --days 7 \
            --output html > weekly-cost-report.html
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: cost-report
          path: weekly-cost-report.html
```
```