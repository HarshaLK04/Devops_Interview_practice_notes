# 🔷 Accenture — Custom Software Engineer (DevOps Focus)
## Interview Preparation Guide

> **Role:** Custom Software Engineer | **Location:** Bengaluru  
> **Experience:** 3+ Years | **Focus:** DevOps, CI/CD, Cloud, Containers

---

## 📋 Table of Contents

1. [Core DevOps Concepts](#1-core-devops-concepts)
2. [CI/CD & Automation](#2-cicd--automation)
3. [Cloud Services & Infrastructure](#3-cloud-services--infrastructure)
4. [Containerization (Docker & Kubernetes)](#4-containerization-docker--kubernetes)
5. [Infrastructure as Code](#5-infrastructure-as-code)
6. [Monitoring & Reliability](#6-monitoring--reliability)
7. [Agile & Collaboration](#7-agile--collaboration)
8. [Scenario-Based Problem Solving](#8-scenario-based-problem-solving)
9. [Interview Tips & Key Takeaways](#9-interview-tips--key-takeaways)

---

## Legend

| Badge | Level | What to Expect |
|-------|-------|----------------|
| 🟢 | **Basic** | Fundamental concepts, definitions, "what is" questions |
| 🟡 | **Intermediate** | Hands-on experience, "how would you implement" questions |
| 🔴 | **Hardcore** | Architecture, trade-offs, real production crisis scenarios |

---

## 1. Core DevOps Concepts

### 🟢 Basic

**Q1. What is DevOps and why did it emerge?**

**Answer:**  
DevOps is a cultural and technical movement that bridges the gap between Development (Dev) and Operations (Ops) teams. Traditionally, developers wrote code and "threw it over the wall" to operations, leading to friction, slow deployments, and blame games when things broke.

**Why DevOps emerged:**
- Traditional waterfall processes were too slow for modern business needs
- Separate Dev and Ops teams created silos and communication barriers
- Manual deployments led to errors, downtime, and slow time-to-market
- Need for continuous delivery of software (multiple releases per day instead of per quarter)

**Core DevOps principles:**
- **Collaboration:** Dev and Ops work as one team
- **Automation:** Automate repetitive tasks (testing, deployment, infrastructure provisioning)
- **Continuous Integration/Continuous Deployment (CI/CD):** Code changes flow automatically from commit to production
- **Monitoring & Feedback:** Continuous monitoring with fast feedback loops
- **Infrastructure as Code (IaC):** Treat infrastructure like application code

---

**Q2. What is the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?**

**Answer:**

| Term | Definition | Who Decides to Deploy? |
|------|------------|----------------------|
| **Continuous Integration (CI)** | Developers merge code to a shared repository multiple times per day. Every commit triggers automated build + test. | N/A (no deployment yet) |
| **Continuous Delivery (CD)** | Code is **always** in a deployable state. Deployment to production requires manual approval. | Human (one-click deploy) |
| **Continuous Deployment (CD)** | Every code change that passes automated tests is **automatically** deployed to production without human intervention. | Fully automated |

**Example:**
- **CI:** You commit code → Jenkins builds → unit tests run automatically
- **Continuous Delivery:** After CI passes → the artifact is ready to deploy, waiting for someone to click "Deploy to Production"
- **Continuous Deployment:** After CI passes → the artifact automatically deploys to production if all tests pass (no human gate)

---

**Q3. What is Infrastructure as Code (IaC) and why is it important?**

**Answer:**  
Infrastructure as Code means managing and provisioning infrastructure (servers, networks, storage) through machine-readable definition files rather than manual configuration via UI or CLI.

**Why it's important:**
- **Repeatability:** Same infrastructure can be spun up in dev, test, and prod environments identically
- **Version Control:** Infrastructure changes are tracked in Git like application code
- **Collaboration:** Team members can review infrastructure changes via pull requests
- **Disaster Recovery:** Can rebuild entire infrastructure from scratch if something fails
- **Consistency:** Eliminates "it works on my machine" for infrastructure

**Common IaC tools:**
- Terraform (cloud-agnostic, declarative)
- AWS CloudFormation (AWS-specific, declarative)
- Ansible (configuration management, imperative)
- Pulumi (uses general-purpose languages like Python/TypeScript)

**Example:** Instead of manually clicking through AWS console to create an EC2 instance, you write:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = { Name = "WebServer" }
}
```
Then run `terraform apply` and the instance is created automatically.

---

**Q4. Explain the concept of "Shift Left" in DevOps.**

**Answer:**  
"Shift Left" means moving tasks earlier (to the left) in the software development lifecycle to catch issues sooner when they're cheaper to fix.

**Traditional approach (Shift Right):**
```
Code → Build → Test (QA) → Deploy → Find bugs in production
                                      ↑ Most expensive to fix
```

**Shift Left approach:**
```
Code with unit tests → Build → Automated tests → Security scan → Deploy
  ↑ Cheapest to fix
```

**Examples of Shift Left:**
- **Security:** Security scanning in CI pipeline (SAST/DAST) instead of waiting for pen-test before production
- **Testing:** Developers write unit tests as they code instead of QA testing at the end
- **Quality:** SonarQube analyzing code quality on every commit instead of manual code review once a sprint
- **Infrastructure validation:** Terraform plan validation in CI instead of discovering IaC errors during deployment

**Benefits:**
- Faster feedback (bugs found in minutes, not weeks)
- Lower cost to fix (fixing a unit test failure costs less than rolling back a production deploy)
- Better quality software reaching production

---

### 🟡 Intermediate

**Q5. How would you implement a zero-downtime deployment strategy?**

**Answer:**  
Zero-downtime deployment means updating an application without service interruption. Common strategies:

**1. Blue-Green Deployment:**
```
Blue (v1.0 - current production) ← 100% traffic
Green (v1.1 - new version)       ← 0% traffic

Deploy v1.1 to Green environment
Run smoke tests on Green
Switch load balancer: 100% traffic → Green
Keep Blue running for rollback
After 24 hours with no issues, decommission Blue
```

**Pros:** Instant rollback (just switch traffic back to Blue)  
**Cons:** Requires 2× infrastructure capacity

**2. Rolling Deployment:**
```
10 servers running v1.0
Deploy v1.1 to 2 servers → test → deploy to 2 more → repeat
Gradual transition: v1.0 → v1.1 over 15-30 minutes
```

**Pros:** No extra infrastructure needed  
**Cons:** Both versions run simultaneously (must be backward compatible)

**3. Canary Deployment:**
```
Deploy v1.1 to 5% of servers
Monitor error rates, latency for 1 hour
If metrics look good → increase to 25%, then 50%, then 100%
If errors spike → rollback immediately
```

**Pros:** Risk mitigation (only 5% of users affected if bug exists)  
**Cons:** Requires sophisticated monitoring and traffic routing

**Implementation in Accenture context:**
- Use Kubernetes with rolling update strategy
- Implement health checks (liveness, readiness probes)
- Use service mesh (Istio) for canary traffic splitting
- Monitor with Prometheus/Grafana during deployment
- Automated rollback if error rate exceeds threshold

---

**Q6. How do you handle secrets management in a CI/CD pipeline?**

**Answer:**

**❌ NEVER do this:**
- Store secrets in Git (even in `.env` files)
- Hardcode credentials in code or Dockerfiles
- Pass secrets as environment variables in CI config files visible to all developers

**✅ Best practices:**

**1. Use a Secrets Management Service:**
- AWS Secrets Manager / Parameter Store
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager

**2. CI/CD Pipeline Integration:**

**Example with Jenkins + AWS Secrets Manager:**
```groovy
pipeline {
    agent any
    environment {
        DB_CREDS = credentials('aws-secret-db-credentials')
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    // Jenkins fetches secret from AWS at runtime
                    sh 'deploy.sh --db-user $DB_CREDS_USR --db-pass $DB_CREDS_PSW'
                }
            }
        }
    }
}
```

**Example with GitLab CI + HashiCorp Vault:**
```yaml
deploy:
  script:
    - export DB_PASSWORD=$(vault kv get -field=password secret/db)
    - kubectl apply -f deployment.yaml
  secrets:
    DB_PASSWORD:
      vault: secret/db/password
```

**3. Kubernetes Secrets:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: <base64-encoded-password>
---
# Pod uses the secret
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

**4. Secret Rotation:**
- Rotate secrets every 90 days automatically
- Use short-lived tokens (STS in AWS) instead of long-lived credentials
- Implement break-glass procedures for emergency access

---

### 🔴 Hardcore

**Q7. Design a complete CI/CD pipeline for a microservices application with 15 services. Explain your architecture and trade-offs.**

**Answer:**

**Architecture:**

```
Developer Commits → GitHub/GitLab
                         ↓
                    Webhook trigger
                         ↓
              ┌─────────────────────┐
              │   Jenkins/GitLab CI  │
              └─────────────────────┘
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   [Lint/SAST]    [Unit Tests]    [Build Docker]
        │                │                │
        └────────────────┴────────────────┘
                         ↓
              [Push to ECR/Docker Hub]
                         ↓
              [Deploy to Dev K8s (ArgoCD)]
                         ↓
           [Integration Tests (Selenium/Postman)]
                         ↓
              [Security Scan (Trivy/Snyk)]
                         ↓
         [Manual Approval Gate for Staging]
                         ↓
            [Deploy to Staging K8s]
                         ↓
           [Load Testing (K6/JMeter)]
                         ↓
     [Manual Approval Gate for Production]
                         ↓
       [Canary Deploy to Prod (5% → 100%)]
                         ↓
    [Monitor with Prometheus/Grafana/New Relic]
```

**Key Design Decisions:**

**1. Monorepo vs. Polyrepo for 15 microservices:**
- **Choice:** Polyrepo (one repo per service)
- **Reason:** Independent deployment cadence (service A can deploy 5x/day, service B 1x/week)
- **Trade-off:** More CI configuration overhead, harder to refactor across services

**2. Pipeline Orchestration:**
- **Choice:** Jenkins with shared libraries OR GitLab CI with includes
- **Reason:** Reusable pipeline templates across all 15 services
- **Template structure:**
```groovy
// Jenkinsfile (uses shared library)
@Library('accenture-devops-lib') _
microservicePipeline(
    serviceName: 'user-service',
    dockerfile: 'Dockerfile',
    deploymentYaml: 'k8s/deployment.yaml'
)
```

**3. Artifact Management:**
- Each service produces Docker image tagged with `${SERVICE_NAME}:${GIT_COMMIT_SHA}`
- Immutable tags (never overwrite `latest`)
- Clean up images older than 90 days

**4. Environment Promotion Strategy:**
- **Dev:** Auto-deploy on every commit to `main`
- **Staging:** Auto-deploy on Git tag (e.g., `v1.2.3-rc`)
- **Production:** Manual approval + canary deployment

**5. Service Dependencies:**
- Use contract testing (Pact) to catch breaking API changes before deployment
- Each service has its own CI pipeline but integration tests run in a separate pipeline that tests all 15 services together

**6. Monitoring & Rollback:**
- Each canary deployment monitored for 30 minutes
- Automated rollback if error rate > 1% or p95 latency > 500ms
- Prometheus alerts trigger rollback via Kubernetes API

**Trade-offs:**
- **Complexity:** 15 separate pipelines vs. 1 monolith pipeline — more overhead but more flexibility
- **Build time:** Parallel builds across services reduce total time but require more compute resources
- **Testing:** Integration tests are expensive (spin up all 15 services in test environment) — run nightly instead of per-commit

---

## 2. CI/CD & Automation

### 🟢 Basic

**Q1. What is Jenkins and how does it work?**

**Answer:**  
Jenkins is an open-source automation server used to implement CI/CD pipelines. It automates the build, test, and deployment process.

**How it works:**
1. **Trigger:** Developer pushes code to Git → Webhook notifies Jenkins
2. **Build:** Jenkins pulls code, runs build commands (e.g., `npm install`, `mvn package`)
3. **Test:** Runs automated tests (unit, integration)
4. **Artifact:** Creates deployable artifact (JAR, Docker image, etc.)
5. **Deploy:** Pushes artifact to target environment (dev, staging, prod)

**Key concepts:**
- **Job/Pipeline:** A sequence of automated steps
- **Agent/Node:** Machine where the job runs (master delegates work to agents)
- **Plugin:** Extends Jenkins functionality (Git plugin, Docker plugin, Kubernetes plugin)
- **Jenkinsfile:** Pipeline defined as code (declarative or scripted syntax)

**Example Jenkinsfile:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
```

---

**Q2. What is a webhook and how is it used in CI/CD?**

**Answer:**  
A webhook is an HTTP callback that sends real-time notifications when an event occurs. In CI/CD, webhooks automate pipeline triggers.

**How it works:**
```
Developer pushes code to GitHub
            ↓
GitHub sends POST request to Jenkins URL
            ↓
Jenkins receives payload (repo, branch, commit SHA)
            ↓
Jenkins triggers the pipeline automatically
```

**Example GitHub webhook payload:**
```json
{
  "ref": "refs/heads/main",
  "repository": {
    "name": "user-service",
    "url": "https://github.com/accenture/user-service"
  },
  "pusher": {
    "name": "developer1"
  },
  "commits": [
    {
      "id": "abc123",
      "message": "Fix bug in login"
    }
  ]
}
```

**Jenkins configuration:**
1. Install GitHub plugin
2. Create webhook in GitHub: `https://jenkins.company.com/github-webhook/`
3. Configure job to trigger on push events

**Benefits:**
- No polling (Jenkins doesn't need to check Git every minute)
- Instant feedback (pipeline starts within seconds of commit)
- Event-driven architecture

---

**Q3. What are the different types of testing in a CI/CD pipeline?**

**Answer:**

| Test Type | What it tests | When to run | Example |
|-----------|--------------|-------------|---------|
| **Unit Tests** | Individual functions/methods in isolation | Every commit | `assertEquals(add(2,3), 5)` |
| **Integration Tests** | Multiple components working together | Every commit or nightly | Test API + database interaction |
| **Contract Tests** | Service A's expectations of Service B's API | Every commit | Pact tests between microservices |
| **End-to-End (E2E) Tests** | Full user workflow through UI | Nightly or pre-release | Selenium: Login → Add to cart → Checkout |
| **Performance Tests** | System behavior under load | Pre-staging deployment | Load 1000 requests/sec for 10 minutes |
| **Security Tests** | Vulnerabilities in code/dependencies | Every commit | SAST (SonarQube), DAST (OWASP ZAP) |
| **Smoke Tests** | Basic functionality after deployment | Post-deployment | HTTP GET /health returns 200 OK |

**Test Pyramid (bottom = most tests, top = fewest):**
```
        /\
       /E2E\      ← Slowest, most expensive
      /─────\
     / Integ \    ← Medium speed
    /─────────\
   /   Unit    \  ← Fastest, cheapest
  /─────────────\
```

**In a CI pipeline:**
```yaml
stages:
  - lint
  - unit_test       # 2 minutes
  - build
  - integration_test # 10 minutes
  - deploy_to_dev
  - smoke_test      # 1 minute
  - (nightly) e2e_test # 45 minutes
```

---

### 🟡 Intermediate

**Q4. How do you implement automated rollback in a CI/CD pipeline?**

**Answer:**

**Strategy 1 — Health Check Based Rollback (Kubernetes):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  minReadySeconds: 30
  template:
    spec:
      containers:
      - name: app
        image: user-service:v2.0
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 3
          failureThreshold: 2
```

**How it works:**
- Kubernetes deploys new pods with v2.0
- If `/health` endpoint fails 3 times → pod marked unhealthy → Kubernetes stops rollout
- Old pods remain running → zero downtime
- Manual or automated command to rollback: `kubectl rollout undo deployment/user-service`

**Strategy 2 — Metric-Based Rollback (CI/CD Script):**

```python
import time
import requests

def deploy_and_monitor(service, version, canary_percentage=10):
    # Deploy canary
    deploy_canary(service, version, canary_percentage)
    
    # Monitor for 10 minutes
    for _ in range(10):
        metrics = get_prometheus_metrics(service)
        error_rate = metrics['error_rate']
        latency_p95 = metrics['latency_p95']
        
        if error_rate > 0.01:  # More than 1% errors
            print(f"Error rate {error_rate*100}% exceeded threshold")
            rollback(service, version)
            return False
        
        if latency_p95 > 500:  # More than 500ms p95 latency
            print(f"Latency {latency_p95}ms exceeded threshold")
            rollback(service, version)
            return False
        
        time.sleep(60)
    
    # If monitoring passed, promote to 100%
    deploy_full(service, version)
    return True
```

**Strategy 3 — GitOps Rollback (ArgoCD):**
- All deployments tracked in Git
- Rollback = revert Git commit
- ArgoCD automatically syncs cluster state to Git state

```bash
# Rollback last deployment
git revert HEAD
git push origin main
# ArgoCD detects change and rolls back automatically
```

---

**Q5. Explain how you would set up a multi-environment CI/CD pipeline (dev, staging, prod).**

**Answer:**

**Architecture:**

```
Source Code (Git)
       ↓
    [Build once, deploy everywhere]
       ↓
Docker Image: app:v1.2.3-abc123def
       ↓
    Push to Registry
       ↓
   ┌────┴────┬────────┬────────┐
   ↓         ↓        ↓        ↓
 DEV    STAGING     UAT      PROD
(auto)   (auto)   (manual) (manual)
```

**GitLab CI Example:**

```yaml
stages:
  - build
  - deploy_dev
  - deploy_staging
  - deploy_prod

variables:
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA

build:
  stage: build
  script:
    - docker build -t myapp:$IMAGE_TAG .
    - docker push myregistry/myapp:$IMAGE_TAG
  only:
    - main

deploy_dev:
  stage: deploy_dev
  script:
    - kubectl config use-context dev-cluster
    - kubectl set image deployment/myapp myapp=myregistry/myapp:$IMAGE_TAG
  environment:
    name: development
  only:
    - main

deploy_staging:
  stage: deploy_staging
  script:
    - kubectl config use-context staging-cluster
    - kubectl set image deployment/myapp myapp=myregistry/myapp:$IMAGE_TAG
  environment:
    name: staging
  only:
    - tags  # Only deploy staging on Git tags

deploy_prod:
  stage: deploy_prod
  script:
    - kubectl config use-context prod-cluster
    - kubectl set image deployment/myapp myapp=myregistry/myapp:$IMAGE_TAG
  environment:
    name: production
  when: manual  # Require manual approval
  only:
    - tags
```

**Environment Configuration Management:**

**Option 1 — Environment Variables:**
```yaml
# values-dev.yaml
database:
  host: dev-db.internal
  replicas: 1

# values-prod.yaml
database:
  host: prod-db.internal
  replicas: 3
  backups: enabled
```

**Option 2 — Git Branches per Environment:**
```
main → deploys to dev automatically
staging → deploys to staging automatically
production → deploys to prod (manual approval)
```

**Best practices:**
- Use the **same Docker image** across all environments (build once, configure via environment variables)
- **DEV:** Auto-deploy on every commit
- **Staging:** Auto-deploy on Git tags, run full test suite
- **Prod:** Manual approval gate + monitoring window
- Use **namespaces** in Kubernetes to isolate environments
- Separate **AWS accounts** or **GCP projects** for prod isolation

---

### 🔴 Hardcore

**Q6. You have a CI/CD pipeline that takes 45 minutes to run. Management wants it under 10 minutes. How do you optimize it?**

**Answer:**

**Step 1 — Profile the Pipeline (find bottlenecks):**

```
Current pipeline:
Lint:             2 min
Unit tests:       8 min  ← BOTTLENECK
Build Docker:     5 min
Integration tests: 20 min ← BOTTLENECK
Security scan:    7 min
Deploy:           3 min
Total:            45 min
```

**Step 2 — Optimization Strategies:**

**1. Parallelize Everything Possible:**

```yaml
# Before (sequential)
stages:
  - test
  - build
  - security

# After (parallel)
stages:
  - test_and_build  # Run unit tests and Docker build in parallel
  - security

test:
  stage: test_and_build
  script: npm test

build:
  stage: test_and_build
  script: docker build -t app:$TAG .
```

**2. Cache Dependencies Aggressively:**

```yaml
# Cache npm/Maven/pip dependencies between runs
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .m2/repository/
    - .pip/cache/

# In Docker, use multi-stage builds with layer caching
FROM node:18 AS builder
COPY package*.json ./
RUN npm ci  # This layer is cached if package.json unchanged
COPY . .
RUN npm run build
```

**3. Run Only Affected Tests (Monorepo):**

```bash
# Detect which services changed
CHANGED_SERVICES=$(git diff HEAD~1 --name-only | grep "^services/" | cut -d'/' -f2 | sort -u)

# Run tests only for changed services
for service in $CHANGED_SERVICES; do
  cd services/$service
  npm test
done
```

**4. Distributed Testing:**

```yaml
# Split integration tests across 4 parallel jobs
integration_test:
  parallel: 4
  script:
    - npm run test:integration -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
```

**5. Move Slow Tests to Nightly Builds:**

```yaml
# Fast tests (every commit)
unit_test:
  script: npm test -- --testPathPattern=unit

# Slow E2E tests (nightly only)
e2e_test:
  script: npm run test:e2e
  only:
    - schedules  # Run only on nightly cron job
```

**6. Incremental Builds:**

```dockerfile
# Use BuildKit cache mounts
# syntax=docker/dockerfile:1
FROM node:18
RUN --mount=type=cache,target=/root/.npm npm ci
```

**Expected Result After Optimization:**

```
Optimized pipeline:
Lint (parallel):           1 min
Unit tests (parallel):     3 min  ← Parallelized + cached deps
Build Docker (parallel):   2 min  ← Layer caching
Integration tests:         5 min  ← Only affected tests
Security scan (parallel):  2 min
Deploy:                    1 min
Total:                    ~9 min ← Under 10 minutes!
```

**Trade-offs:**
- More complex pipeline configuration
- Higher resource usage (parallel jobs need more CPU/memory)
- Nightly builds are now critical (catch regressions not caught in fast pipeline)

---

## 3. Cloud Services & Infrastructure

### 🟢 Basic

**Q1. Compare AWS, Azure, and GCP — which would you choose for a new project?**

**Answer:**

| Aspect | AWS | Azure | GCP |
|--------|-----|-------|-----|
| **Market Share** | ~32% (largest) | ~23% | ~10% |
| **Strengths** | Mature, widest service catalog, strong enterprise support | Best for Microsoft shops, excellent hybrid cloud (Azure Arc), tight Office 365 integration | Best ML/AI tools, BigQuery, Kubernetes (GKE) is native, simpler pricing |
| **Compute** | EC2 (flexible) | Virtual Machines | Compute Engine |
| **Kubernetes** | EKS (managed) | AKS (managed) | GKE (most mature, auto-upgrade, auto-repair) |
| **Serverless** | Lambda | Azure Functions | Cloud Functions / Cloud Run |
| **Database** | RDS, Aurora, DynamoDB | SQL Database, Cosmos DB | Cloud SQL, Spanner, Firestore |
| **CI/CD Native** | CodePipeline | Azure DevOps | Cloud Build |

**Decision criteria for Accenture projects:**

**Choose AWS if:**
- Client has no existing cloud preference
- Widest range of services needed (AWS has ~200+ services)
- Need mature enterprise support and case studies

**Choose Azure if:**
- Client is already using Microsoft stack (.NET, SQL Server, Active Directory)
- Hybrid cloud requirement (on-prem + cloud)
- Enterprise agreement with Microsoft already in place

**Choose GCP if:**
- Client prioritizes data analytics / ML (BigQuery, Vertex AI)
- Kubernetes-first architecture (GKE is the gold standard)
- Cleaner, simpler APIs and pricing model

**My recommendation for most Accenture projects:** Start with **AWS** unless there's a strong reason otherwise (existing Azure/GCP commitment, specific technology need).

---

**Q2. What is the difference between IaaS, PaaS, and SaaS? Give examples.**

**Answer:**

| Model | What You Manage | What Cloud Provider Manages | Example |
|-------|----------------|---------------------------|---------|
| **IaaS** (Infrastructure) | OS, runtime, app, data | Physical hardware, virtualization, networking, storage | AWS EC2, Azure VMs |
| **PaaS** (Platform) | Application code, data | OS, runtime, middleware, scaling, patching | AWS Elastic Beanstalk, Google App Engine, Heroku |
| **SaaS** (Software) | Just use the app | Everything | Gmail, Salesforce, Office 365 |

**Analogy — Pizza as a Service:**

```
On-Premises:  You make everything (dough, sauce, oven, etc.)
IaaS:         You get a kitchen, you cook the pizza
PaaS:         You get pizza ingredients + oven, you just assemble and cook
SaaS:         You order pizza delivery, just eat it
```

**DevOps relevance:**
- **IaaS:** You manage the entire CI/CD pipeline, install Jenkins on EC2, configure everything
- **PaaS:** Platform handles scaling, you just push code (e.g., AWS CodePipeline, Azure DevOps)
- **SaaS:** Fully managed services (e.g., GitHub Actions, CircleCI)

**Trend:** DevOps is moving toward **PaaS** and **serverless** to reduce operational overhead — focus on code, not infrastructure management.

---

**Q3. What is an AWS VPC and why do you need it?**

**Answer:**  
AWS VPC (Virtual Private Cloud) is a logically isolated network within AWS where you can launch resources like EC2 instances, RDS databases, and load balancers.

**Why you need it:**
- **Security:** Isolate your resources from the public internet
- **Network control:** Define your own IP address range (CIDR), subnets, route tables
- **Compliance:** Meet regulatory requirements (e.g., healthcare data must not be on public internet)

**VPC Components:**

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   ├── Internet Gateway (allows outbound internet)
│   └── NAT Gateway (allows private subnet internet access)
├── Private Subnet (10.0.2.0/24)
│   ├── EC2 instances (no public IP)
│   └── RDS database (not accessible from internet)
└── Security Groups (firewall rules)
```

**Example use case:**
- Web servers in **public subnet** (need to receive traffic from internet)
- Application servers in **private subnet** (only accessible from web servers)
- Database in **private subnet** (only accessible from app servers)

**DevOps relevance:**
- VPC is created via Terraform (IaC)
- CI/CD pipelines run in private subnets for security
- Load balancers in public subnet route to private instances

---

### 🟡 Intermediate

**Q4. How would you design a highly available and scalable architecture on AWS?**

**Answer:**

**Requirements:**
- **High Availability:** System stays up even if an entire data center fails
- **Scalability:** Handles increasing load automatically
- **Cost-Effective:** Pay only for what you use

**Architecture:**

```
Route 53 (DNS)
       ↓
CloudFront (CDN for static assets)
       ↓
Application Load Balancer (multi-AZ)
       ↓
┌──────────┴──────────┬──────────────┐
↓                     ↓              ↓
ECS/EKS (us-east-1a)  (1b)         (1c)  ← Multi-AZ
  Auto Scaling Group (2-10 instances)
       ↓
RDS Multi-AZ (primary in 1a, standby in 1b)
       ↓
S3 (99.999999999% durability)
```

**Key Decisions:**

**1. Multi-AZ Deployment:**
- Deploy across at least 3 Availability Zones (AZs are separate data centers)
- If one AZ fails, traffic routes to healthy AZs automatically

**2. Auto Scaling:**
```yaml
# Kubernetes Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**3. Database:**
- **RDS Multi-AZ:** Automatic failover to standby in <2 minutes
- **Read replicas:** Offload read traffic (analytics, reporting)
- **Backup:** Automated daily snapshots + point-in-time recovery

**4. Static Content:**
- Store in S3, serve via CloudFront CDN
- Reduces load on application servers
- Low latency for global users

**5. Monitoring & Alerts:**
- CloudWatch alarms for CPU, memory, disk, error rates
- Auto-scaling triggered by CloudWatch metrics
- PagerDuty alerts for critical failures

**Cost Optimization:**
- Use Spot Instances for non-critical workloads (70-90% cheaper)
- Right-size instances (don't use t3.2xlarge if t3.medium is enough)
- Use S3 lifecycle policies (move old data to Glacier after 90 days)

---

**Q5. Explain Infrastructure as Code with a Terraform example for deploying a web application.**

**Answer:**

**Scenario:** Deploy a simple web app on AWS with auto-scaling.

**File structure:**
```
terraform/
├── main.tf         # Main infrastructure
├── variables.tf    # Input variables
├── outputs.tf      # Output values
└── terraform.tfvars # Variable values
```

**main.tf:**
```hcl
provider "aws" {
  region = var.aws_region
}

# VPC and Networking
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "${var.project_name}-vpc" }
}

resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  subnets            = aws_subnet.public[*].id
  security_groups    = [aws_security_group.alb.id]
}

resource "aws_lb_target_group" "app" {
  name     = "${var.project_name}-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 10
  }
}

# Auto Scaling Group
resource "aws_launch_template" "app" {
  name_prefix   = "${var.project_name}-"
  image_id      = var.ami_id
  instance_type = var.instance_type

  user_data = base64encode(<<-EOF
              #!/bin/bash
              docker run -d -p 80:8080 ${var.docker_image}
              EOF
  )
}

resource "aws_autoscaling_group" "app" {
  desired_capacity    = 2
  max_size            = 10
  min_size            = 2
  target_group_arns   = [aws_lb_target_group.app.arn]
  vpc_zone_identifier = aws_subnet.public[*].id

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }
}

# Auto Scaling Policy
resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale-up"
  scaling_adjustment     = 2
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.app.name
}

