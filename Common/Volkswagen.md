# 🚗 Volkswagen Group Digital Solutions India
## Senior AWS DevOps Engineer — Interview Preparation Guide

> **Role:** AWS DevOps Engineer | **Location:** Pune (Hybrid) | **Experience:** 4–6 Years  
> **Domains:** CI/CD · Snowflake · Databricks · AWS · Kubernetes · Security · Observability · Tableau

---

## 📋 Table of Contents

1. [CI/CD & Version Control](#1-cicd--version-control)
2. [Cloud Platforms (AWS) & IaC](#2-cloud-platforms-aws--iac)
3. [Snowflake & Databricks](#3-snowflake--databricks)
4. [Security, Access & Identity](#4-security-access--identity)
5. [Observability & Quality (New Relic, SonarQube)](#5-observability--quality)
6. [Scripting & Automation](#6-scripting--automation)
7. [Tableau Administration](#7-tableau-administration)
8. [Scenario-Based Questions](#8-scenario-based-questions)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## Legend

| Badge | Level | What to Expect |
|-------|-------|----------------|
| 🟢 | **Basic** | Concepts, definitions, "what is" questions |
| 🟡 | **Intermediate** | Design, hands-on, "how would you" questions |
| 🔴 | **Hardcore** | Architecture, trade-offs, crisis/incident questions |
| 🎯 | **Scenario** | Real-world situation-based problem solving |

---

## 1. CI/CD & Version Control

> Covers: GitHub Actions, GitLab CI, branching strategies, quality gates, repo bootstrapping, pipeline templates

---

### 🟢 Basic

**Q1. What is CI/CD and why is it important in a DevOps culture?**

**Answer:**  
CI (Continuous Integration) is the practice of automatically building and testing code on every commit to catch integration bugs early. CD (Continuous Delivery/Deployment) automates releasing validated code to production or staging environments.

Together they:
- Reduce manual deployment errors
- Shorten release cycles from weeks to hours
- Provide fast feedback loops to developers
- Enable rollback when something breaks

In the context of this role, CI/CD pipelines are the backbone connecting code commits to live Snowflake/Databricks integrations safely.

---

**Q2. What is a branching strategy and which would you recommend for a large enterprise team?**

**Answer:**  
A branching strategy defines how code is isolated across feature development, releases, and hotfixes in Git.

**GitFlow** — best for teams with scheduled release cycles:
- `main` (production), `develop` (integration), `feature/*`, `release/*`, `hotfix/*`
- Clear structure, good for compliance, slower release pace

**Trunk-Based Development** — best for CI/CD-first teams:
- Single `main` branch with short-lived feature branches (< 1-2 days)
- Forces continuous integration, reduces merge conflicts
- Requires feature flags for incomplete features

For a 4–6yr DevOps role at VW, I'd recommend **Trunk-Based + feature flags** for data pipeline repos and **GitFlow** for infrastructure repos that have quarterly release windows.

---

**Q3. What is a quality gate in CI and what does it enforce?**

**Answer:**  
A quality gate is an automated threshold check that must pass before code can advance in the pipeline. If it fails, the pipeline stops and the PR cannot merge.

Common quality gates:
- Code coverage must be > 80%
- Zero critical/blocker issues from SonarQube
- All unit/integration tests must pass
- No high-severity vulnerabilities in SAST (security) scan
- Dockerfile linting must pass (Hadolint)

In this role, quality gates would be enforced on every PR targeting data platform integration repos (Snowflake connectors, Databricks notebooks, Terraform modules).

---

### 🟡 Intermediate

**Q4. How would you implement branch protection rules and workflow templates for a new repository?**

**Answer:**  
**Branch Protection Rules (GitHub):**
- Require minimum 2 reviewers before merge
- Require CI status checks to pass (build, test, SonarQube gate, security scan)
- Disallow force-push on `main` and `develop`
- Require branch to be up-to-date before merging
- Require signed commits for audit trail
- Restrict who can push directly (CODEOWNERS file)

**CI/CD Workflow Templates:**  
Store reusable templates in a `.github/workflow-templates/` directory in a central org-level repo, or use GitLab's `include` keyword with remote YAML. Templates should cover:
1. `lint-test-build.yml` — linting, unit tests, build artifact
2. `docker-build-push.yml` — build image, scan with Trivy, push to ECR
3. `iac-validate-plan-apply.yml` — terraform fmt, validate, plan, apply with approvals
4. `smoke-test.yml` — post-deploy integration tests

**Repo Bootstrapping:**  
A script (Python + GitHub API) that on new repo creation:
1. Applies branch protection rules via API
2. Copies standard workflow YAMLs
3. Creates required secrets (from Vault/Secrets Manager)
4. Registers repo in Confluence/CMDB
5. Creates SonarQube project and assigns quality profile

---

**Q5. How do you manage secrets securely in a CI/CD pipeline?**

**Answer:**  
- **Never** store secrets in code, pipeline YAML as plaintext, or `.env` files in Git
- Use GitHub Actions Secrets / GitLab CI masked variables for basic pipelines
- For enterprise: integrate with **AWS Secrets Manager** or **HashiCorp Vault** using **OIDC** (OpenID Connect) — no long-lived credentials, pipelines get short-lived tokens
- Rotate secrets automatically using Lambda rotation functions (for Snowflake, Databricks PATs)
- Audit every secret access with CloudTrail or Vault audit logs
- Use `git-secrets` pre-commit hook or GitHub Advanced Security secret scanning to prevent accidental commits

For this role (Snowflake/Databricks): service account passwords stored in AWS Secrets Manager, rotated every 30 days, pipelines fetch at runtime — never stored in environment variables.

---

**Q6. What are SLIs and SLOs and how do you define them for a CI/CD pipeline?**

**Answer:**  
- **SLI (Service Level Indicator):** A specific metric measuring service health (e.g., pipeline success rate)
- **SLO (Service Level Objective):** The target value for that metric (e.g., pipeline success rate ≥ 95% over 7 days)
- **Error Budget:** 100% − SLO = how much failure is acceptable before action is required

**Example SLOs for this role:**

| SLI | SLO | Alert Threshold |
|-----|-----|-----------------|
| Pipeline success rate | ≥ 99% (rolling 7 days) | Alert when < 95% |
| p95 build duration | < 10 minutes | Alert when > 12 min |
| Snowflake data freshness | < 4 hours | Alert when > 3.5 hours |
| Integration smoke test pass rate | ≥ 99.5% | Alert when < 98% |

SLOs drive alert thresholds — if error budget burns too fast, PagerDuty fires and the team enters incident mode.

---

### 🔴 Hardcore

**Q7. Design a repo bootstrapping framework that enforces governance across 100+ repositories at enterprise scale.**

**Answer:**  
**Architecture — Golden Path Platform:**

```
New Repo Created
       ↓
GitHub App Webhook fires
       ↓
Lambda Bootstrapper reads config from 'platform-config' repo
       ↓
┌──────────────────────────────────────────┐
│  1. Apply branch protection rules (API)  │
│  2. Seed CI/CD workflow YAMLs            │
│  3. Create scoped secrets from Vault     │
│  4. Create SonarQube project + profile   │
│  5. Register in CMDB/Confluence          │
│  6. Assign CODEOWNERS from team manifest │
└──────────────────────────────────────────┘
       ↓
Nightly Drift Detection Job
       ↓
Queries all repos' settings via GitHub API
       ↓
Raises PRs or Jira tickets for any deviation
```

**Key design decisions:**
- `platform-config` repo stores canonical templates — branch rules, workflow YAMLs, SonarQube profiles, CODEOWNERS templates
- Bootstrapping is **idempotent** — safe to re-run, fixes drift automatically
- Each environment (DEV/QA/PROD) has separate secrets scopes
- Audit log of every bootstrap action stored in S3 with 1-year retention
- Compliance dashboard in Confluence updated nightly showing governance score per repo

---

**Q8. Design a multi-stage promotion pipeline with automatic rollback for a Databricks/Snowflake data pipeline.**

**Answer:**  
**Pipeline Stages:**

```
DEV                    QA                      PROD
────────────────────────────────────────────────────────
Unit tests           Integration tests        Blue-green deploy
PySpark local        Against QA Snowflake     Smoke tests (15 min)
Schema validation    Great Expectations       Auto-rollback if fail
dbt test             Data quality checks      PagerDuty alert
```

**Automatic Rollback Logic:**
1. Deploy new Databricks job version alongside old (blue-green)
2. Run smoke tests: row count assertions, SLA check, schema validation
3. If all pass within 15 minutes → switch trigger to new job version
4. If any test fails → terminate new job, re-enable old job via Databricks API
5. Track deployment state in DynamoDB (job_id, version, status, timestamp)
6. Alert via New Relic / PagerDuty with full error context

**SoD enforcement:**  
DEV engineers cannot approve PROD deployments — enforced by CODEOWNERS requiring a platform team reviewer for any changes to prod deployment configs.

---

## 2. Cloud Platforms (AWS) & IaC

> Covers: AWS services, Terraform, networking, IAM, FinOps, Kubernetes

---

### 🟢 Basic

**Q1. What AWS services would you use to build a data ingestion pipeline?**

**Answer:**

| Use Case | AWS Service |
|----------|-------------|
| Batch ingestion landing zone | S3 |
| ETL/transformation | AWS Glue |
| Workflow orchestration | Step Functions |
| Streaming ingest | Kinesis Data Streams |
| Stream-to-S3 buffering | Kinesis Data Firehose |
| Monitoring & alerts | CloudWatch + SNS |
| Serverless processing | Lambda |
| Data warehouse | Redshift or Snowflake (via PrivateLink) |

For this role, Glue + Step Functions + Snowflake/Databricks is the typical stack.

---

**Q2. What is Infrastructure as Code (IaC) and which tool would you use and why?**

**Answer:**  
IaC means defining and provisioning infrastructure through version-controlled code files rather than manual UI clicks. Benefits: repeatability, auditability, drift detection, peer review via PRs.

**Tool comparison:**

| Tool | Language | Best For |
|------|----------|----------|
| **Terraform** | HCL | Multi-cloud, large teams, rich provider ecosystem |
| CloudFormation | YAML/JSON | AWS-only, native IAM integration |
| CDK | Python/TypeScript | Developers who prefer general-purpose languages |
| Pulumi | Python/Go/TS | Complex logic needs, modern teams |

For this role: **Terraform** with remote state in S3 + DynamoDB locking is the enterprise standard. Use the Snowflake and Databricks Terraform providers to manage those platforms as code too.

---

**Q3. What is the difference between an IAM Role and an IAM Policy?**

**Answer:**  
- **IAM Policy:** A JSON document defining permissions — what actions are Allowed/Denied on which resources
- **IAM Role:** An identity (no password) that can be assumed by AWS services, EC2 instances, Lambda functions, or CI pipelines. Policies are attached to roles.

**Example for this role:**  
A Glue ETL job assumes `GlueDataPipelineRole` which has:
- Policy A: `s3:GetObject`, `s3:PutObject` on `data-lake-bucket/*`
- Policy B: `snowflake:DescribeConnection` (custom policy)
- Policy C: `secretsmanager:GetSecretValue` on the Snowflake credential secret

Roles are preferred over IAM users with access keys because roles issue **temporary STS tokens** — if leaked, they expire automatically.

---

### 🟡 Intermediate

**Q4. How do you implement least-privilege IAM for a Snowflake/Databricks integration on AWS?**

**Answer:**

**For Snowflake:**
- Create a dedicated IAM Role that Snowflake assumes via **Storage Integration** (no long-lived access keys)
- Grant only `s3:GetObject` and `s3:ListBucket` on specific bucket/prefix
- Snowflake's External ID prevents confused deputy attacks

**For Databricks:**
- Use **Instance Profiles** (IAM roles attached to EC2/EKS nodes) scoped per cluster or workspace
- Never use shared admin credentials or access key pairs
- Separate roles per environment: `DatabricksDevRole`, `DatabricksProdRole`

**Audit & Governance:**
- Run AWS **IAM Access Analyzer** weekly to detect overly permissive policies
- Use AWS **Config rule** `iam-no-inline-policy` to enforce managed policies only
- For SoD: data engineers cannot modify IAM roles — only platform team can, via a gated Terraform repo

---

**Q5. How would you implement FinOps practices for a Snowflake/Databricks environment?**

**Answer:**

**Databricks cost controls:**
- Enable auto-termination on clusters (idle > 30 minutes)
- Use spot instances for non-critical batch workloads
- Implement cluster policies to cap max workers per team
- Tag all clusters: `cost_center`, `project`, `team`, `environment` for chargeback
- Alert when daily DBU spend exceeds threshold via CloudWatch custom metric

**Snowflake cost controls:**
- Set resource monitors with auto-suspend (default 5 min idle)
- Enable warehouse auto-scaling with sensible max clusters (e.g., max 3)
- Analyze `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` for expensive queries
- Use query result caching (free re-use within 24 hours)
- Schedule large transforms during off-peak (lower credit consumption)

**AWS broadly:**
- Cost Explorer tags for per-team visibility
- Budget alerts via SNS at 80% and 100% of monthly budget
- Savings Plans for predictable Glue/EMR workloads
- Monthly FinOps report: Athena query on CUR (Cost & Usage Report) → S3 → QuickSight dashboard

---

**Q6. Explain Kubernetes resource limits and requests — how would you set them for a data workload?**

**Answer:**

- **Requests:** Guaranteed minimum CPU/memory Kubernetes reserves for the pod on a node
- **Limits:** Maximum allowed — exceeding CPU limit causes throttling; exceeding memory limit causes OOMKill

**Example for PySpark on Kubernetes:**

```yaml
# Driver pod
resources:
  requests:
    cpu: "2"
    memory: "8Gi"
  limits:
    cpu: "4"
    memory: "16Gi"

# Executor pods
resources:
  requests:
    cpu: "1"
    memory: "4Gi"
  limits:
    cpu: "2"
    memory: "8Gi"
```

**Best practices:**
- Set `LimitRange` on namespaces to prevent unbounded allocations
- Use `ResourceQuota` per namespace to cap total team consumption
- Configure HPA (Horizontal Pod Autoscaler) on custom metrics (e.g., queue depth) for streaming workloads
- Use `PodDisruptionBudget` for stateful data workloads to prevent eviction during node drains

---

### 🔴 Hardcore

**Q7. Design a multi-account AWS landing zone for a data platform serving Snowflake, Databricks, and Power BI.**

**Answer:**

```
AWS Organizations
├── Management Account (billing, SCPs, governance)
├── Security Account (CloudTrail, Config, GuardDuty, SecurityHub aggregation, WIZ)
├── Shared Services Account (Transit Gateway, DNS, VPC endpoints, Secrets Manager)
└── Data Platform Accounts
    ├── DEV Account (Databricks DEV workspace, Glue DEV, Snowflake DEV via PrivateLink)
    ├── QA Account  (Databricks QA workspace, Glue QA, Snowflake QA via PrivateLink)
    └── PROD Account (Databricks PROD workspace, Glue PROD, Snowflake PROD via PrivateLink)
```

**Network:**
- Hub-spoke topology with **Transit Gateway** — data accounts connect to Shared Services
- Snowflake via **AWS PrivateLink** — no traffic over internet
- Databricks in **VPC-injected private mode** — no public cluster IPs
- Power BI connects to Snowflake via private endpoint

**Governance (SCPs):**
- Deny actions outside approved regions
- Deny disabling CloudTrail
- Require MFA for root accounts
- Deny creation of untagged resources

**Security:**
- AWS Config rules: encryption at rest, no public S3 buckets, required tags
- WIZ CSPM weekly scans with Jira ticketing for findings
- SoD via permission boundaries — engineers cannot modify VPCs or IAM roles

---

## 3. Snowflake & Databricks

> Covers: Roles, connectors, pipelines, performance, admin, integration runbooks

---

### 🟢 Basic

**Q1. What is Snowflake's virtual warehouse and how does scaling work?**

**Answer:**  
A Snowflake virtual warehouse is a cluster of compute resources that executes queries, completely separated from storage (you pay for compute only when queries run).

**Scaling options:**
- **Vertical (scale up):** Resize warehouse from XS → S → M → L → XL → 2XL → 4XL → 6XL (each tier doubles compute)
- **Horizontal (scale out):** Multi-cluster warehouse automatically adds clusters during high concurrency (up to 10 clusters)
- **Auto-suspend:** Warehouse pauses billing when idle (set to 1–5 minutes)
- **Auto-resume:** Restarts automatically on the next query (transparent to users)

For DevOps: manage warehouses via Terraform Snowflake provider or REST API. Set resource monitors per warehouse with credit limits and alert/suspend thresholds.

---

**Q2. What is Databricks and how does it differ from running Spark on EMR?**

**Answer:**

| Feature | Databricks | EMR with Spark |
|---------|-----------|----------------|
| Cluster management | Fully managed, UI/API driven | Manual, complex config |
| Performance | Photon engine (8x faster for SQL) | Standard open-source Spark |
| Data format | Delta Lake (ACID, time travel) | Parquet/ORC (no ACID) |
| Governance | Unity Catalog (lineage, access control) | Manual or AWS Glue Catalog |
| ML | MLflow built-in | Separate setup required |
| Notebooks | Collaborative, version-controlled | Jupyter, separate setup |
| Cost | Higher per-unit, lower total (faster) | Lower per-unit, more ops cost |

For this role, Databricks is preferred because Unity Catalog enables data lineage tracking (required for audit/compliance) and Delta Lake provides schema evolution support needed for Snowflake integration pipelines.

---

**Q3. What is a connector in a Snowflake/Databricks integration and how do you validate it?**

**Answer:**  
A connector is the software bridge enabling one system to read from or write to another.

**Common connectors in this stack:**
- Snowflake JDBC connector in Databricks (spark-snowflake library)
- Databricks Delta Sharing connector for Snowflake
- Power BI Snowflake connector (DirectQuery mode)
- Tableau Snowflake connector (ODBC/JDBC)

**Validation steps (integration runbook):**
1. **Smoke test:** Run `SELECT 1` via the connector — validates auth and network
2. **Schema check:** Verify column types are correctly mapped between systems
3. **Data integrity test:** Load 1,000 rows, compare row count and checksums
4. **Performance test:** Load a representative dataset, measure throughput vs baseline
5. **Error handling:** Pass malformed data, verify error is logged correctly
6. **Runbook update:** Document connection string format, credential location, and retry logic in Confluence

---

### 🟡 Intermediate

**Q4. How do you set up RBAC in Snowflake for a multi-team data platform?**

**Answer:**

**Role hierarchy design:**
```
ACCOUNTADMIN (platform team only)
    └── SYSADMIN
        ├── DATA_ENGINEER_ROLE   → CREATE TABLE, COPY INTO, manage pipelines
        ├── ANALYST_ROLE         → SELECT on mart schemas only
        ├── PIPELINE_ROLE        → Used by service accounts for ETL jobs
        └── TABLEAU_ROLE         → SELECT on published schemas for BI
```

**Best practices:**
- Assign roles to **functional groups**, not individuals — use Okta/AD group sync
- Never grant `SYSADMIN` or `ACCOUNTADMIN` to service accounts
- Use **Secondary Roles** for temporary elevated access (equivalent to sudo)
- Implement **Dynamic Data Masking** — mask PII columns (email, SSN) for ANALYST_ROLE, show plaintext to DATA_ENGINEER_ROLE
- **Row Access Policies** — regional analysts see only their region's data

**Audit:**
```sql
-- Check who has what roles
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.GRANTS_TO_USERS;

-- See all queries run by role
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE ROLE_NAME = 'ANALYST_ROLE'
ORDER BY START_TIME DESC;
```

---

**Q5. How do you set up observability for a Databricks + Snowflake pipeline using New Relic?**

**Answer:**

**Three-layer instrumentation:**

**Layer 1 — Infrastructure:**
- New Relic agent on Databricks driver nodes for cluster CPU/memory/disk metrics
- Snowflake warehouse utilization metrics via New Relic Snowflake quickstart integration

**Layer 2 — Application:**
- Instrument PySpark jobs with New Relic APM (Python agent)
- Trace job stages, task durations, shuffle sizes
- Custom spans for each pipeline phase (ingest → transform → load)

**Layer 3 — Business Metrics:**
- Post custom events to New Relic Insights API:
  - `rows_loaded`, `load_duration_seconds`, `data_freshness_lag_hours`
  - `snowflake_credits_consumed`, `pipeline_run_id`

**SLO Dashboards in New Relic:**
- Data freshness SLO: data must not be > 4 hours stale
- Pipeline success rate SLO: > 99% over rolling 7 days
- Snowflake query p95 < 5 seconds for Power BI users

**Alert routing:**
- P1 (pipeline down) → PagerDuty immediate page
- P2 (SLO burn rate > 5x) → Slack + Jira ticket auto-created
- P3 (approaching threshold) → Jira ticket only

---

**Q6. How do you optimize a slow Databricks PySpark job?**

**Answer:**

**Step 1 — Diagnose first using Spark UI:**
- Check which stage is the bottleneck
- Look for data skew (one task taking 10x longer than others)
- Check shuffle read/write sizes
- Look for spill to disk (indicates memory pressure)

**Step 2 — Apply fixes:**

| Problem | Fix |
|---------|-----|
| Data skew on join key | Salt the skewed key (add random prefix) |
| Large shuffle on join | Broadcast small tables (< 200MB) with `broadcast()` hint |
| Too many small files | Run `OPTIMIZE` + `ZORDER` on Delta table |
| Wrong partitioning | Re-partition on filter columns (date, region) |
| Recomputing same DF | Cache with `df.cache()` or `df.persist()` |
| Slow SQL | Enable AQE: `spark.sql.adaptive.enabled=true` |
| CPU-bound SQL | Enable Photon engine (Databricks Runtime 11+) |

**Step 3 — Delta Lake optimizations:**
```python
# Compact small files
spark.sql("OPTIMIZE my_table ZORDER BY (date, region)")

# Vacuum old versions (keep 7 days by default)
spark.sql("VACUUM my_table RETAIN 168 HOURS")
```

---

### 🔴 Hardcore

**Q7. Design an end-to-end data platform architecture: Snowflake + Databricks + Power BI with full governance.**

**Answer:**

```
Data Sources
    │
    ▼
Kafka → Kinesis → Databricks Structured Streaming
                         │
                         ▼
                   Delta Lake (Bronze - raw)
                         │
                    dbt + Databricks Jobs
                         │
                         ▼
                   Delta Lake (Silver - cleaned)
                         │
                    dbt + data quality tests
                         │
                         ▼
                   Delta Lake (Gold - business-ready)
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Snowflake              Power BI
   (COPY INTO from S3      (DirectQuery via
    or Delta Sharing)       Snowflake SSO)
              │
              ▼
       Tableau Server
    (ODBC + OAuth SSO)
```

**Governance:**
- Unity Catalog in Databricks for data lineage and column-level access control
- Snowflake Dynamic Data Masking for PII in Gold layer
- Power BI Row-Level Security matching Snowflake roles
- All dbt models and notebooks versioned in Git, promoted DEV → QA → PROD via GitHub Actions

**SoD:**
- Data engineers: can modify bronze/silver only
- Analysts: need approval workflow to access gold
- Platform team: CODEOWNERS for infra and IAM changes

---

## 4. Security, Access & Identity

> Covers: SoD, zero-trust, WIZ, credential lifecycle, audit, SOC2

---

### 🟢 Basic

**Q1. What is Separation of Duties (SoD) and how do you implement it in DevOps?**

**Answer:**  
SoD is a security principle ensuring no single person has end-to-end control over a critical process. It reduces fraud, errors, and insider threat risk.

**In DevOps:**
- Developer who writes code ≠ person who approves and deploys it to production
- Enforce via mandatory PR approvals by a different team member
- Protected branches — developers cannot push directly to `main`
- Separate Jira/ServiceNow approval for production change requests

**In Snowflake/Databricks:**
- Pipeline engineer ≠ ACCOUNTADMIN
- Person creating a pipeline ≠ person granting it permissions
- Audit log access is read-only for all except security team

**In Terraform:**
- DEV engineers can `plan` but not `apply` to PROD
- PROD apply requires a separate gated approval in the GitHub Actions workflow

---

**Q2. What is a pre-flight check and why is it important?**

**Answer:**  
A pre-flight check is a set of automated validations run before an integration or deployment executes to catch obvious failures before wasting resources.

**For a Snowflake data load:**
- Source S3 bucket is accessible (IAM check)
- Target schema exists and service account can authenticate
- Snowflake warehouse is running and has available credits
- Upstream pipeline completed successfully (dependency check)
- Row count of source data is within expected range (not suspiciously low)

**For a Databricks job:**
- Cluster configuration is valid
- Delta table schema hasn't changed unexpectedly (schema evolution check)
- Downstream systems are ready to receive data

Pre-flight checks make failures actionable rather than mysterious and prevent cascading failures across dependent pipelines.

---

**Q3. What is WIZ and how does it fit into a DevOps workflow?**

**Answer:**  
WIZ is a **Cloud Security Posture Management (CSPM)** platform that agentlessly scans AWS/Azure/GCP to detect:
- IAM misconfigurations (overly permissive roles, unused permissions)
- Network exposure (public S3 buckets, open security groups)
- Unencrypted storage or data at rest
- Container image vulnerabilities
- Toxic risk combinations (e.g., public EC2 + attached role with admin access)

**In DevOps workflow:**
- WIZ CLI integrates into CI/CD pipeline — scans Terraform plans before `apply` (shift-left security)
- WIZ Security Graph shows blast radius of any vulnerability
- Findings auto-create Jira tickets with SLA by severity (Critical: 24hr, High: 7 days)
- Weekly scan results reviewed in Operational Readiness Reviews (ORR) before any new integration goes live

---

### 🟡 Intermediate

**Q4. How do you implement and audit access controls for a Tableau + Snowflake integration?**

**Answer:**

**Setup:**
1. Create dedicated Snowflake service account `TABLEAU_SVC_USER`
2. Create `TABLEAU_ROLE` — `SELECT` only on published mart schemas, no access to raw/PII tables
3. Enable **Snowflake OAuth for Tableau** — users authenticate via SSO (no shared password)
4. Configure Tableau's Initial SQL to set session context (warehouse, role, time zone)
5. Enable Row Access Policies in Snowflake — regional users see only their region's data

**Audit:**
```sql
-- Every query Tableau ran, by which user
SELECT
    QUERY_TEXT, USER_NAME, ROLE_NAME,
    START_TIME, ROWS_PRODUCED, CREDITS_USED_CLOUD_SERVICES
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE USER_NAME = 'TABLEAU_SVC_USER'
ORDER BY START_TIME DESC;
```

**Automation:**
- Weekly Lambda generates access audit report → S3 → email to security team
- Flags: queries accessing PII columns, logins from new IPs, off-hours access
- Quarterly access review: compare actual Snowflake role grants vs approved entitlements in ServiceNow

---

**Q5. How do you conduct an Operational Readiness Review (ORR)?**

**Answer:**  
An ORR is a structured gate review ensuring a system is production-ready before launch.

**ORR Checklist:**

**Reliability**
- [ ] SLOs defined and documented
- [ ] Runbooks written for top 3 failure modes
- [ ] Alerts tuned — not too noisy, not too silent
- [ ] Auto-rollback tested and working
- [ ] Disaster recovery procedure tested (restore tested, not just backed up)

**Security**
- [ ] WIZ scan passed (zero critical findings)
- [ ] Credentials in Secrets Manager, not in code/config
- [ ] SoD review completed and signed off
- [ ] Penetration test or threat model reviewed

**Observability**
- [ ] Integration visible in New Relic dashboards
- [ ] SLO tracking live
- [ ] Log aggregation configured, retention policy set

**Change Management**
- [ ] Release in change calendar, coordinated with ACS/GITC teams
- [ ] Rollback plan documented and tested
- [ ] Stakeholder communication sent

**Documentation**
- [ ] Integration runbook published in Confluence
- [ ] CODEOWNERS assigned
- [ ] Architecture diagram updated

---

### 🔴 Hardcore

**Q6. Design a zero-trust access model for a hybrid data platform spanning AWS, Snowflake, Databricks, and Power BI with SOC2 requirements.**

**Answer:**

**Zero-trust principles applied:**
- Verify explicitly (every access authenticated and authorized)
- Least privilege (minimum permissions for each identity)
- Assume breach (detect and contain, not just prevent)

**Identity Layer:**
- All human access via SSO (Okta/Azure AD) with MFA enforced
- No passwords stored locally anywhere
- Privileged access via PAM (CyberArk) with session recording

**Machine Identity:**
- Databricks: IAM Instance Profiles (temporary STS tokens, no access keys)
- Snowflake: Key-pair authentication + Snowflake OAuth (no passwords)
- Power BI: Azure Managed Identity
- Glue: IAM Role assumed via STS

**Network:**
- Snowflake: AWS PrivateLink (no public endpoints)
- Databricks: VPC-injected private mode (no public cluster IPs)
- No inbound internet access to any data service

**Audit Trail (SOC2 evidence):**

| System | Audit Log | Retention |
|--------|-----------|-----------|
| AWS | CloudTrail → S3 | 1 year |
| Snowflake | ACCOUNT_USAGE schema | 365 days |
| Databricks | Audit logs → S3 | 1 year |
| Power BI | Activity logs → Log Analytics | 90 days |

**Automated compliance:**
- AWS Config + Security Hub for continuous evidence collection
- Quarterly automated access reviews via IGA tool comparing actual vs approved entitlements
- WIZ weekly scans with Jira tickets tracked to SLA

---

## 5. Observability & Quality

> Covers: New Relic, SonarQube, SLI/SLO, alert tuning, dashboards

---

### 🟢 Basic

**Q1. What is the difference between monitoring and observability?**

**Answer:**

| Aspect | Monitoring | Observability |
|--------|-----------|---------------|
| Question it answers | "Is something wrong?" | "Why is something wrong?" |
| Approach | Predefined metrics + thresholds | Explore unknown failure modes |
| Data types | Metrics | Metrics + Logs + Traces (3 pillars) |
| Example | Alert: pipeline failed | Trace: job failed because Snowflake warehouse suspended due to Databricks job returning 0 rows from a schema change at 03:14 |

New Relic provides all three pillars: infrastructure metrics, distributed tracing (APM), and log management.

**The three pillars of observability:**
- **Metrics:** Numerical measurements over time (CPU %, rows/sec, error rate)
- **Logs:** Timestamped events with context (job started, error stack trace)
- **Traces:** End-to-end request flow across services (pipeline step A → B → C with timing)

---

**Q2. What is SonarQube and what does it analyze?**

**Answer:**  
SonarQube is a static code analysis platform that runs as part of CI and checks:

| Category | What it checks |
|----------|----------------|
| **Bugs** | Logic errors likely to cause runtime failures |
| **Code Smells** | Maintainability issues — duplicated code, complex functions, dead code |
| **Vulnerabilities** | Security issues — SQL injection patterns, hardcoded credentials, insecure deserialization |
| **Coverage** | Lines/branches not covered by tests |
| **Duplications** | Copy-pasted code blocks that should be refactored |

**In CI pipeline:**
1. Test stage runs → generates coverage report
2. SonarQube scanner reads coverage + source code
3. Analysis published to SonarQube server
4. Quality gate evaluated (pass/fail)
5. PR blocked if gate fails

For this role: configure SonarQube projects for each integration repo, assign the appropriate quality profile (Python for PySpark, SQL for dbt), and include the gate check in all CI templates.

---

### 🟡 Intermediate

**Q3. How do you tune alerts to reduce noise in a New Relic setup for data pipelines?**

**Answer:**  
Alert noise causes alert fatigue — engineers stop responding. Systematic tuning:

**Step 1 — Diagnose noise:**
- Export alert history via New Relic NerdGraph API
- Calculate false positive rate per alert (fired but no action taken)
- Identify top 10 noisiest alerts

**Step 2 — Apply tuning techniques:**

| Problem | Fix |
|---------|-----|
| Too sensitive threshold | Increase alert window (5 min sustained, not 1 min spike) |
| Seasonal patterns (weekends) | Switch to **baseline alerting** (anomaly detection) |
| Cascading alerts | Implement **alert inhibition** — suppress downstream when upstream is alerting |
| Wrong recipients | Route by severity: P1 → PagerDuty, P2 → Slack, P3 → Jira only |
| Flapping alerts | Add hysteresis — require N consecutive breaches before firing |

**Step 3 — Governance:**
- Alert-as-code using Terraform New Relic provider (all policies reviewed in PRs)
- Monthly alert review meeting with on-call team
- Track MTTA (Mean Time to Acknowledge) — if > 10 minutes, alert needs tuning
- Target: < 10% false positive rate

---

**Q4. How do you implement SLI/SLO tracking for a Snowflake-based reporting pipeline serving Power BI?**

**Answer:**

**Define SLIs and SLOs:**

| SLI | Measurement | SLO |
|-----|-------------|-----|
| Data freshness | Hours since last successful Snowflake load | < 4 hours during business hours |
| Pipeline success rate | % of scheduled runs completing without error | ≥ 99.5% over 28 days |
| Query response time | p95 Snowflake query time for Power BI sessions | < 5 seconds |
| Extract refresh time | Time to complete Power BI dataset refresh | < 30 minutes |

**Implementation:**
1. Lambda runs every 15 minutes
2. Queries `SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY` to compute data freshness
3. Posts result as custom metric to New Relic Metric API
4. New Relic SLM (Service Level Management) tracks error budget
5. When error budget < 20% remaining for the month → P2 alert fires
6. Confluence SLO page auto-updated via New Relic API integration weekly

---

### 🔴 Hardcore

**Q5. You inherit a New Relic environment with 500+ alerts — 60% are noise. How do you rationalize them?**

**Answer:**

**Phase 1 — Audit (Week 1):**
- Export all alert policies via New Relic NerdGraph GraphQL API
- Join with incident history: which alerts fired > 5×/week with no resulting action?
- Build a heatmap: alert volume vs actual incident correlation rate
- Classify each alert: Signal | Noise | Stale | Misconfigured

**Phase 2 — Remediation by category:**

| Category | Action |
|----------|--------|
| Signal (correlated with real incidents) | Keep, improve routing and context |
| Noise (high frequency, low action rate) | Apply baseline alerting or increase duration window |
| Stale (service no longer exists) | Delete with documented reason |
| Misconfigured (wrong threshold) | Tune using 90-day P95 historical data as new baseline |

**Phase 3 — Governance (permanent fix):**
- All alert policies as Terraform code (New Relic Terraform provider)
- PRs required to change any alert — reviewed by on-call team lead
- Quarterly alert rationalization sprint
- Track false positive rate as a team metric; target < 10%
- Measure MTTA before and after — demonstrate improvement

---

## 6. Scripting & Automation

> Covers: Python, Bash, Terraform, automation patterns, runbooks

---

### 🟢 Basic

**Q1. Write a Bash script to check if a Snowflake endpoint is reachable.**

**Answer:**

```bash
#!/bin/bash

SNOWFLAKE_HOST="myaccount.snowflakecomputing.com"
PORT=443
TIMEOUT=5

echo "Checking Snowflake connectivity to $SNOWFLAKE_HOST:$PORT..."

if timeout $TIMEOUT bash -c "</dev/tcp/$SNOWFLAKE_HOST/$PORT" 2>/dev/null; then
    echo "[OK] Snowflake endpoint is reachable"
else
    echo "[ERROR] Cannot reach Snowflake endpoint $SNOWFLAKE_HOST:$PORT"
    echo "Check: network rules, VPC peering, PrivateLink configuration"
    exit 1
fi

# Full auth test using Python connector
python3 -c "
import snowflake.connector
import sys
try:
    conn = snowflake.connector.connect(
        user='svc_pipeline',
        private_key_path='/secrets/snowflake_key.p8',
        account='myaccount',
        warehouse='PIPELINE_WH',
        database='ANALYTICS'
    )
    conn.cursor().execute('SELECT 1')
    print('[OK] Snowflake authentication successful')
    conn.close()
except Exception as e:
    print(f'[ERROR] Auth failed: {e}')
    sys.exit(1)
"
```

---

**Q2. What is Terraform remote state and why is it critical for teams?**

**Answer:**  
Terraform state is a JSON file mapping HCL configuration to real infrastructure resources. Remote state (S3 + DynamoDB) is critical because:

1. **State locking** (DynamoDB): prevents two engineers running `terraform apply` simultaneously — prevents resource corruption
2. **Shared state**: all team members and CI pipelines read the same current state
3. **Durability**: S3 with versioning — state history is preserved, accidental deletions recoverable
4. **Security**: state often contains sensitive outputs — S3 encryption + strict IAM access

```hcl
# Terraform backend configuration
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "data-platform/prod/snowflake/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"
  }
}
```

For this role: each environment (DEV/QA/PROD) has its own state file path. DEV engineers have read-only access to PROD state — enforced via S3 bucket policy.

---

### 🟡 Intermediate

**Q3. Write a Python script to automate Databricks job submission and monitor completion.**

**Answer:**

```python
import boto3
import requests
import time
import json
from datetime import datetime

def get_databricks_token():
    """Fetch Databricks PAT from AWS Secrets Manager"""
    client = boto3.client('secretsmanager', region_name='ap-south-1')
    secret = client.get_secret_value(SecretId='prod/databricks/pat')
    return json.loads(secret['SecretString'])['token']

def submit_databricks_job(workspace_url: str, token: str, job_id: int, params: dict) -> str:
    """Submit a Databricks job run and return run_id"""
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    payload = {
        "job_id": job_id,
        "notebook_params": params
    }
    response = requests.post(
        f"{workspace_url}/api/2.1/jobs/run-now",
        headers=headers,
        json=payload,
        timeout=30
    )
    response.raise_for_status()
    run_id = response.json()['run_id']
    print(f"[{datetime.now()}] Job submitted. Run ID: {run_id}")
    return run_id

def monitor_job(workspace_url: str, token: str, run_id: str, 
                poll_interval: int = 30, max_wait: int = 3600) -> bool:
    """Poll job status until terminal state. Returns True if successful."""
    headers = {"Authorization": f"Bearer {token}"}
    terminal_states = {"TERMINATED", "SKIPPED", "INTERNAL_ERROR"}
    elapsed = 0

    while elapsed < max_wait:
        response = requests.get(
            f"{workspace_url}/api/2.1/jobs/runs/get?run_id={run_id}",
            headers=headers,
            timeout=30
        )
        response.raise_for_status()
        data = response.json()
        state = data['state']['life_cycle_state']
        result = data['state'].get('result_state', 'RUNNING')
        
        print(f"[{datetime.now()}] State: {state} | Result: {result}")

        if state in terminal_states:
            if result == "SUCCESS":
                print(f"[OK] Job completed successfully in {elapsed}s")
                return True
            else:
                error_msg = data['state'].get('state_message', 'Unknown error')
                print(f"[FAIL] Job failed: {error_msg}")
                return False

        time.sleep(poll_interval)
        elapsed += poll_interval

    print(f"[TIMEOUT] Job did not complete within {max_wait}s")
    return False

# Usage
if __name__ == "__main__":
    WORKSPACE = "https://adb-1234567890.1.azuredatabricks.net"
    TOKEN = get_databricks_token()
    JOB_ID = 12345

    run_id = submit_databricks_job(WORKSPACE, TOKEN, JOB_ID, {
        "run_date": "2026-02-17",
        "source_schema": "raw",
        "target_schema": "silver"
    })

    success = monitor_job(WORKSPACE, TOKEN, run_id)
    exit(0 if success else 1)
```

---

**Q4. How do you manage Snowflake infrastructure as code with Terraform?**

**Answer:**

```hcl
# providers.tf
terraform {
  required_providers {
    snowflake = {
      source  = "Snowflake-Labs/snowflake"
      version = "~> 0.89"
    }
  }
}

provider "snowflake" {
  account   = var.snowflake_account
  username  = var.snowflake_user
  role      = "SYSADMIN"
  # Using key-pair auth — no passwords in Terraform
  private_key_path = var.snowflake_private_key_path
}

# main.tf
resource "snowflake_database" "analytics" {
  name    = "ANALYTICS_${upper(var.environment)}"
  comment = "Analytics database for ${var.environment}"
}

resource "snowflake_warehouse" "pipeline_wh" {
  name           = "PIPELINE_WH_${upper(var.environment)}"
  warehouse_size = var.environment == "prod" ? "MEDIUM" : "XSMALL"
  auto_suspend   = 300      # 5 minutes
  auto_resume    = true
  min_cluster_count = 1
  max_cluster_count = var.environment == "prod" ? 3 : 1
}

resource "snowflake_role" "data_engineer" {
  name = "DATA_ENGINEER_ROLE_${upper(var.environment)}"
}

resource "snowflake_grant_privileges_to_role" "engineer_grants" {
  role_name  = snowflake_role.data_engineer.name
  privileges = ["CREATE TABLE", "CREATE STAGE", "USAGE"]
  on_schema {
    schema_name = "${snowflake_database.analytics.name}.RAW"
  }
}
```

**Pipeline:** terraform plan on PR (output posted as PR comment) → terraform apply only on merge to `main` via GitHub Actions with environment approval gate for prod.

---

### 🔴 Hardcore

**Q5. Design an automated remediation framework for common data platform incidents.**

**Answer:**

```
New Relic Alert Fires
        ↓
SNS Topic receives alert payload
        ↓
Lambda Dispatcher
  - Reads alert metadata (service, alert type, severity)
  - Looks up runbook mapping in DynamoDB
        ↓
Step Function State Machine (per runbook type)

Example: "Snowflake Warehouse Suspended"
┌────────────────────────────────────┐
│ 1. Pre-check: Is warehouse still   │
│    suspended? (avoid race condition)│
├────────────────────────────────────┤
│ 2. Execute fix: ALTER WAREHOUSE    │
│    PIPELINE_WH RESUME              │
├────────────────────────────────────┤
│ 3. Verify: Run smoke test query    │
│    SELECT COUNT(*) FROM heartbeat  │
├────────────────────────────────────┤
│ 4. Post result:                    │
│    - PagerDuty: resolve incident   │
│    - Jira: auto-close ticket       │
│    - Slack: post summary           │
└────────────────────────────────────┘
        ↓
If Step Function fails 3 times:
  → Page on-call engineer via PagerDuty
  → Include full execution log and error context
```

**Other runbook state machines:**
- `databricks-cluster-unhealthy` → terminate + restart cluster via API
- `s3-pipeline-stuck` → re-queue failed SQS messages
- `data-freshness-breach` → trigger emergency Glue job
- `snowflake-credits-exhausted` → resize warehouse + page FinOps team

**Governance:**  
All runbook code (Python Lambda + Step Function ASL) in Git, reviewed via PR, tested monthly in DEV via chaos engineering exercises.

---

## 7. Tableau Administration

> Covers: tabcmd, REST API, performance tuning, scaling, permissions

---

### 🟢 Basic

**Q1. What is tabcmd and what does a DevOps engineer use it for?**

**Answer:**  
`tabcmd` is Tableau's command-line tool for automating Tableau Server administration tasks.

**Common DevOps uses:**

```bash
# Publish a workbook to Tableau Server
tabcmd publish "Sales_Dashboard.twbx" \
  --server https://tableau.company.com \
  --username svc_admin \
  --project "Analytics" \
  --overwrite

# Trigger an extract refresh
tabcmd refreshextracts --datasource "Snowflake_Sales_Data"

# Export a view as PDF (for automated reporting)
tabcmd export "Finance/MonthlyReport" \
  --pdf \
  --filename /reports/monthly_report_$(date +%Y%m).pdf

# Delete stale workbook
tabcmd delete workbook "OldDashboard" --project "Archive"
```

**In CI/CD:** After a Snowflake schema change, a GitHub Actions workflow runs `tabcmd refreshextracts` to update Tableau data sources so dashboards reflect the new schema automatically.

**Note:** tabcmd is being superseded by the Tableau REST API / TSC Python library for complex automation, but is still widely used for quick scripting.

---

**Q2. What are common Tableau performance issues and how do you investigate them?**

**Answer:**

| Symptom | Likely Cause | Investigation |
|---------|-------------|---------------|
| Dashboard loads slowly | Snowflake queries slow | Check Snowflake QUERY_HISTORY for Tableau-generated SQL |
| Extract refresh takes hours | Extract too large, no incremental refresh | Enable incremental refresh on timestamp column |
| "Too many marks" warning | Dashboard shows millions of data points | Aggregate data, use filters, apply data source filters |
| Server high CPU during refresh | Too many concurrent Backgrounder jobs | Increase Backgrounder count, stagger refresh schedules |
| LOD calculations slow | Expensive FIXED LOD expressions | Rewrite as INCLUDE, or pre-compute in Snowflake |

**Investigation tool — Performance Recording:**
1. Enable performance recording for a user session in Tableau Server admin
2. Session generates a detailed trace workbook (JSON-based)
3. Open trace workbook → see time breakdown: query, data engine, rendering
4. Use query text to optimize in Snowflake (add clustering key, create materialized view)

---

### 🟡 Intermediate

**Q3. How do you automate Tableau permission management and content migration using REST API?**

**Answer:**

```python
import tableauserverclient as TSC

# Authenticate with PAT (not username/password)
server = TSC.Server('https://tableau.company.com', use_server_version=True)
tableau_auth = TSC.PersonalAccessTokenAuth(
    token_name='ci_pipeline_token',
    personal_access_token='TOKEN_FROM_SECRETS_MANAGER',
    site_id='DataPlatform'
)

with server.auth.sign_in(tableau_auth):
    # --- Permission Management ---
    # Get all workbooks in a project
    project_filter = TSC.RequestOptions()
    project_filter.filter.add(TSC.Filter('projectName', TSC.RequestOptions.Operator.Equals, 'Analytics'))
    workbooks, _ = server.workbooks.get(project_filter)

    # Grant view permission to a group
    group = server.groups.get_by_name('Data_Analysts')
    for workbook in workbooks:
        permission = TSC.PermissionsRule(group, {'Read': 'Allow', 'Filter': 'Allow'})
        server.workbooks.add_permissions(workbook, [permission])

    # --- Content Migration (DEV → PROD) ---
    # Download workbook from DEV
    dev_server = TSC.Server('https://tableau-dev.company.com', use_server_version=True)
    with dev_server.auth.sign_in(dev_auth):
        workbook_item, _ = dev_server.workbooks.get_by_name('Sales_Dashboard', 'Analytics')
        dev_server.workbooks.download(workbook_item.id, '/tmp/Sales_Dashboard.twbx')

    # Re-publish to PROD with connection remapping
    # (edit .twbx XML to point to PROD Snowflake endpoint)
    new_workbook = TSC.WorkbookItem(project_id='prod_analytics_project_id')
    new_workbook.name = 'Sales_Dashboard'
    server.workbooks.publish(new_workbook, '/tmp/Sales_Dashboard.twbx',
                             TSC.Server.PublishMode.Overwrite)
    print("Migration complete")
```

---

### 🔴 Hardcore

**Q4. Tableau Server is experiencing 300% load increase. Design a scaling and resilience strategy.**

**Answer:**

**Immediate response (Day 0):**
1. Add worker nodes to Tableau Server cluster (horizontal scale)
2. Add Backgrounder processes on worker nodes (they handle extract refreshes — usually the bottleneck)
3. Add VizQL Server processes on rendering-focused nodes
4. Identify and reschedule conflicting extract refreshes (stagger across 24h)
5. Set Snowflake warehouse for Tableau to multi-cluster (max 3) to handle concurrent queries

**Medium-term architecture:**

```
Load Balancer
     │
     ├── Primary Node (repository, gateway, application)
     ├── Worker Node 1 (VizQL Server, Data Server — interactive sessions)
     ├── Worker Node 2 (VizQL Server, Data Server — interactive sessions)
     └── Worker Node 3 (Backgrounder only — extract refreshes, isolated)
```

**Snowflake side:**
- Dedicated `TABLEAU_WH` warehouse with `MAX_CLUSTER_COUNT = 3`
- Separate `TABLEAU_EXTRACT_WH` for scheduled extract refreshes (doesn't compete with live queries)

**Long-term:**
- Migrate to **Tableau Cloud** (Salesforce managed) — eliminates infrastructure ops
- Or implement **Tableau in Containers** on Kubernetes for elastic scaling

**Observability:**
- New Relic agent on all Tableau nodes
- Alert on: VizQL memory > 85%, Backgrounder queue depth > 50 pending jobs, active sessions > 80% of licensed limit
- Capacity planning: track 30-day session trends, forecast license and hardware needs quarterly

---

## 8. Scenario-Based Questions

> Real-world situations you must be able to discuss end-to-end. Use STAR format: Situation → Task → Action → Result.

---

### 🎯 Scenario 1: Production Data Pipeline Down at 2 AM

**Scenario:**  
You're on call. At 2:07 AM, PagerDuty fires: the nightly Snowflake load job has been failing for 3 hours, and the Power BI dashboard used by 500 business users shows stale data. The SLO breach timer has started. What do you do?

**Answer:**

**Immediate (0–15 min):**
1. Acknowledge PagerDuty — start the incident timer
2. Open New Relic dashboard → identify which pipeline stage failed (Glue? Databricks? Snowflake COPY INTO?)
3. Check CloudWatch/Databricks logs for the specific error message
4. If data is stale but not corrupt → send stakeholder communication: "We are aware, investigating, update in 30 minutes"

**Diagnose (15–45 min):**
- Check Snowflake `INFORMATION_SCHEMA.LOAD_HISTORY` — did COPY INTO fail?
- Check S3 source — did upstream job produce the expected file?
- Check IAM credentials — did a Secrets Manager rotation fire at midnight and break the connection?
- Check Snowflake warehouse — auto-suspended and failed to resume?

**Fix (depends on root cause):**
- Expired credential → trigger emergency Secrets Manager rotation, re-run job
- Upstream job failed → re-trigger the upstream pipeline, then re-run this job
- Schema change → update Snowflake target table DDL, re-run job
- Warehouse suspended → `ALTER WAREHOUSE PIPELINE_WH RESUME`, re-run job

**Recovery verification:**
- Confirm Snowflake data freshness metric in New Relic is back in SLO
- Confirm Power BI shows updated data (check `LAST_UPDATED` timestamp)
- Send resolution stakeholder update

**Post-incident (within 48 hours):**
- Write incident report (5-whys root cause)
- Add pre-flight check for the specific failure mode
- Create Jira ticket for permanent fix
- Review alert: why did this silently fail for 3 hours before PagerDuty fired?

---

### 🎯 Scenario 2: Snowflake Credentials Leaked in a Git Commit

**Scenario:**  
A developer accidentally committed Snowflake service account credentials (username + password) to a public GitHub repository. It was live for 45 minutes before being caught by a secret scanning alert. What is your incident response?

**Answer:**

**Containment (within 10 minutes):**
1. Immediately revoke the compromised Snowflake user: `ALTER USER svc_pipeline DISABLE;`
2. Rotate the password via Secrets Manager emergency rotation (even if the account is disabled — belt and braces)
3. If GitHub repo is public: contact GitHub support to request secret removal from git history (or use `git filter-repo` to purge commit from history, then force-push)

**Investigation (within 1 hour):**
```sql
-- Check every action taken with the compromised credential during the 45-minute window
SELECT
    USER_NAME, EVENT_TYPE, CLIENT_IP, QUERY_TEXT,
    START_TIME, ROWS_PRODUCED
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE USER_NAME = 'SVC_PIPELINE'
AND START_TIME BETWEEN '2026-02-17 02:00:00' AND '2026-02-17 02:45:00';

-- Check login attempts and IPs
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY
WHERE USER_NAME = 'SVC_PIPELINE'
AND EVENT_TIMESTAMP BETWEEN '2026-02-17 02:00:00' AND '2026-02-17 02:45:00';
```
- Identify any suspicious IPs (non-company IP ranges)
- Identify what data was accessed/exported
- If PII was accessed → trigger data breach notification process (GDPR/compliance team)

**Eradication:**
- Determine how the credential ended up in code (review dev's workflow — did they have credentials hardcoded in local `.env` file?)
- Remove commit from git history

**Recovery:**
- Enable compromised Snowflake user with new credentials from Secrets Manager
- Verify all pipelines that used the credential are functioning
- Run smoke tests across all 5 dependent pipelines

**Prevention (permanent fixes):**
- Enforce `git-secrets` pre-commit hook across all repos
- Enable GitHub Advanced Security secret scanning on all repos (blocks push if secret detected)
- Move developer local environments to use Vault agent for credential injection (no plaintext credentials ever on disk)
- Add quarterly developer security training on secret management

---

### 🎯 Scenario 3: Databricks Cluster Costs 3× Over Budget

**Scenario:**  
FinOps alerts you that Databricks spend this month is 3× the expected budget. You need to diagnose, stop the bleeding, and prevent recurrence — without breaking production pipelines. What do you do?

**Answer:**

**Diagnose the spike:**
1. Open AWS Cost Explorer → filter by `databricks` tag → identify which clusters/workspaces are driving cost
2. Check Databricks cluster policies: are there clusters running 24/7 that should have auto-termination?
3. Check for zombie clusters (spun up manually by data scientists, never terminated)
4. Check for cluster config violations: is someone using on-demand instances instead of spot?

**Stop the bleeding (immediate):**
```bash
# List all running clusters via Databricks API
curl -X GET \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  "https://adb-workspace.azuredatabricks.net/api/2.0/clusters/list" \
  | jq '.clusters[] | select(.state=="RUNNING") | {cluster_id, cluster_name, creator_user_name}'

# Terminate zombie clusters manually or via script
```
- Enable auto-termination on all clusters immediately (idle > 30 minutes)
- Convert dev/QA workloads to spot instances (90% cost reduction)

**Root cause investigation:**
- A new ML team onboarded and created large all-purpose clusters without cluster policies
- They ran hyperparameter tuning jobs on a 50-node cluster for 72 hours continuously

**Permanent fixes:**
1. **Cluster Policies** — create policies with max cluster size per team:
   ```json
   {
     "num_workers": {"type": "range", "maxValue": 8},
     "autotermination_minutes": {"type": "fixed", "value": 30},
     "node_type_id": {"type": "allowlist", "values": ["spot-instance-types"]}
   }
   ```
2. **Cost tagging enforcement** — require `team`, `project`, `environment` tags via cluster policy (reject clusters without tags)
3. **Budget alerts** — CloudWatch custom metric from Databricks usage API → alert at 50%, 80%, 100% of monthly budget
4. **Monthly FinOps review** — automated report showing cost by team, with anomaly detection

---

### 🎯 Scenario 4: New Databricks/Snowflake Integration Going Live — What's Your Checklist?

**Scenario:**  
Your team is launching a new integration: Databricks reads from Kafka, transforms data, and loads into Snowflake for Power BI reporting. It goes live in 72 hours. Walk through your preparation.

**Answer:**

**72 hours out — Technical readiness:**
- [ ] Pre-flight checks implemented and tested (connectivity, auth, schema validation)
- [ ] CI/CD pipeline tested end-to-end in QA environment (not just DEV)
- [ ] Auto-rollback mechanism tested (can we go back to previous state in < 15 min?)
- [ ] Secrets in Secrets Manager, rotation tested, all 3 pipelines fetch correctly
- [ ] Terraform IaC for Snowflake roles/schemas applied to PROD, peer-reviewed

**48 hours out — Security & compliance:**
- [ ] WIZ scan on PROD infrastructure — zero critical findings
- [ ] SoD review: who has access to what, sign-off documented in Confluence
- [ ] ORR meeting completed — all stakeholders signed off
- [ ] Change request raised in ServiceNow, scheduled in change calendar with ACS/GITC

**24 hours out — Observability:**
- [ ] New Relic dashboard live with SLI metrics (freshness, success rate, query latency)
- [ ] Alerts configured and tested (sent test alert to PagerDuty, correct team received it)
- [ ] Runbook published in Confluence: "Top 5 failure scenarios and how to fix them"
- [ ] CODEOWNERS file updated — platform team assigned as reviewers

**Go-live — Execute:**
- [ ] Run in parallel with any existing pipeline for 24 hours before cutting over
- [ ] Monitor New Relic intensively for first 4 hours post go-live
- [ ] Stakeholder communication: "Integration is live, here's the dashboard link"

**72 hours post go-live:**
- [ ] Review SLO compliance for first 3 days
- [ ] Document any surprises as lessons learned
- [ ] Close change request in ServiceNow

---

### 🎯 Scenario 5: SonarQube Quality Gate Fails — 47% Code Coverage

**Scenario:**  
A critical data pipeline hotfix needs to be deployed in 2 hours, but the SonarQube quality gate is blocking the PR with 47% code coverage (minimum 80%). The lead developer says "just bypass it this time." What do you do?

**Answer:**

**Immediate assessment:**
- Is the hotfix truly critical? (production down? data loss occurring?) vs. "nice to have urgently"
- If production is down: the 2-hour window is real pressure

**Do NOT silently bypass the gate.** Bypassing without process creates precedent and audit gaps.

**Options:**

**Option A — Fast coverage improvement:**
- Ask the developer: can they add targeted unit tests for just the new code (not the entire file)?
- If the hotfix is 20 lines of new code, adding tests for those 20 lines may be enough to move the gate
- This is the preferred path — takes 30–45 minutes if the logic is straightforward

**Option B — Formal exception with documented risk:**
- Raise a documented exception in Jira: "Coverage gate bypassed for hotfix [ID] — risk accepted by [Engineering Manager name]"
- Use the override mechanism (SonarQube admin can set a gate exception for a specific analysis)
- Commit to a follow-up ticket: "Add missing coverage for [module] — due within 5 business days"
- Post-deploy, create the tests before the next sprint ends

**What I would NOT do:**
- Silently turn off the quality gate (audit breach)
- Delete the SonarQube project temporarily (data loss)
- Merge the PR from a branch that skips CI entirely

**Communication to the developer:**  
"I understand the urgency. Let's take 30 minutes to add basic tests for the new code — that's better than creating technical debt and an audit gap. If we truly can't, I'll help you raise a formal exception with documented risk acceptance."

---

### 🎯 Scenario 6: Power BI Dashboard Showing Wrong Data

**Scenario:**  
A business team reports their Power BI sales dashboard shows numbers that are 20% lower than expected for yesterday. It's 9 AM on a Monday and leadership is asking questions. How do you investigate?

**Answer:**

**Triage (first 10 minutes):**
1. Check when the Snowflake data load last ran → `SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY`
2. Check if Power BI dataset refreshed successfully → Power BI Activity Log
3. Quickly check row count in Snowflake vs. what Power BI shows (are they the same? Is the issue in Snowflake or in Power BI?)
4. Check if this is a weekend data issue (did Saturday or Sunday data fail to load?)

**Layer-by-layer investigation:**

```
Business Report (wrong)
        ↑
Power BI Dataset → Did refresh succeed? Correct Snowflake endpoint?
        ↑
Snowflake Table → Is row count correct? Check LOAD_HISTORY for weekend
        ↑
Databricks/Glue Job → Did Saturday/Sunday runs complete? Check job run status
        ↑
Source Data (Kafka/S3) → Was source data actually produced over the weekend?
```

**Common root causes:**
- Weekend Databricks job failed silently (no alert configured for weekend runs)
- Snowflake warehouse auto-suspended and timed out, job partially loaded
- Power BI dataset refresh failed but no one configured failure alerts
- Date filter in Power BI dashboard incorrectly filtering out Sunday data

**Communication (within 30 minutes):**  
"We've identified the issue: the Saturday night Databricks job failed at 02:15 AM due to [cause], resulting in 2 days of missing data. We are reloading the data now — estimated completion 10:45 AM. We'll send a confirmation when Power BI reflects correct numbers."

**Fix and prevent:**
- Rerun the failed jobs, verify data completeness in Snowflake
- Add weekend alert configuration to all critical pipelines
- Add data freshness monitor in New Relic that alerts if data is > 6 hours stale (would have caught this at 8 AM)

---

### 🎯 Scenario 7: You're Asked to Bootstrap a New Repository for a Data Team

**Scenario:**  
A new data engineering team is joining the organization. They need a GitHub repository set up following platform standards. Walk through exactly what you do.

**Answer:**

**Step 1 — Gather requirements (30 min):**
- What will the repo contain? (PySpark notebooks, dbt models, Terraform, all?)
- What environments does it deploy to? (DEV/QA/PROD)
- Who are the team members and what are their roles?
- Which Snowflake databases/schemas will this team own?

**Step 2 — Create and configure the repo:**

```bash
# Create repo via GitHub CLI
gh repo create vwgdsi/data-team-alpha-pipeline \
  --private \
  --description "Data pipeline for Alpha team" \
  --team "data-engineers"

# Apply standard .github structure
cp -r ./platform-templates/.github/ ./data-team-alpha-pipeline/.github/

# Commit standard workflows
git add .github/workflows/
git commit -m "chore: seed platform CI/CD workflows"
git push
```

**Step 3 — Apply branch protection (via Python + GitHub API):**

```python
import requests

# Apply protection to main branch
protection_rules = {
    "required_status_checks": {
        "strict": True,
        "contexts": ["ci/lint-test", "sonarqube", "security-scan"]
    },
    "enforce_admins": False,
    "required_pull_request_reviews": {
        "required_approving_review_count": 2,
        "dismiss_stale_reviews": True,
        "require_code_owner_reviews": True
    },
    "restrictions": None,
    "required_signed_commits": True,
    "allow_force_pushes": False,
    "allow_deletions": False
}
requests.put(
    "https://api.github.com/repos/vwgdsi/data-team-alpha-pipeline/branches/main/protection",
    headers={"Authorization": f"Bearer {GITHUB_TOKEN}"},
    json=protection_rules
)
```

**Step 4 — Configure secrets and integrations:**
- Create repo-scoped secrets in GitHub (Snowflake credentials path, Databricks workspace URL)
- Linked to Secrets Manager via OIDC (no plaintext secrets stored in GitHub)
- Create SonarQube project and assign Python quality profile via SonarQube API

**Step 5 — Register and document:**
- Add repo entry to CMDB/Confluence with: purpose, team, CODEOWNERS, environments, dependencies
- Create Snowflake roles for this team using Terraform module (reviewed PR, applied to DEV first)
- Send onboarding checklist to team with: repo URL, CI/CD documentation link, runbook template, platform contact

---

## 9. Quick Reference Cheat Sheet

### AWS Services Map for This Role

| Need | Service |
|------|---------|
| Batch ETL | AWS Glue |
| Streaming | Kinesis Data Streams + Firehose |
| Orchestration | Step Functions |
| Serverless | Lambda |
| IaC state | S3 + DynamoDB |
| Secrets | Secrets Manager |
| Monitoring | CloudWatch |
| Security scanning | WIZ + AWS Security Hub |
| Cost analysis | Cost Explorer + AWS Budgets |
| Event-driven | EventBridge |
| Containers | EKS (Kubernetes) |

---

### Snowflake Key Commands

```sql
-- Check data freshness
SELECT MAX(LAST_LOAD_TIME) FROM INFORMATION_SCHEMA.LOAD_HISTORY WHERE TABLE_NAME = 'MY_TABLE';

-- Check expensive queries
SELECT QUERY_TEXT, TOTAL_ELAPSED_TIME/1000 AS seconds, CREDITS_USED_CLOUD_SERVICES
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE START_TIME > DATEADD(DAY, -7, CURRENT_TIMESTAMP())
ORDER BY TOTAL_ELAPSED_TIME DESC LIMIT 20;

-- Check role grants
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.GRANTS_TO_ROLES WHERE GRANTED_ON = 'TABLE';

-- Check login history (security audit)
SELECT USER_NAME, CLIENT_IP, IS_SUCCESS, ERROR_MESSAGE
FROM SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY
WHERE EVENT_TIMESTAMP > DATEADD(HOUR, -24, CURRENT_TIMESTAMP())
ORDER BY EVENT_TIMESTAMP DESC;
```

---

### Key Terms to Remember

| Domain | Must-Know Terms |
|--------|-----------------|
| **CI/CD** | GitFlow, trunk-based, CODEOWNERS, quality gates, OIDC, SonarQube |
| **AWS** | IAM Role, STS, PrivateLink, VPC injection, Control Tower, SCPs |
| **Snowflake** | Virtual Warehouse, Storage Integration, RBAC, Dynamic Masking, Snowpipe |
| **Databricks** | Delta Lake, Unity Catalog, AQE, Photon, Instance Profile, Backgrounder |
| **Observability** | SLI/SLO, error budget, MTTA, MTTR, three pillars, baseline alerting |
| **Security** | SoD, zero-trust, CSPM, WIZ, Secrets Manager rotation, ORR |
| **Tableau** | tabcmd, TSC Python, REST API, Performance Recording, Hyper API |
| **FinOps** | Cluster policies, resource monitors, chargeback, Savings Plans, CUR |

---

### STAR Framework for Behavioral Questions

Use this for any "Tell me about a time when..." question:

- **S**ituation — What was the context? (brief, 1-2 sentences)
- **T**ask — What was your specific responsibility?
- **A**ction — What did YOU do? (most detail here)
- **R**esult — What was the measurable outcome? (include numbers)

**Example:**  
*"Tell me about a time you reduced costs significantly."*

> **S:** Our Databricks spend exceeded budget by 3× in Q3 2025.  
> **T:** I was tasked with diagnosing and resolving the cost overrun without disrupting pipelines.  
> **A:** Audited all clusters via API, identified zombie clusters from a new ML team, implemented cluster policies enforcing auto-termination and spot instances, set up cost alerts.  
> **R:** Reduced monthly Databricks spend by 65% within 2 weeks while maintaining all SLOs. Documented the cluster policy template that is now standard across all teams.

---

*Last updated: February 2026 | Prepared for VW Group Digital Solutions India — Senior AWS DevOps Engineer Interview*