# CloudWatch Alarm (trigger scale up when CPU > 70%)
resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "cpu-utilization-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 70
  alarm_actions       = [aws_autoscaling_policy.scale_up.arn]

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }
}
```

**Usage:**
```bash
terraform init
terraform plan
terraform apply
```

**Benefits of this IaC approach:**
- **Reproducible:** Can recreate entire stack in minutes
- **Version controlled:** Changes tracked in Git
- **Reviewable:** Team reviews infrastructure changes via PR
- **Multi-environment:** Use different `.tfvars` for dev/staging/prod

---

### 🔴 Hardcore

**Q6. You're asked to migrate a monolithic application to a microservices architecture on Kubernetes. Explain your migration strategy.**

**Answer:**

**Phase 1 — Assessment & Planning (Week 1-2):**

**1. Identify Bounded Contexts:**
```
Monolith:
┌──────────────────────┐
│  User Management     │
│  Product Catalog     │
│  Order Processing    │
│  Payment Gateway     │
│  Notification Service│
└──────────────────────┘

Microservices:
┌─────────┐ ┌──────────┐ ┌───────┐ ┌─────────┐ ┌──────────────┐
│  Users  │ │ Products │ │Orders │ │ Payment │ │Notifications │
└─────────┘ └──────────┘ └───────┘ └─────────┘ └──────────────┘
```

**2. Dependency Mapping:**
- Which modules talk to which?
- What are the database dependencies?
- Can services be extracted independently?

**Phase 2 — Strangler Fig Pattern (Months 1-6):**

**Step 1:** Extract the **easiest** service first (least dependencies)

```
           ┌─────────────┐
Internet → │ API Gateway │
           └─────────────┘
                  ↓
          ┌───────┴───────┐
          ↓               ↓
  ┌──────────────┐   ┌──────────┐
  │  Monolith    │   │Notification│ ← New microservice
  │  (90% code)  │   │ Service    │
  └──────────────┘   └──────────┘
```

**Step 2:** Route notification requests to new service, rest to monolith

```nginx
# API Gateway routing
location /api/notifications {
    proxy_pass http://notification-service:8080;
}

location /api {
    proxy_pass http://monolith:8080;
}
```

**Step 3:** Repeat for each service (one per month)

**Phase 3 — Data Migration Strategy:**

**Option A — Shared Database (transitional):**
```
┌─────────┐ ┌──────────┐
│ Service │ │ Service  │
│   A     │ │    B     │
└────┬────┘ └────┬─────┘
     │           │
     └─────┬─────┘
           ↓
     [Shared DB]  ← Risk: tight coupling
```

**Option B — Database per Service (target state):**
```
┌─────────┐      ┌──────────┐
│ Service │      │ Service  │
│   A     │      │    B     │
└────┬────┘      └────┬─────┘
     │                │
     ↓                ↓
 [DB A]             [DB B]
     ↑                ↑
     └────────────────┘
      API calls between services
```

**Migration approach:**
1. Duplicate data temporarily (read from shared DB, write to both)
2. Gradually migrate reads to service-specific DB
3. Eventually cut over fully and remove shared DB access

**Phase 4 — Kubernetes Deployment:**

```yaml
# notification-service/k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notification-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: notification-service
  template:
    metadata:
      labels:
        app: notification-service
        version: v1.0.0
    spec:
      containers:
      - name: app
        image: accenture/notification-service:v1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: notification-db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: notification-service
spec:
  selector:
    app: notification-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

**Phase 5 — Observability & Monitoring:**

```
Install service mesh (Istio/Linkerd)
        ↓
Distributed tracing (Jaeger)
        ↓
Centralized logging (ELK stack)
        ↓
Metrics (Prometheus + Grafana)
        ↓
Alerts (PagerDuty)
```

**Challenges & Solutions:**

| Challenge | Solution |
|-----------|----------|
| **Data consistency across services** | Use Saga pattern or event-driven architecture (Kafka) |
| **Network latency** | Service mesh with circuit breakers, retries, timeouts |
| **Testing complexity** | Contract testing (Pact), chaos engineering (simulate failures) |
| **Operational overhead** | GitOps (ArgoCD), centralized logging, auto-scaling |

**Timeline:** 6-12 months depending on monolith size and team capacity.

---

## 4. Containerization (Docker & Kubernetes)

### 🟢 Basic

**Q1. What is Docker and why is it used?**

**Answer:**  
Docker is a platform for building, shipping, and running applications in **containers**. A container packages an application with all its dependencies (libraries, runtime, system tools) so it runs consistently across different environments.

**Why Docker?**
- **"Works on my machine" problem solved:** Code that runs on developer's laptop will run identically in production
- **Lightweight:** Containers share the host OS kernel (unlike VMs which need full OS per instance)
- **Fast startup:** Containers start in seconds (vs. minutes for VMs)
- **Consistent environments:** Dev, test, and prod use the exact same Docker image

**Docker vs. Virtual Machine:**

```
Virtual Machines:
┌─────────┬─────────┬─────────┐
│ App A   │ App B   │ App C   │
├─────────┼─────────┼─────────┤
│ Guest OS│ Guest OS│ Guest OS│ ← Each VM has full OS (GBs of overhead)
├─────────┴─────────┴─────────┤
│      Hypervisor             │
├─────────────────────────────┤
│      Host OS                │
└─────────────────────────────┘

Docker Containers:
┌─────────┬─────────┬─────────┐
│ App A   │ App B   │ App C   │
├─────────┴─────────┴─────────┤
│    Docker Engine            │ ← Containers share OS kernel
├─────────────────────────────┤
│      Host OS                │
└─────────────────────────────┘
```

**Basic Docker Commands:**
```bash
# Build an image
docker build -t myapp:v1.0 .

# Run a container
docker run -d -p 8080:80 myapp:v1.0

# List running containers
docker ps

# Stop a container
docker stop <container_id>

# View logs
docker logs <container_id>
```

---

**Q2. What is Kubernetes and why do we need it?**

**Answer:**  
Kubernetes (K8s) is an **orchestration platform** for managing containerized applications at scale. While Docker runs containers on a single machine, Kubernetes manages containers across a **cluster** of machines.

**Why Kubernetes?**
- **Auto-scaling:** Automatically add/remove containers based on load
- **Self-healing:** If a container crashes, K8s restarts it automatically
- **Load balancing:** Distributes traffic across healthy containers
- **Rolling updates:** Deploy new versions with zero downtime
- **Service discovery:** Containers find each other automatically

**Without Kubernetes (manual):**
```
Developer deploys containers to 10 servers manually
One server crashes → app goes down
Traffic spike → manually add more servers
Need to update app → manually update each server
```

**With Kubernetes:**
```
Developer: kubectl apply -f deployment.yaml
Kubernetes:
- Distributes containers across cluster
- Monitors health, restarts crashed containers
- Auto-scales based on CPU usage
- Rolling update with zero downtime
```

**Key Kubernetes Concepts:**

| Concept | What it is |
|---------|-----------|
| **Pod** | Smallest deployable unit (1+ containers) |
| **Deployment** | Manages desired state (e.g., "run 5 replicas") |
| **Service** | Stable endpoint to access pods (load balancer) |
| **ConfigMap** | Store configuration (non-sensitive) |
| **Secret** | Store sensitive data (passwords, API keys) |
| **Namespace** | Logical isolation (dev, staging, prod) |

---

**Q3. Write a simple Dockerfile for a Node.js application.**

**Answer:**

```dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port 3000
EXPOSE 3000

# Start the application
CMD ["node", "server.js"]
```

**Best Practices Applied:**

1. **Use Alpine image:** `node:18-alpine` is much smaller (50MB vs 900MB for full image)
2. **Layer caching:** Copy `package.json` first, then `RUN npm ci`, so dependencies are cached unless `package.json` changes
3. **Production dependencies only:** `npm ci --only=production` excludes dev dependencies
4. **Non-root user:** Add this for security:
```dockerfile
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs
```

**Build and run:**
```bash
docker build -t myapp:v1.0 .
docker run -d -p 3000:3000 --name myapp-container myapp:v1.0
```

---

### 🟡 Intermediate

**Q4. Explain Kubernetes Deployments, Services, and Ingress with an example.**

**Answer:**

**Scenario:** Deploy a web application with load balancing and external access.

**1. Deployment (manages pods):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3  # Run 3 copies
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:v1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

**2. Service (load balancer for pods):**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp  # Routes traffic to pods with this label
  ports:
  - protocol: TCP
    port: 80        # Service listens on port 80
    targetPort: 8080 # Forwards to container port 8080
  type: ClusterIP   # Internal to cluster only
```

**3. Ingress (external access with routing):**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.accenture.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```

**How it works:**

```
Internet
    ↓
Ingress Controller (nginx)
    ↓ (routes myapp.accenture.com)
Service (webapp-service)
    ↓ (load balances)
┌─────────┬─────────┬─────────┐
│  Pod 1  │  Pod 2  │  Pod 3  │
└─────────┴─────────┴─────────┘
```

**Deploy everything:**
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

**Q5. How do you handle configuration and secrets in Kubernetes?**

**Answer:**

**ConfigMap (non-sensitive configuration):**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "db.accenture.internal"
  LOG_LEVEL: "info"
  FEATURE_FLAG_NEW_UI: "true"
---
# Use ConfigMap in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  template:
    spec:
      containers:
      - name: webapp
        image: myapp:v1.0
        envFrom:
        - configMapRef:
            name: app-config  # Inject all keys as environment variables
```

**Secret (sensitive data):**

```bash
# Create secret from literal values
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=SuperSecret123

# Or from file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=./server.crt \
  --from-file=tls.key=./server.key
```

```yaml
# Use Secret in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  template:
    spec:
      containers:
      - name: webapp
        image: myapp:v1.0
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

**Best Practices:**

1. **Never commit secrets to Git**
2. **Encrypt secrets at rest:** Enable encryption in K8s etcd
3. **Use external secret management:**
   - AWS Secrets Manager + External Secrets Operator
   - HashiCorp Vault + Vault Agent Injector
4. **Least privilege:** RBAC to control who can read secrets
5. **Rotate secrets regularly:** Automate rotation every 90 days

**Example with External Secrets Operator:**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: db-secret
  data:
  - secretKey: password
    remoteRef:
      key: prod/db/password  # Fetches from AWS Secrets Manager
```

---

### 🔴 Hardcore

**Q6. Design a complete production-ready Kubernetes cluster architecture for a high-traffic application.**

**Answer:**

**Architecture:**

```
┌─────────────────────── Production Cluster ───────────────────────┐
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Ingress Controller (NGINX) - 3 replicas (HA)           │    │
│  │  + WAF (ModSecurity)                                     │    │
│  │  + Rate Limiting (100 req/sec per IP)                   │    │
│  │  + TLS termination                                       │    │
│  └──────────────────────┬──────────────────────────────────┘    │
│                         │                                         │
│  ┌──────────────────────┴────────────┐                           │
│  │      Service Mesh (Istio)         │                           │
│  │  - mTLS between all services      │                           │
│  │  - Circuit breaker, retries       │                           │
│  │  - Distributed tracing (Jaeger)   │                           │
│  └──────────────────┬────────────────┘                           │
│                     │                                             │
│  ┌──────────────────┴─────────────────────────────────────────┐ │
│  │                Application Namespaces                       │ │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐           │ │
│  │  │ API Gateway│  │ User Service│  │Auth Service│           │ │
│  │  │ (3 replicas)│  │ (5 replicas)│  │(3 replicas)│           │ │
│  │  └────────────┘  └────────────┘  └────────────┘           │ │
│  │  HPA (2-20 replicas based on CPU/memory/custom metrics)   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌───────────────── Data Layer ─────────────────┐               │
│  │  - PostgreSQL (RDS Multi-AZ)                 │               │
│  │  - Redis Cluster (3 masters, 3 replicas)     │               │
│  │  - Kafka (3 brokers for event streaming)     │               │
│  └──────────────────────────────────────────────┘               │
│                                                                   │
│  ┌───────────────── Observability ──────────────┐               │
│  │  - Prometheus (metrics, 30-day retention)    │               │
│  │  - Grafana (dashboards)                      │               │
│  │  - ELK Stack (logs, 7-day retention)         │               │
│  │  - Jaeger (distributed tracing)              │               │
│  │  - Alert Manager → PagerDuty                 │               │
│  └──────────────────────────────────────────────┘               │
│                                                                   │
│  ┌───────────────── Node Configuration ─────────────────┐       │
│  │  Node Pool 1 (application): t3.xlarge (10 nodes)     │       │
│  │  Node Pool 2 (data): r5.2xlarge (3 nodes)            │       │
│  │  Node Pool 3 (monitoring): t3.large (2 nodes)        │       │
│  │  All nodes: Auto Scaling Group (ASG), spot instances │       │
│  └───────────────────────────────────────────────────────┘       │
└───────────────────────────────────────────────────────────────────┘
```

**Key Design Decisions:**

**1. Multi-Zone Deployment:**
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: user-service
      topologyKey: topology.kubernetes.io/zone
# Ensures replicas spread across availability zones
```

**2. Resource Management:**
```yaml
resources:
  requests:  # Guaranteed
    memory: "512Mi"
    cpu: "500m"
  limits:    # Maximum allowed
    memory: "1Gi"
    cpu: "1000m"

# + Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: user-service-pdb
spec:
  minAvailable: 2  # Always keep at least 2 replicas running
  selector:
    matchLabels:
      app: user-service
```

**3. Auto-Scaling (3 levels):**

**a) Horizontal Pod Autoscaler (pods):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

**b) Vertical Pod Autoscaler (resource limits):**
- Automatically adjusts CPU/memory requests based on usage
- Prevents over-provisioning

**c) Cluster Autoscaler (nodes):**
- Adds nodes when pods can't be scheduled
- Removes underutilized nodes

**4. Security:**

**a) Network Policies:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-policy
spec:
  podSelector:
    matchLabels:
      app: api-gateway
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: user-service
    ports:
    - protocol: TCP
      port: 8080
```

**b) Pod Security Standards:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

**5. Disaster Recovery:**

**Backup Strategy:**
- Velero for cluster backup (daily, 30-day retention)
- Database snapshots every 6 hours
- Etcd backups every hour

**RTO/RPO:**
- RTO (Recovery Time Objective): < 15 minutes
- RPO (Recovery Point Objective): < 1 hour

**6. CI/CD Integration:**

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: user-service
spec:
  project: production
  source:
    repoURL: https://github.com/accenture/k8s-manifests
    targetRevision: main
    path: user-service/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

**Cost Optimization:**
- 70% spot instances for non-critical workloads
- Reserved instances for core services (20% cost saving)
- Karpenter for intelligent node provisioning
- Cluster autoscaler scales down during off-peak

---

## 5. Infrastructure as Code

### 🟢 Basic

**Q1. What is the difference between declarative and imperative IaC?**

**Answer:**

| Approach | What you specify | How it works | Example |
|----------|------------------|--------------|---------|
| **Declarative** | Desired end state | Tool figures out how to achieve it | Terraform, CloudFormation |
| **Imperative** | Step-by-step instructions | You specify exact commands | Bash scripts, Ansible playbooks |

**Example:**

**Declarative (Terraform):**
```hcl
# Desired state: "I want 3 EC2 instances"
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-abc123"
  instance_type = "t2.micro"
}
# Terraform figures out what to create/update/delete
```

**Imperative (Bash):**
```bash
# Step-by-step: "Create exactly 3 instances"
for i in {1..3}; do
  aws ec2 run-instances \
    --image-id ami-abc123 \
    --instance-type t2.micro
done
# You specify the exact API calls
```

**Idempotency:**
- **Declarative:** Running terraform apply multiple times produces the same result (safe)
- **Imperative:** Running the bash script 3 times creates 9 instances (not safe)

**When to use each:**
- **Declarative:** Infrastructure provisioning (Terraform for cloud resources)
- **Imperative:** Configuration management, complex logic (Ansible for server config)

---

**Q2. What is Terraform state and why is it important?**

**Answer:**  
Terraform state is a JSON file that tracks the **current** state of your infrastructure (what actually exists in AWS/Azure/GCP) and maps it to your Terraform configuration.

**Why state is critical:**

1. **Knows what exists:** Terraform compares desired (code) vs. actual (state) to determine what to create/update/delete
2. **Performance:** Doesn't need to query cloud provider for every resource on every run
3. **Metadata:** Stores resource dependencies and relationships

**State file example (`terraform.tfstate`):**
```json
{
  "version": 4,
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "attributes": {
            "id": "i-abc123def456",
            "ami": "ami-abc123",
            "instance_type": "t2.micro",
            "public_ip": "54.123.45.67"
          }
        }
      ]
    }
  ]
}
```

**Remote State (production best practice):**

```hcl
terraform {
  backend "s3" {
    bucket         = "accenture-terraform-state"
    key            = "prod/web-app/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # State locking
  }
}
```

**Benefits of remote state:**
- **Team collaboration:** Multiple engineers use the same state
- **State locking:** Prevents simultaneous changes (via DynamoDB lock)
- **Security:** S3 encryption protects sensitive data in state
- **Backup:** S3 versioning allows rollback

---

### 🟡 Intermediate

**Q3. How do you manage multiple environments (dev/staging/prod) in Terraform?**

**Answer:**

**Strategy 1 — Workspaces (simple projects):**

```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch between environments
terraform workspace select prod
terraform apply

# In code, reference workspace
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  tags = {
    Environment = terraform.workspace
  }
}
```

**Pros:** Simple, built-in  
**Cons:** Hard to manage different configurations per environment

---

**Strategy 2 — Directory Structure (recommended):**

```
terraform/
├── modules/
│   └── web-app/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       └── terraform.tfvars
```

**Module (reusable):**
```hcl
# modules/web-app/main.tf
variable "environment" {}
variable "instance_type" {}
variable "instance_count" {}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  tags = {
    Environment = var.environment
  }
}
```

**Environment-specific config:**
```hcl
# environments/prod/main.tf
module "web_app" {
  source = "../../modules/web-app"
  
  environment    = "prod"
  instance_type  = "t3.large"
  instance_count = 5
}

# environments/prod/terraform.tfvars
aws_region = "ap-south-1"
vpc_cidr   = "10.0.0.0/16"
```

**Usage:**
```bash
cd environments/prod
terraform init
terraform plan
terraform apply
```

---

**Strategy 3 — Terragrunt (DRY for complex setups):**

```hcl
# terragrunt.hcl (shared config)
remote_state {
  backend = "s3"
  config = {
    bucket = "accenture-tf-state-${get_aws_account_id()}"
    key    = "${path_relative_to_include()}/terraform.tfstate"
    region = "ap-south-1"
    encrypt = true
  }
}

# environments/prod/terragrunt.hcl
include {
  path = find_in_parent_folders()
}

inputs = {
  environment    = "prod"
  instance_type  = "t3.large"
  instance_count = 5
}
```

---

### 🔴 Hardcore

**Q4. You need to migrate 50+ legacy manually-created AWS resources to Terraform without downtime. Explain your approach.**

**Answer:**

**Phase 1 — Discovery & Inventory (Week 1):**

**1. Automated scanning:**
```bash
# Use AWS Config to inventory all resources
aws configservice get-discovered-resource-counts --region ap-south-1

# Export to JSON
aws resourcegroupstaggingapi get-resources \
  --region ap-south-1 \
  --resource-type-filters "AWS::EC2::Instance" "AWS::RDS::DBInstance" \
  > inventory.json
```

**2. Use `terraformer` to import existing resources:**
```bash
# Install terraformer
brew install terraformer

# Import all EC2 instances
terraformer import aws --resources=ec2_instance --regions=ap-south-1 --profile=prod

# This generates .tf files for existing resources
```

**Phase 2 — Terraform Import (Weeks 2-3):**

**Manual import for critical resources:**
```bash
# Import existing VPC
terraform import aws_vpc.main vpc-abc123

# Import existing EC2 instance
terraform import aws_instance.web[0] i-xyz789

# Import existing RDS database
terraform import aws_db_instance.postgres db-instance-id
```

**Parallel state file:**
```hcl
# Create NEW state file (don't touch existing resources yet)
terraform {
  backend "s3" {
    bucket = "accenture-tf-state"
    key    = "imported/terraform.tfstate"  # Separate from existing state
    region = "ap-south-1"
  }
}
```

**Phase 3 — Validation (Week 4):**

**1. Terraform plan should show "No changes":**
```bash
terraform plan
# Expected output:
# No changes. Infrastructure is up-to-date.
```

**2. If plan shows changes:**
- Adjust Terraform code to match actual resource configuration
- Use `terraform refresh` to sync state with reality
- Iterate until `terraform plan` shows zero changes

**Phase 4 — Controlled Cutover (Week 5):**

**1. Enable maintenance window (optional):**
- Schedule during low-traffic period
- Notify stakeholders

**2. Test apply on non-critical resource first:**
```bash
# Target a specific resource
terraform apply -target=aws_s3_bucket.logs
```

**3. Full apply:**
```bash
terraform apply
# Terraform should apply zero changes (already imported)
```

**4. Post-cutover validation:**
```bash
# Verify all resources still running
aws ec2 describe-instances --instance-ids i-xyz789
curl https://app.accenture.com/health
```

**Phase 5 — Future Changes via Terraform Only (Week 6+):**

**1. Block manual changes:**
```python
# AWS Config rule: Alert if resources modified outside Terraform
{
  "ConfigRuleName": "terraform-managed-only",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "REQUIRED_TAGS"
  },
  "Scope": {
    "ComplianceResourceTypes": [
      "AWS::EC2::Instance",
      "AWS::RDS::DBInstance"
    ]
  },
  "RequiredTags": {
    "ManagedBy": "Terraform"
  }
}
```

**2. Update documentation:**
- All infrastructure changes must go through Terraform PR
- Runbook: "How to add a new EC2 instance" → Terraform module

**Challenges & Solutions:**

| Challenge | Solution |
|-----------|----------|
| **Import errors (resource not found)** | Resource was deleted — remove from Terraform code |
| **State conflicts (resource exists in multiple places)** | Use `terraform state rm` to clean up duplicates |
| **Drift detection** | Run `terraform plan` daily in CI, alert on drift |
| **Team adoption** | Training session + pair programming for first month |

---

## 6. Monitoring & Reliability

### 🟢 Basic

**Q1. What is the difference between monitoring and observability?**

**Answer:**

| Aspect | Monitoring | Observability |
|--------|-----------|---------------|
| **Focus** | Known failures | Unknown failures |
| **Question** | "Is the system broken?" | "Why is the system broken?" |
| **Data** | Predefined metrics + alerts | Metrics + logs + traces (3 pillars) |
| **Example** | Alert: CPU > 80% | Trace: Request slow because DB query X took 5s due to missing index on table Y |

**The Three Pillars of Observability:**

1. **Metrics:** Time-series numerical data (CPU %, request/sec, error rate)
2. **Logs:** Timestamped event records ("User login failed", "Payment processed")
3. **Traces:** End-to-end request path (frontend → API → database → response)

**Example:**

**Monitoring approach:**
```
Prometheus alert: Error rate > 5%
    ↓
Page on-call engineer
    ↓
Engineer manually investigates logs
```

**Observability approach:**
```
Distributed trace shows:
Frontend (50ms) → API Gateway (100ms) → User Service (3000ms) ← SLOW
                                              ↓
                                         Database query took 2.9s
                                              ↓
                                      Missing index on users.email
```

---

**Q2. What is Prometheus and how does it work?**

**Answer:**  
Prometheus is an open-source monitoring system that collects metrics from applications and infrastructure, stores them in a time-series database, and enables querying and alerting.

**Architecture:**

```
   Application exposes /metrics endpoint
              ↓
   Prometheus scrapes metrics every 15s
              ↓
   Time-series database stores data
              ↓
   PromQL queries analyze data
              ↓
   Alert Manager fires alerts
              ↓
   Grafana visualizes dashboards
```

**Example metric endpoint (`/metrics`):**
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 12543
http_requests_total{method="GET",status="500"} 23
http_requests_total{method="POST",status="201"} 8901

# HELP memory_usage_bytes Current memory usage
# TYPE memory_usage_bytes gauge
memory_usage_bytes 524288000
```

**Prometheus configuration:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'webapp'
    static_configs:
      - targets: ['webapp:8080']
  
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

**Query examples (PromQL):**
```promql
# Average request rate over last 5 minutes
rate(http_requests_total[5m])

# 95th percentile response time
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Pods using > 80% memory
(container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100 > 80
```

---

### 🟡 Intermediate

**Q3. How would you implement health checks for a microservices application?**

**Answer:**

**Three types of health checks:**

**1. Liveness Probe (is the app running?):**
- If fails → Kubernetes restarts the container
- Should be a fast check (<100ms)

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 1
  failureThreshold: 3  # Restart after 3 consecutive failures
```

**Implementation:**
```javascript
// /health/live endpoint
app.get('/health/live', (req, res) => {
  // Simple check: can the process respond?
  res.status(200).send('OK');
});
```

**2. Readiness Probe (is the app ready to serve traffic?):**
- If fails → Kubernetes removes pod from load balancer (doesn't restart)
- Check dependencies (DB, cache, external APIs)

```yaml
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 2
```

**Implementation:**
```javascript
app.get('/health/ready', async (req, res) => {
  try {
    // Check database
    await db.ping();
    
    // Check Redis cache
    await redis.ping();
    
    // Check external API (with timeout)
    await axios.get('https://api.payment.com/health', { timeout: 500 });
    
    res.status(200).json({ status: 'ready' });
  } catch (error) {
    res.status(503).json({ status: 'not ready', error: error.message });
  }
});
```

**3. Startup Probe (for slow-starting apps):**
- Gives extra time for app initialization
- Prevents liveness probe from killing container during startup

```yaml
startupProbe:
  httpGet:
    path: /health/startup
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
  failureThreshold: 30  # 30 * 10s = 5 minutes to start
```

**Best practices:**

| Do | Don't |
|----|-------|
| ✅ Fast checks (<100ms for liveness) | ❌ Heavy computation in health checks |
| ✅ Check critical dependencies in readiness | ❌ Check non-critical dependencies in liveness |
| ✅ Different endpoints for liveness/readiness | ❌ Same endpoint for both |
| ✅ Return 200 for healthy, 503 for unhealthy | ❌ Return 200 with JSON {"healthy": false} |

---

**Q4. Explain SLIs, SLOs, and SLAs. How would you define them for a web service?**

**Answer:**

| Term | Definition | Example |
|------|------------|---------|
| **SLI** (Service Level Indicator) | A metric that measures service performance | Request success rate |
| **SLO** (Service Level Objective) | Target value for an SLI | 99.9% of requests succeed |
| **SLA** (Service Level Agreement) | Contract with customers (with penalties) | 99.5% uptime or refund |

**Hierarchy:**
```
SLA (customer-facing contract)
  ↓
SLO (internal target, stricter than SLA)
  ↓
SLI (measured metric)
```

**Example for an e-commerce website:**

**SLI #1 — Availability:**
```
SLI = (successful requests) / (total requests)
SLO = 99.95% over rolling 30 days
SLA = 99.5% uptime (customer gets refund if below)

Error budget = 100% - 99.95% = 0.05%
            = 21 minutes downtime allowed per month
```

**SLI #2 — Latency:**
```
SLI = p95 response time (95th percentile)
SLO = p95 < 300ms
No SLA (latency usually not in customer contracts)
```

**SLI #3 — Data freshness:**
```
SLI = Time since last successful data sync
SLO = Data must be < 5 minutes stale
```

**Implementation in Prometheus:**

```promql
# Availability SLI
sum(rate(http_requests_total{status=~"2.."}[5m]))
/
sum(rate(http_requests_total[5m]))

# Alert when burning error budget too fast
(1 - availability_sli) > (0.05 / 30)  # Burning > 1 day of error budget per day
```

**Error Budget Policy:**

```
If error budget exhausted:
- Freeze feature releases (only bug fixes)
- All hands on reliability improvements
- Postmortem required

If error budget healthy:
- Ship features faster
- Take calculated risks
```

---

### 🔴 Hardcore

**Q5. Design a complete observability stack for a microservices application with 15 services.**

**Answer:**

**Full Stack Architecture:**

```
                    ┌──────────────────────┐
                    │   Grafana (UI)       │
                    │   - Dashboards       │
                    │   - Alerting         │
                    └──────────────────────┘
                             ↑
          ┌──────────────────┼──────────────────┐
          ↓                  ↓                  ↓
┌──────────────────┐  ┌──────────────┐  ┌──────────────┐
│   Prometheus     │  │  Loki        │  │  Jaeger      │
│   (Metrics)      │  │  (Logs)      │  │  (Traces)    │
│   - Time series  │  │  - Log       │  │  - Distrib.  │
│   - PromQL       │  │    aggreg.   │  │    tracing   │
└──────────────────┘  └──────────────┘  └──────────────┘
          ↑                  ↑                  ↑
          └──────────────────┴──────────────────┘
                             │
        ┌────────────────────┴────────────────────┐
        │         Kubernetes Cluster              │
        │  ┌─────────┬─────────┬─────────┐        │
        │  │Service A│Service B│Service C│ ... 15 │
        │  └─────────┴─────────┴─────────┘        │
        └─────────────────────────────────────────┘
```

**1. Metrics (Prometheus):**

**Service instrumentation:**
```javascript
// app.js (Node.js example)
const promClient = require('prom-client');
const express = require('express');

// Enable default metrics (CPU, memory, etc.)
promClient.collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// Middleware to record metrics
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || req.path, res.statusCode).observe(duration);
    httpRequestTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
  });
  next();
});

// Expose /metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

**Prometheus scraping config:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
```

**2. Logs (Loki):**

**Logging pattern:**
```javascript
const winston = require('winston');
const LokiTransport = require('winston-loki');

const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [
    new LokiTransport({
      host: 'http://loki:3100',
      labels: {
        service: 'user-service',
        environment: 'production'
      },
      json: true
    })
  ]
});

// Structured logging
logger.info('User login successful', {
  userId: 12345,
  ip: '192.168.1.1',
  duration: 234
});

logger.error('Database connection failed', {
  error: err.message,
  stack: err.stack,
  retryCount: 3
});
```

**Loki query (LogQL):**
```logql
# All errors in last hour from user-service
{service="user-service"} |= "error" | json | level="error"

# Login failures by user
{service="auth-service"} |= "login failed" | json | count_over_time(1h) by (userId)

# Slow queries (> 1 second)
{service="api-gateway"} | json | duration > 1000 | line_format "{{.method}} {{.path}} took {{.duration}}ms"
```

**3. Distributed Tracing (Jaeger):**

**Instrumentation (OpenTelemetry):**
```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');

const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new SimpleSpanProcessor(
    new JaegerExporter({
      serviceName: 'user-service',
      endpoint: 'http://jaeger-collector:14268/api/traces'
    })
  )
);

registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation()
  ]
});

provider.register();

// Traces are automatically created for HTTP requests
// Manual span for custom operation:
const tracer = provider.getTracer('user-service');
const span = tracer.startSpan('processPayment');
// ... do work
span.end();
```

**Trace view shows:**
```
Request ID: abc-123-def
Total duration: 2.4s

┌─ API Gateway ────────────────────────── 2.4s ─┐
│  ├─ Auth Check ──── 50ms                       │
│  ├─ User Service ────────────── 1.8s ─┐        │
│  │  ├─ DB Query ───── 1.5s            │        │
│  │  └─ Cache Update ── 100ms          │        │
│  └─ Billing Service ─── 500ms                  │
└────────────────────────────────────────────────┘
```

**4. Alerting (Alert Manager):**

```yaml
# alertmanager.yml
route:
  group_by: ['alertname', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'pagerduty-critical'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<pagerduty-integration-key>'
  
  - name: 'slack-warnings'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx'
        channel: '#devops-alerts'
```

**5. Dashboards (Grafana):**

**Golden Signals Dashboard:**
```json
{
  "panels": [
    {
      "title": "Request Rate (per service)",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total[5m])) by (service)"
        }
      ]
    },
    {
      "title": "Error Rate",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))"
        }
      ],
      "alert": {
        "conditions": [
          {
            "evaluator": { "type": "gt", "params": [0.01] }  // > 1% error rate
          }
        ],
        "frequency": "1m"
      }
    },
    {
      "title": "p95 Latency",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))"
        }
      ]
    },
    {
      "title": "Saturation (CPU)",
      "targets": [
        {
          "expr": "avg(rate(container_cpu_usage_seconds_total[5m])) by (pod)"
        }
      ]
    }
  ]
}
```

**Cost & Retention:**
- Prometheus: 30-day retention, 1TB storage
- Loki: 7-day retention, 500GB storage
- Jaeger: 3-day trace retention, sample 10% of traces

---

## 7. Agile & Collaboration

### 🟢 Basic

**Q1. What is Agile and how does DevOps fit into it?**

**Answer:**  
Agile is a project management methodology focused on iterative development, collaboration, and rapid feedback. Development happens in short sprints (1-2 weeks) rather than long waterfall phases.

**Agile ceremonies:**
- **Sprint Planning:** Define what to build this sprint
- **Daily Standups:** 15-min sync on progress/blockers
- **Sprint Review/Demo:** Show completed work to stakeholders
- **Retrospective:** What went well, what to improve

**DevOps + Agile:**

| Agile (Dev) | DevOps (Dev + Ops) |
|-------------|-------------------|
| Ship features fast | Ship features fast **and** reliably |
| 2-week sprints | Continuous deployment (multiple times per day) |
| Manual testing at end of sprint | Automated testing in CI pipeline |
| Ops team deploys quarterly | Dev team deploys themselves via CI/CD |

**Example workflow:**
```
Sprint 1 (2 weeks):
Day 1: Sprint planning
Day 2-9: Development (commit → CI runs tests → deploy to dev automatically)
Day 10: Feature complete → deploy to staging (automated)
Day 11: QA testing
Day 12: Deploy to production (via CI/CD, one-click approval)
Day 13: Monitoring, fix bugs
Day 14: Sprint review, retrospective
```

**DevOps extends Agile by:**
- Automating manual steps (testing, deployment)
- Including Ops in the development process
- Enabling continuous feedback from production

---

**Q2. What does "you build it, you run it" mean in DevOps?**

**Answer:**  
"You build it, you run it" means the team that develops a feature is also responsible for operating it in production — including monitoring, on-call, and fixing issues.

**Traditional model:**
```
Dev team: Writes code, hands off to Ops
    ↓
Ops team: Deploys, maintains, gets paged at 3 AM
    ↓
Dev team: "Works on my machine, not my problem"
```

**DevOps model:**
```
Dev team:
- Writes code
- Writes tests
- Deploys via CI/CD
- Monitors production dashboards
- Gets paged when app breaks
- Fixes issues immediately
```

**Benefits:**
- **Faster feedback:** Developers see production issues immediately
- **Better quality:** Knowing you'll be paged at 3 AM motivates writing better code
- **Ownership:** Pride in running reliable systems
- **Reduced blame:** No more "Dev vs. Ops" finger-pointing

**Example at Accenture:**
- Team owns a microservice (user-service)
- On-call rotation: 1 week on-call per person every 6 weeks
- PagerDuty alerts go to the team, not a separate Ops team
- Team reviews post-mortems and implements fixes

---

### 🟡 Intermediate

**Q3. You're on-call and get paged at 2 AM for a production incident. Walk through your incident response process.**

**Answer:**

**Phase 1 — Acknowledge & Assess (0-5 minutes):**

1. **Acknowledge the alert** (PagerDuty/Opsgenie)
   - Stops re-paging others
   - Shows you're on it

2. **Triage severity:**
   - **P1 (Critical):** Service completely down, data loss, security breach → All hands on deck
   - **P2 (High):** Partial outage, degraded performance → Can wait until morning if contained
   - **P3 (Medium):** Non-critical feature broken → Fix next business day

3. **Check dashboards:**
   - Grafana: What's the error rate? Which service?
   - Jaeger: Are traces showing a specific slow component?
   - Logs: Any recent error spikes?

**Phase 2 — Communicate (5-10 minutes):**

4. **Post in incident channel:**
   ```
   #incident-2024-02-17
   🚨 INCIDENT DECLARED 🚨
   Severity: P1
   Issue: API Gateway returning 500 errors
   Impact: 80% of users unable to login
   Investigating: @alex
   Status updates every 15 minutes
   ```

5. **Notify stakeholders** (if customer-facing):
   - Update status page: "We're aware of login issues, investigating"
   - Set up bridge call if needed (P1 only)

**Phase 3 — Mitigate (10-30 minutes):**

6. **Quick fix (stop the bleeding):**
   - Roll back recent deployment: `kubectl rollout undo deployment/api-gateway`
   - Scale up resources: `kubectl scale deployment/api-gateway --replicas=10`
   - Disable problematic feature flag
   - Redirect traffic to backup region

7. **Verify fix:**
   - Check error rate drops to baseline
   - Spot-check user logins working
   - Monitor for 10 minutes to ensure stable

**Phase 4 — Resolve (30-60 minutes):**

8. **Root cause identified:**
   - Database connection pool exhausted (max 10 connections, needed 50)

9. **Permanent fix:**
   - Increase connection pool size in config
   - Deploy fix via CI/CD
   - Monitor for 30 minutes

10. **Close incident:**
   ```
   ✅ RESOLVED
   Root cause: DB connection pool too small
   Fix: Increased from 10 to 50 connections
   Duration: 45 minutes
   Postmortem: Due Monday 9 AM
   ```

**Phase 5 — Postmortem (next business day):**

11. **Write postmortem** (blameless):
   - Timeline of events
   - Root cause (why did it happen?)
   - Why detection was slow
   - Action items to prevent recurrence

**Example action items:**
- [ ] Add alert for DB connection pool saturation
- [ ] Load test before production deploy (catch this in staging)
- [ ] Auto-scale connection pool based on load

---

### 🔴 Hardcore

**Q4. You're leading a DevOps transformation at a client with 50 developers who have never used CI/CD. How do you approach this?**

**Answer:**

**Phase 1 — Assessment (Weeks 1-2):**

**1. Current state analysis:**
```
Deployment process today:
- Developer emails ZIP file to Ops team
- Ops manually SSHs into 10 servers
- Copy files, restart services
- Takes 4 hours, happens once per month
- 30% of deployments fail
```

**2. Pain point interviews:**
- Shadow a deployment start-to-finish
- Interview devs: "What frustrates you most?"
- Interview ops: "What breaks most often?"

**3. Quick wins identification:**
- Automated tests (currently: manual QA team tests for 3 days)
- Docker packaging (currently: "works on my machine" issues)
- Staging environment (currently: test directly in prod 😱)

**Phase 2 — Pilot Project (Weeks 3-6):**

**4. Select 1 team (5 devs) for pilot:**
- Criteria: enthusiastic, not mission-critical app
- Weekly check-ins with team

**5. Build MVP pipeline:**
```
GitHub → Jenkins → Docker build → Deploy to dev → Automated tests
```

**6. Measure metrics:**
| Metric | Before | After Pilot |
|--------|--------|-------------|
| Deployment frequency | 1/month | 5/week |
| Lead time (commit → prod) | 4 hours | 15 minutes |
| Deployment failure rate | 30% | 5% |
| Mean time to recovery | 2 hours | 10 minutes |

**Phase 3 — Rollout (Weeks 7-16):**

**7. Training program:**
- Week 1: Git & branching strategies (hands-on workshop)
- Week 2: Docker basics (each dev Dockerizes one app)
- Week 3: Jenkins pipeline as code (build their own Jenkinsfile)
- Week 4: Kubernetes fundamentals

**8. Create "golden path" templates:**
```
accenture-devops-templates/
├── Jenkinsfile-node.js
├── Jenkinsfile-java-maven
├── Dockerfile-node
├── Dockerfile-java
└── k8s-deployment-template.yaml
```

**9. Onboard teams incrementally:**
- 5 teams every 2 weeks
- Pair programming: experienced DevOps engineer + 2 devs from new team
- Each team migrates 1 app fully before moving to next app

**Phase 4 — Governance & Scaling (Weeks 17-24):**

**10. Establish standards:**
- All apps must have:
  - [ ] Dockerfile
  - [ ] Jenkinsfile (CI/CD pipeline)
  - [ ] Health check endpoint (/health)
  - [ ] Structured logging (JSON format)
  - [ ] Prometheus metrics endpoint (/metrics)

**11. Inner-source community:**
- Monthly DevOps guild meeting (show & tell)
- Slack channel: #devops-help
- "DevOps Champions" in each team (1 person trained deeply)

**12. Measure success:**
```
DORA metrics (6 months after transformation):

Deployment Frequency: 1/month → 10/day ✅
Lead Time: 4 hours → 1 hour ✅
Change Failure Rate: 30% → 8% ✅
MTTR: 2 hours → 20 minutes ✅
```

**Common Challenges & Solutions:**

| Challenge | Solution |
|-----------|----------|
| **"But we've always done it this way"** | Show metrics: faster, more reliable, happier developers |
| **"We don't have time to learn new tools"** | Invest 2 weeks training, save 50 hours/month in manual work |
| **"Our app is too complex for automation"** | Start with smallest piece (e.g., just build automation) |
| **Fear of breaking production** | Start with non-critical apps, build confidence |

---

## 8. Scenario-Based Problem Solving

### 🎯 Scenario 1: Application Slowness After Deployment

**Problem:**  
You deployed a new version of your microservice to production at 2 PM. At 2:30 PM, response times spike from 200ms to 3 seconds. Customers are complaining. What do you do?

**Answer:**

**Step 1 — Immediate Actions (0-5 min):**
1. Check if issue correlates with deployment time (yes → likely caused by new code)
2. Check Grafana dashboard: Which endpoint is slow? All endpoints or specific one?
3. Quick decision: **Roll back** immediately if customer impact is severe
   ```bash
   kubectl rollout undo deployment/user-service
   ```

**Step 2 — Root Cause Investigation (5-30 min):**

**Check distributed tracing (Jaeger):**
```
Request /api/users/:id
Total: 3.2s
  ├─ API Gateway: 50ms
  ├─ User Service: 3.1s ← SLOW
       ├─ Get user from DB: 2.9s ← PROBLEM HERE
       └─ Build response: 100ms
```

**Identified issue: Database query slow**

**Compare new vs. old code:**
```diff
// Old code
- SELECT * FROM users WHERE id = ?

// New code (deployed 30 min ago)
+ SELECT * FROM users WHERE id = ? AND status != 'deleted' AND last_login < NOW()
```

**Problem:** New query doesn't use index!

**Step 3 — Permanent Fix:**
1. Add database index:
   ```sql
   CREATE INDEX idx_users_status_last_login ON users(status, last_login);
   ```
2. Re-deploy the new version
3. Monitor for 15 minutes → latency back to 200ms ✅

**Step 4 — Postmortem (next day):**
- **Root cause:** New query added without checking explain plan
- **Detection:** Took 30 minutes (should have been 5 minutes)
- **Action items:**
  - [ ] Add slow query detection alert (Prometheus: p95 > 1 second → alert)
  - [ ] Require database explain plan review in PRs
  - [ ] Add load testing in staging before production deploy

---

### 🎯 Scenario 2: Kubernetes Cluster Instability

**Problem:**  
Your Kubernetes cluster starts showing random pod restarts. Some pods are stuck in "CrashLoopBackOff". Nodes are intermittently going "NotReady". What do you investigate?

**Answer:**

**Step 1 — Assess Impact:**
```bash
# Check pod status
kubectl get pods --all-namespaces | grep -v Running

# Check node health
kubectl get nodes
NAME         STATUS     ROLES    AGE   VERSION
node-1       Ready      worker   5d    v1.27.0
node-2       NotReady   worker   5d    v1.27.0  ← PROBLEM
node-3       Ready      worker   5d    v1.27.0

# Check events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'
```

**Step 2 — Investigate Node Issues:**

**SSH into problem node:**
```bash
# Check disk space (common cause)
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   49G     0 100% /var/lib/kubelet  ← FULL DISK

# Check Docker disk usage
docker system df
Images          15GB
Containers      20GB
Volumes         14GB
```

**Root Cause:** Disk full due to old Docker images and logs

**Step 3 — Immediate Fix:**
```bash
# Clean up old Docker images
docker image prune -a --force

# Clean up old logs
journalctl --vacuum-time=2d

# Restart kubelet
systemctl restart kubelet

# Node comes back online
kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
node-2       Ready    worker   5d    v1.27.0  ✅
```

**Step 4 — Prevent Recurrence:**

**1. Set up log rotation:**
```yaml
# /etc/logrotate.d/docker-container
/var/lib/docker/containers/*/*.log {
  rotate 7
  daily
  compress
  size=10M
  missingok
  delaycompress
  copytruncate
}
```

**2. Enable Kubernetes garbage collection:**
```yaml
# kubelet config
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
```

**3. Add monitoring alert:**
```yaml
# Prometheus alert
- alert: NodeDiskPressure
  expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 15
  for: 10m
  labels:
    severity: warning
  annotations:
    summary: "Node {{ $labels.instance }} disk usage > 85%"
```

---

### 🎯 Scenario 3: CI/CD Pipeline Suddenly Takes 2 Hours

**Problem:**  
Your CI pipeline used to take 15 minutes. Starting yesterday, it takes 2 hours. Nothing obvious changed. What do you investigate?

**Answer:**

**Step 1 — Profile the Pipeline:**

**Check Jenkins build log:**
```
[08:00:00] Checkout code: 1 min ✅
[08:01:00] Install dependencies: 58 min ❌ (used to be 2 min)
[08:59:00] Run tests: 15 min ✅
[09:14:00] Build Docker image: 5 min ✅
[09:19:00] Security scan: 3 min ✅
Total: 2 hours
```

**Bottleneck identified:** `npm install` taking 58 minutes

**Step 2 — Investigate Dependency Installation:**

**Check npm registry:**
```bash
# Is npm registry slow?
time npm ping
Ping registry: 5000ms  ← SLOW (normally < 100ms)
```

**Root Cause:** npm registry is down/slow, falling back to slow mirrors

**Step 3 — Immediate Workaround:**

**Use cached dependencies:**
```groovy
// Jenkinsfile
stage('Install Dependencies') {
    steps {
        // Use local npm cache
        sh 'npm ci --cache /var/cache/npm'
        
        // Or use Nexus/Artifactory as proxy
        sh 'npm config set registry https://nexus.accenture.com/repository/npm-proxy/'
        sh 'npm ci'
    }
}
```

**Step 4 — Long-term Solutions:**

**1. Set up internal npm proxy (Nexus/Artifactory):**
- Caches packages locally
- Continues working even if public npm is down
- Speeds up builds (local cache vs. internet)

**2. Use Docker layer caching:**
```dockerfile
FROM node:18

WORKDIR /app

# Copy only package files first
COPY package*.json ./
RUN npm ci --only=production  # This layer is cached

# Then copy app code
COPY . .
```

**3. Parallel stages:**
```groovy
stage('Parallel Tasks') {
    parallel {
        stage('Install Dependencies') { ... }
        stage('Lint') { ... }
        stage('Security Scan') { ... }
    }
}
```

---

## 9. Interview Tips & Key Takeaways

### 📝 How Accenture Interviews Work

**Interview Rounds (typical):**

1. **Technical Screening (1 hour):**
   - Basic DevOps concepts (CI/CD, containers, cloud)
   - Coding/scripting (write a Bash script to automate X)
   - Scenario: "You have a slow CI pipeline, what do you check?"

2. **Technical Deep Dive (1-1.5 hours):**
   - Architecture design: "Design a CI/CD pipeline for 15 microservices"
   - Hands-on: "Here's a Kubernetes cluster with issues, debug it"
   - Past experience: "Tell me about a time you optimized X"

3. **Cultural/Behavioral (30-45 min):**
   - Accenture values: collaboration, innovation, client focus
   - Agile experience: "How do you handle urgent production issues during sprint?"
   - Leadership: "Tell me about a time you led a DevOps transformation"

4. **Manager Round (30 min):**
   - Big-picture thinking
   - Career goals, learning mindset
   - Accenture projects/clients of interest

---

### 🎯 How to Answer Accenture Questions

**Use STAR Format:**

| Component | What to say | Example |
|-----------|-------------|---------|
| **S**ituation | Brief context (1-2 sentences) | "Our CI pipeline took 45 minutes, blocking 50 developers" |
| **T**ask | Your specific responsibility | "I was tasked with reducing it to under 10 minutes" |
| **A**ction | What YOU did (most detail here) | "I profiled the pipeline, parallelized tests, cached dependencies, used Docker layer caching" |
| **R**esult | Measurable outcome | "Reduced to 8 minutes, 10× daily deployments, team velocity +30%" |

**Example Question:**  
*"Tell me about a time you improved system reliability."*

**STAR Answer:**
> **S:** At my previous company, our production API had 3-5 outages per month due to memory leaks.  
> **T:** I was responsible for improving reliability to < 1 outage per quarter.  
> **A:** I implemented: (1) memory profiling with Node.js heap snapshots, (2) added memory limit alerts in Prometheus, (3) automated daily restarts during low-traffic windows, (4) refactored problematic code that wasn't releasing DB connections.  
> **R:** Outages dropped from 5/month to 0 in the following quarter. MTTR decreased from 2 hours to 15 minutes. Implemented learnings across 3 other services.

---

### 💡 Key Technical Areas to Revise

| Area | Must Know | Nice to Have |
|------|-----------|--------------|
| **CI/CD** | Jenkins, GitLab CI, GitHub Actions, webhooks, quality gates | CircleCI, Azure DevOps |
| **Containers** | Docker basics, Dockerfile, Kubernetes (pods, deployments, services) | Helm, Istio, Docker Compose |
| **Cloud** | AWS EC2, S3, VPC, IAM, basic networking | Azure, GCP equivalents |
| **IaC** | Terraform syntax, state management, modules | CloudFormation, Pulumi |
| **Scripting** | Bash, Python (automate tasks, parse JSON/YAML) | Go, PowerShell |
| **Monitoring** | Prometheus, Grafana, basic PromQL | ELK, Jaeger, New Relic |
| **Version Control** | Git (branching, merging, rebasing, conflict resolution) | GitHub API, GitLab CI |

---

### 🚀 Common Accenture DevOps Questions

**They will likely ask:**
1. "Explain your CI/CD pipeline" → Be ready to draw a diagram
2. "How do you handle secrets?" → Never say "environment variables in Git"
3. "Docker vs. VM?" → Emphasize lightweight, fast startup, consistency
4. "Kubernetes architecture" → Master node, worker nodes, etcd, kubelet
5. "Blue-green vs. canary deployment" → Know when to use which
6. "You're on-call at 2 AM, production is down..." → Have a process
7. "Tell me about your Agile experience" → Sprint ceremonies, story points, retrospectives

---

### ✅ Day-Before-Interview Checklist

- [ ] Review this guide (30 min quick scan)
- [ ] Practice 3 STAR stories from your experience
- [ ] Draw out your best CI/CD pipeline design (on paper, timed — 10 min)
- [ ] Be ready to write code: "Write a Bash script to check if a service is healthy"
- [ ] Prepare questions for interviewer:
  - "What DevOps tools does Accenture's client projects typically use?"
  - "What's the team structure — is there a central DevOps team or embedded in product teams?"
  - "What's the biggest DevOps challenge you're facing right now?"

---

### 🎓 Final Advice

**Do:**
- ✅ Think out loud during technical questions (show your thought process)
- ✅ Ask clarifying questions ("When you say 'design a pipeline', should I assume multi-region?")
- ✅ Admit when you don't know something, then explain how you'd find the answer
- ✅ Show enthusiasm for learning and automation

**Don't:**
- ❌ Say "I don't know" and stop (add: "but here's how I'd approach it...")
- ❌ Trash-talk previous employers or teams
- ❌ Claim expertise in tools you've only read about (they will drill down)
- ❌ Skip the "Result" in STAR (quantify impact with numbers)

---

**Good luck with your Accenture interview! 🚀**

*Focus on fundamentals, show a learning mindset, and emphasize collaboration — Accenture values these highly.*