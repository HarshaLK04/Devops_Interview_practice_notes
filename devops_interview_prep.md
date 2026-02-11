# Senior DevOps Engineer Interview Preparation
## Based on Resume: Harshal Kalamkar & Job Description

---

## SECTION 1: BASIC LEVEL QUESTIONS

### 1. CI/CD Pipelines

**Q: Explain the CI/CD pipeline you designed at Birlasoft. What stages did it include?**

**Answer:**
"At Birlasoft, I designed CI/CD pipelines for Kubernetes-based microservices that reduced manual deployment effort by 30%. The pipeline included:

1. **Source Control Integration** - Triggered on Git commits/PRs from GitHub
2. **Build Stage** - Maven builds with unit tests
3. **Code Quality & Security** - SonarQube scans and Docker image security scanning
4. **Artifact Management** - Storing artifacts in JFrog Artifactory
5. **Containerization** - Building Docker images with vulnerability scanning
6. **Deployment Stages** - Progressive deployment to DEV → QA → STAGING → PROD
7. **Testing** - Automated integration and smoke tests
8. **Monitoring Integration** - Dynatrace and Prometheus integration for observability

The pipeline was implemented using Infrastructure-as-Code principles with declarative Jenkins pipelines, ensuring repeatability and version control of the entire delivery process."

---

**Q: What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?**

**Answer:**
- **Continuous Integration (CI)**: Developers frequently merge code changes into a central repository, followed by automated builds and tests. This ensures early bug detection.

- **Continuous Delivery (CD)**: Extends CI by automatically deploying all code changes to a testing/staging environment after the build stage. Deployment to production requires manual approval.

- **Continuous Deployment**: Takes Continuous Delivery further by automatically deploying every change that passes all pipeline stages directly to production without manual intervention.

In my experience at Northern Trust, we practiced Continuous Delivery due to compliance requirements, where production deployments needed manual approval gates."

---

### 2. Terraform & Infrastructure-as-Code

**Q: How did you use Terraform in your projects? Walk me through your workflow.**

**Answer:**
"At both Infosys and Birlasoft, I used Terraform extensively:

**Workflow:**
1. **State Management** - Used remote state backends (S3 with DynamoDB for locking)
2. **Module Structure** - Created reusable modules for VPC, EC2, RDS, IAM resources
3. **Environment Separation** - Separate workspaces/state files for dev/qa/prod
4. **Version Control** - All Terraform code in Git with PR-based reviews
5. **Planning & Validation** - `terraform plan` in CI pipeline before apply
6. **Automated Application** - Applied through Jenkins/AWX with approval gates

**Example at Infosys:**
- Provisioned AWS infrastructure including VPCs with public/private subnets
- Created EC2 instances with proper IAM roles
- Set up S3 buckets with encryption and lifecycle policies
- Managed security groups and network ACLs

The IaC approach ensured consistency, repeatability, and made infrastructure auditable."

---

**Q: What are Terraform state files and why are they important?**

**Answer:**
"Terraform state files track the current state of managed infrastructure. They're crucial because:

1. **Mapping** - Maps Terraform configuration to real-world resources
2. **Metadata Storage** - Stores resource dependencies and metadata
3. **Performance** - Caches attribute values for better performance
4. **Collaboration** - With remote state, teams can work together

**Best Practices I followed:**
- Always used remote state backends (S3 + DynamoDB for AWS)
- Enabled state locking to prevent concurrent modifications
- Encrypted state files as they can contain sensitive data
- Regular state backups
- Never manually edited state files
- Used `terraform state` commands for state management operations

At Infosys, we had incidents where local state caused infrastructure drift, which reinforced why remote state with locking is critical."

---

### 3. Ansible & Configuration Management

**Q: Explain your experience with AWX. How did you deploy and manage it?**

**Answer:**
"At Birlasoft, I deployed AWX on Kubernetes as an enterprise automation platform:

**Deployment:**
- Used AWX Operator for Kubernetes deployment
- Configured PostgreSQL as the backend database
- Set up persistent volumes for data persistence
- Implemented RBAC for multi-tenant access control

**Management:**
- **Inventory Management**: Created dynamic inventories syncing with VMware and cloud providers
- **Credential Management**: Secured credentials using AWX vault for SSH keys, API tokens
- **RBAC**: Configured organizations, teams, and role-based permissions
- **Job Templates**: Created reusable job templates for common automation tasks
- **Workflow Templates**: Built complex workflows chaining multiple playbooks
- **Notifications**: Integrated with Slack/email for job status notifications

This centralized our automation, provided audit trails, and enabled self-service automation for development teams."

---

**Q: What's the difference between Ansible and Terraform?**

**Answer:**
"Both are automation tools but serve different purposes:

**Terraform:**
- **Purpose**: Infrastructure provisioning (IaC)
- **Approach**: Declarative - you define desired state
- **State Management**: Maintains state files
- **Best For**: Creating/destroying infrastructure resources
- **Example**: Provisioning AWS VPC, EC2 instances, RDS databases

**Ansible:**
- **Purpose**: Configuration management and orchestration
- **Approach**: Procedural (can be declarative with modules)
- **State**: Agentless, no state tracking (checks current state each run)
- **Best For**: Configuring servers, deploying applications, orchestration
- **Example**: Installing packages, configuring services, deploying apps

**In Practice:**
I used them together - Terraform to provision infrastructure, then Ansible to configure it. For example, at Infosys:
1. Terraform created EC2 instances
2. Ansible installed and configured applications on those instances

This separation of concerns is a DevOps best practice."

---

### 4. Docker & Kubernetes

**Q: Describe a production issue you troubleshot in Kubernetes. How did you resolve it?**

**Answer:**
"At Birlasoft, we had a critical production issue where pods were failing with PersistentVolumeClaim (PVC) binding errors:

**Issue:**
- Pods stuck in 'Pending' state
- Events showed: 'PersistentVolumeClaim is not bound'
- Impact: Microservice unavailable, affecting customer transactions

**Troubleshooting Steps:**
1. **Checked pod status**: `kubectl describe pod <pod-name>`
2. **Examined PVC**: `kubectl get pvc` - showed 'Pending' status
3. **Checked StorageClass**: Found no dynamic provisioner configured
4. **Reviewed PV availability**: No PersistentVolumes matched PVC requirements

**Root Cause:**
StorageClass was missing the provisioner configuration after cluster upgrade.

**Resolution:**
1. Created proper StorageClass with correct provisioner
2. Updated PVC to use correct StorageClass
3. Verified PV was dynamically created
4. Pods came up successfully

**Prevention:**
- Added StorageClass to Infrastructure-as-Code
- Created monitoring alerts for pending PVCs
- Documented runbook for PVC troubleshooting
- This reduced MTTR for similar issues from hours to minutes."

---

**Q: How did you harden Docker images? What security measures did you implement?**

**Answer:**
"At Birlasoft, I implemented several Docker security hardening practices:

**1. Base Image Selection:**
- Used minimal base images (Alpine, distroless)
- Scanned base images for vulnerabilities before use
- Used specific version tags, never 'latest'

**2. Image Scanning:**
- Integrated SonarQube and registry security scans in CI/CD
- Failed builds if critical vulnerabilities found
- Regular rescanning of images in registry

**3. Dockerfile Best Practices:**
- Ran containers as non-root user
- Removed unnecessary packages and files
- Multi-stage builds to reduce image size
- No hardcoded secrets - used external secret management

**4. Runtime Security:**
- Implemented read-only root filesystems where possible
- Restricted container capabilities
- Used securityContext in Kubernetes deployments

**Example Dockerfile snippet:**
```dockerfile
FROM alpine:3.18
RUN addgroup -g 1000 appgroup && adduser -D -u 1000 -G appgroup appuser
COPY --chown=appuser:appgroup app /app
USER appuser
```

This reduced our vulnerability count by 60% and strengthened our security posture."

---

### 5. Jenkins Administration

**Q: What's your experience with Jenkins? How have you optimized Jenkins pipelines?**

**Answer:**
"I have extensive Jenkins administration experience from Infosys:

**Administration:**
- Installed and configured Jenkins on Linux servers
- Managed plugins (kept them updated, removed unused ones)
- Configured security (LDAP integration, role-based access)
- Set up distributed builds using Jenkins agents
- Backup and disaster recovery planning

**Pipeline Optimization:**
1. **Parallel Execution**: Ran independent stages in parallel (testing, scanning)
2. **Caching**: Cached dependencies to avoid repeated downloads
3. **Incremental Builds**: Used Maven incremental builds
4. **Resource Management**: Configured executors and resource limits properly
5. **Pipeline Libraries**: Created shared libraries for reusable code
6. **Declarative Pipelines**: Migrated from scripted to declarative for better maintainability

**Example Optimization:**
Original pipeline: 25 minutes
After optimization:
- Parallel test execution: saved 8 minutes
- Dependency caching: saved 5 minutes
- Optimized Docker builds: saved 4 minutes
**Final time: 8 minutes** - 68% improvement

This significantly improved developer productivity and feedback loops."

---

### 6. Cloud Platforms (AWS)

**Q: Explain how you designed VPC architecture in AWS.**

**Answer:**
"At Infosys, I designed VPC architectures following AWS best practices:

**Architecture Components:**

1. **VPC CIDR Planning**: 
   - Used /16 CIDR blocks (e.g., 10.0.0.0/16)
   - Left room for expansion

2. **Subnet Strategy**:
   - Public subnets (/24) for internet-facing resources (NAT, Load Balancers)
   - Private subnets (/24) for application servers
   - Database subnets (/24) in isolated tier
   - Spread across multiple AZs for high availability

3. **Routing**:
   - Public route tables pointing to Internet Gateway
   - Private route tables pointing to NAT Gateway
   - Separate route tables per tier

4. **Security**:
   - Security Groups: Stateful, instance-level firewalls
   - NACLs: Stateless, subnet-level (deny rules for known bad IPs)
   - Principle of least privilege

5. **Connectivity**:
   - VPC Peering for cross-VPC communication
   - VPC Endpoints for AWS services (S3, DynamoDB)

**Terraform Implementation:**
Created modular Terraform code for repeatable VPC deployments across environments, ensuring consistency and compliance."

---

### 7. Monitoring & Observability

**Q: How did you implement observability using Dynatrace and Prometheus?**

**Answer:**
"At Birlasoft, I implemented comprehensive observability:

**Dynatrace Implementation:**
- **APM Monitoring**: Installed OneAgent on hosts for automatic application discovery
- **Distributed Tracing**: Tracked requests across microservices
- **AI-powered Anomaly Detection**: Detected performance degradation automatically
- **Business Metrics**: Created custom dashboards for business KPIs
- **Alerting**: Configured problem notifications to PagerDuty/Slack

**Prometheus + Grafana:**
- **Metrics Collection**: 
  - Deployed Prometheus in Kubernetes using Helm
  - Configured ServiceMonitors for automatic scraping
  - Collected application metrics via /metrics endpoints
- **Kubernetes Monitoring**: Used kube-state-metrics and node-exporter
- **Custom Metrics**: Instrumented applications with Prometheus client libraries
- **Visualization**: Built Grafana dashboards for infrastructure and application metrics
- **Alerting**: Configured AlertManager for threshold-based alerts

**Impact:**
- Reduced MTTR (Mean Time To Repair) by 40%
- Proactive issue detection before customer impact
- Better capacity planning with historical metrics
- SLA compliance improved from 97% to 99.5%"

---

### 8. ITIL & SRE Practices

**Q: How have you applied ITIL and SRE principles in your work?**

**Answer:**
"I've extensively applied both frameworks:

**ITIL Practices:**

1. **Incident Management**:
   - Led incident troubleshooting and RCA at Birlasoft
   - Followed structured incident response process
   - Documented incidents in ServiceNow with proper categorization

2. **Change Management**:
   - All infrastructure changes through CAB approval
   - Maintained change calendar to avoid conflicts
   - Implemented rollback plans for all changes

3. **Release Management**:
   - Coordinated releases with stakeholders
   - Created release runbooks and checklists
   - Post-release validation and monitoring

4. **Configuration Management**:
   - Maintained CMDB with infrastructure components
   - Tracked configuration changes through IaC

**SRE Practices:**

1. **Error Budgets**:
   - Defined SLIs (latency, availability, error rate)
   - Set SLOs (99.9% uptime target)
   - Used error budgets to balance feature velocity vs. reliability

2. **Automation**:
   - Automated toil - reduced manual work by 30%
   - Created runbooks for common incidents
   - Implemented auto-remediation where possible

3. **Blameless Postmortems**:
   - Conducted RCA focusing on processes, not people
   - Implemented preventive actions
   - Shared learnings across teams

4. **Capacity Planning**:
   - Monitored resource utilization
   - Planned infrastructure scaling proactively

The combination helped balance reliability with agility."

---

### 9. Version Control & Collaboration

**Q: Describe your Git workflow and branching strategy.**

**Answer:**
"I've used Git extensively with structured branching strategies:

**Git Workflow (GitFlow variant):**

1. **Branches**:
   - `main/master`: Production-ready code
   - `develop`: Integration branch
   - `feature/*`: New features
   - `bugfix/*`: Bug fixes
   - `hotfix/*`: Emergency production fixes
   - `release/*`: Release preparation

2. **Process**:
   - Developers create feature branches from develop
   - Pull Requests for code review (minimum 2 approvers)
   - Automated checks in PR: tests, linting, security scans
   - Merge to develop after approval
   - Release branches cut from develop
   - After UAT, release merged to main and tagged
   - Hotfixes branched from main, merged to both main and develop

3. **Commit Standards**:
   - Meaningful commit messages
   - Conventional commits format where applicable
   - Commits linked to Jira tickets

4. **Repository Management**:
   - Used GitHub at Birlasoft, Bitbucket at Infosys
   - Protected branches - no direct commits to main/develop
   - Required status checks before merge
   - Branch cleanup after merge

**Collaboration Tools:**
- Jira for task tracking
- Confluence for documentation
- Azure Boards for sprint planning
- Slack for communication

This ensured code quality, traceability, and smooth team collaboration."

---

### 10. Agile & Scrum

**Q: How have you worked in Agile/Scrum environments?**

**Answer:**
"I've worked in Agile environments throughout my career:

**Scrum Practices:**

1. **Sprint Planning**:
   - Participated in 2-week sprints
   - Story estimation using planning poker
   - Committed to sprint goals as a team

2. **Daily Standups**:
   - 15-minute sync on progress, blockers
   - Coordinated with development and QA teams

3. **Sprint Reviews**:
   - Demonstrated completed work to stakeholders
   - Gathered feedback for continuous improvement

4. **Retrospectives**:
   - Reflected on what went well, what didn't
   - Identified action items for improvement
   - Implemented process improvements iteratively

**Kanban Elements:**
- Used Kanban boards in Jira for visualizing work
- WIP limits to prevent overload
- Continuous flow for BAU operational tasks

**DevOps in Agile:**
- Embedded DevOps practices into sprints
- Infrastructure stories in the backlog
- Automated deployments enabled faster iterations
- Maintained Definition of Done including deployment criteria

**Cross-functional Collaboration:**
- Regular sync with developers for pipeline requirements
- Worked with QA on test automation integration
- Coordinated with security teams for compliance

This collaborative approach ensured alignment and faster value delivery."

---

## SECTION 2: HARDCORE/CHALLENGING QUESTIONS

### 1. Advanced CI/CD & Pipeline Optimization

**Q: Your CI/CD pipeline is taking 45 minutes, and developers are complaining about slow feedback. You have limited budget to add more compute resources. How would you optimize this pipeline? Walk me through your analysis and solution.**

**Answer:**
"This is a common challenge. I'd approach it systematically:

**Phase 1: Analysis (Day 1)**

1. **Profile the Pipeline**:
   - Break down time spent in each stage
   - Identify the longest running stages
   - Example breakdown:
     - Checkout: 2 min
     - Build: 15 min
     - Unit Tests: 12 min
     - Integration Tests: 10 min
     - Security Scans: 5 min
     - Docker Build: 8 min
     - Deployment: 3 min

2. **Identify Bottlenecks**: Let's say testing and building are the culprits

**Phase 2: Quick Wins (Week 1)**

1. **Parallel Execution**:
   ```groovy
   stage('Parallel Tests') {
     parallel {
       stage('Unit Tests') { ... }
       stage('Integration Tests') { ... }
       stage('Security Scan') { ... }
     }
   }
   ```
   **Savings: 15 minutes** (now tests run concurrently)

2. **Dependency Caching**:
   - Cache Maven dependencies (.m2 directory)
   - Cache Docker layers
   - Use Jenkins workspace caching
   **Savings: 5 minutes**

3. **Incremental Builds**:
   - Maven incremental compilation
   - Only rebuild changed modules in monorepo
   **Savings: 5 minutes**

**Phase 3: Medium-term Optimizations (Week 2-3)**

1. **Test Optimization**:
   - Run only affected tests based on changed code
   - Run critical tests first (fail fast)
   - Move slow integration tests to nightly builds
   - Parallelize test execution across multiple executors
   **Savings: 8 minutes**

2. **Docker Build Optimization**:
   - Multi-stage builds to reduce layers
   - Use specific base image tags for better caching
   - BuildKit for parallel build steps
   **Savings: 4 minutes**

3. **Pipeline Structure**:
   - Separate fast feedback pipeline (build + unit tests)
   - Slower comprehensive pipeline (all checks)
   - Let developers choose based on context

**Phase 4: Infrastructure Optimization (Week 4)**

1. **Build Agent Optimization**:
   - Right-size build agents (CPU/memory)
   - Use spot instances for cost savings
   - Implement build agent autoscaling

2. **Artifact Management**:
   - Use artifact repository proximity/caching
   - Cleanup old artifacts aggressively

**Final Result:**
- Original: 45 minutes
- After optimizations: **10 minutes** (78% improvement)
- No additional infrastructure cost
- Developers get feedback in < 15 minutes

**Metrics I'd Track**:
- Build duration trend
- Developer satisfaction surveys
- Deployment frequency
- DORA metrics improvement

**Trade-offs Considered**:
- Some tests moved to nightly (acceptable for non-critical paths)
- Slightly more complex pipeline code (managed with shared libraries)
- Need to maintain cache invalidation strategy

This is how I'd optimize while respecting budget constraints and maintaining quality."

---

**Q: You have a multi-region deployment with blue-green strategy. During a deployment to production, the green environment passes all health checks, but after switching traffic, you see a 5% error rate spike. What's your incident response process, and how do you prevent this?**

**Answer:**
"This is a critical production scenario. Here's my response:

**Immediate Response (0-5 minutes):**

1. **ROLLBACK IMMEDIATELY**:
   ```bash
   # Switch traffic back to blue
   kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
   ```
   - Don't wait to investigate - customer experience first
   - Automated rollback based on error rate threshold is ideal

2. **Incident Declaration**:
   - Open P1 incident in ServiceNow
   - Page on-call team
   - Start incident bridge
   - Notify stakeholders

**Investigation Phase (5-30 minutes):**

1. **Gather Evidence**:
   - Check Dynatrace distributed traces for failing requests
   - Review application logs for new errors
   - Compare metrics: blue vs green environment
   - Check dependency health (databases, APIs, cache)

2. **Hypothesis Generation**:
   Possible causes:
   - **Configuration drift**: Green environment has wrong config
   - **Data issue**: Database migration not completed
   - **External dependency**: Version incompatibility
   - **Load-related**: Green environment sized differently
   - **Cache warming**: Cold cache causing timeouts
   - **Network/firewall**: Security group changes

3. **Testing**:
   - Send 1% traffic to green to reproduce
   - Capture detailed traces and logs
   - Compare successful vs failed requests

**Root Cause Example Scenario:**

Let's say investigation reveals:
- Green environment connects to Redis cache cluster
- Cache was pre-warmed in blue but not in green
- Cold cache causing timeout failures
- After 5 minutes, cache warms up and errors disappear

**Prevention Strategy:**

1. **Enhanced Pre-Production Testing**:
   ```yaml
   # Add synthetic load testing before traffic switch
   - name: Load Test Green
     run: |
       k6 run --vus 100 --duration 5m load-test.js
       # Must achieve same performance as blue
   ```

2. **Progressive Traffic Shifting**:
   ```yaml
   # Instead of instant switch, gradual canary
   - 5% traffic → monitor 10 min
   - 25% traffic → monitor 10 min
   - 50% traffic → monitor 10 min
   - 100% traffic
   # Auto-rollback if error rate > threshold
   ```

3. **Environment Parity**:
   - Infrastructure-as-Code ensures identical environments
   - Automated configuration validation
   - Database migration verification as pipeline stage

4. **Comprehensive Health Checks**:
   ```python
   # Deep health check, not just HTTP 200
   def health_check():
       checks = {
           'database': check_db_connection(),
           'cache': check_cache_latency(),
           'external_api': check_api_dependency(),
           'disk_space': check_disk_space(),
       }
       return all(checks.values())
   ```

5. **Smoke Tests Post-Deployment**:
   - Run critical user journeys
   - Verify key business transactions
   - Test all integration points

6. **Monitoring & Alerting**:
   ```yaml
   # Alert on error rate increase
   - alert: ErrorRateSpike
     expr: rate(http_errors[1m]) > 0.01
     for: 1m
     annotations:
       action: Auto-rollback triggered
   ```

7. **Deployment Checklist**:
   - [ ] Database migrations completed
   - [ ] Cache pre-warmed
   - [ ] Configuration validated
   - [ ] Security groups verified
   - [ ] Dependency versions compatible
   - [ ] Rollback tested

**Post-Incident:**

1. **Blameless Postmortem**:
   - What happened?
   - Why did our checks not catch it?
   - How do we prevent recurrence?
   - Action items with owners

2. **Runbook Update**:
   - Document this scenario
   - Add to troubleshooting guide
   - Update rollback procedures

3. **Metrics**:
   - MTTR: How fast did we recover?
   - MTTD: How fast did we detect?
   - Customer impact: How many users affected?

**Architectural Improvement:**

Long-term, implement:
- Feature flags for safer releases
- Automated canary analysis
- Circuit breakers for graceful degradation
- Chaos engineering to test failure scenarios

This demonstrates both tactical incident response and strategic prevention thinking."

---

### 2. Advanced Kubernetes

**Q: Explain how Kubernetes scheduling works. A critical pod is stuck in 'Pending' state. Walk me through your complete debugging methodology, including all possible causes and how you'd investigate each.**

**Answer:**
"Excellent question. Let me cover both theory and practical debugging:

**Kubernetes Scheduling Process:**

1. **API Server** receives pod creation request
2. **Scheduler** watches for unscheduled pods
3. **Filtering Phase**: Eliminates nodes that don't meet requirements
   - Resource requirements (CPU/memory)
   - Node selectors/affinity rules
   - Taints and tolerations
   - PVCs and volume availability
4. **Scoring Phase**: Ranks suitable nodes
   - Resource balance
   - Pod spread
   - Node affinity preferences
5. **Binding**: Assigns pod to best-scored node
6. **Kubelet** on selected node pulls image and starts pod

**Systematic Debugging Methodology:**

**Step 1: Gather Basic Information (1 minute)**
```bash
# Check pod status
kubectl get pod <pod-name> -n <namespace> -o wide

# Check pod events - MOST IMPORTANT
kubectl describe pod <pod-name> -n <namespace>
# Look at Events section at bottom
```

**Possible Causes & Investigation:**

**Cause 1: Insufficient Resources**

**Symptom in Events:**
```
0/3 nodes are available: 3 Insufficient cpu.
0/3 nodes are available: 3 Insufficient memory.
```

**Investigation:**
```bash
# Check pod resource requests
kubectl get pod <pod-name> -o yaml | grep -A 5 resources

# Check node capacity
kubectl describe nodes | grep -A 5 "Allocated resources"

# Find nodes with available resources
kubectl top nodes
```

**Solution:**
- Reduce pod resource requests if over-provisioned
- Add more nodes to cluster
- Remove unused pods
- Implement Horizontal Pod Autoscaling
- Use cluster autoscaler

---

**Cause 2: PersistentVolumeClaim Not Bound**

**Symptom:**
```
pod has unbound immediate PersistentVolumeClaims
```

**Investigation:**
```bash
# Check PVC status
kubectl get pvc -n <namespace>

# Describe PVC
kubectl describe pvc <pvc-name> -n <namespace>

# Check PV availability
kubectl get pv

# Check StorageClass
kubectl get storageclass
kubectl describe storageclass <sc-name>
```

**Possible Issues:**
1. No PV available matching PVC requirements
2. StorageClass missing or misconfigured
3. Dynamic provisioner not working
4. Access mode mismatch (ReadWriteOnce vs ReadWriteMany)
5. Volume size requested not available

**Solution:**
```bash
# Fix StorageClass provisioner
kubectl edit storageclass <sc-name>

# Manually create PV if needed
kubectl apply -f pv.yaml

# Check provisioner logs
kubectl logs -n kube-system -l app=provisioner
```

---

**Cause 3: Node Selector / Affinity Issues**

**Symptom:**
```
0/3 nodes are available: 3 node(s) didn't match node selector.
```

**Investigation:**
```bash
# Check pod node selector
kubectl get pod <pod-name> -o yaml | grep -A 3 nodeSelector

# Check node labels
kubectl get nodes --show-labels

# Check if required label exists
kubectl get nodes -l disktype=ssd
```

**Solution:**
- Add required label to nodes: `kubectl label nodes <node-name> disktype=ssd`
- Or modify pod spec to remove/fix node selector

---

**Cause 4: Taints and Tolerations**

**Symptom:**
```
0/3 nodes are available: 3 node(s) had taints that the pod didn't tolerate.
```

**Investigation:**
```bash
# Check node taints
kubectl describe nodes | grep Taints

# Check pod tolerations
kubectl get pod <pod-name> -o yaml | grep -A 5 tolerations
```

**Common Taints:**
- `node.kubernetes.io/not-ready`
- `node.kubernetes.io/unreachable`
- `node.kubernetes.io/disk-pressure`
- Custom taints: `dedicated=gpu:NoSchedule`

**Solution:**
```yaml
# Add toleration to pod spec
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

---

**Cause 5: Pod Affinity/Anti-Affinity**

**Symptom:**
```
0/3 nodes are available: 3 node(s) didn't match pod affinity rules.
```

**Investigation:**
```bash
# Check pod affinity rules
kubectl get pod <pod-name> -o yaml | grep -A 10 affinity

# Check existing pods distribution
kubectl get pods -n <namespace> -o wide --show-labels
```

**Solution:**
- Review and fix affinity rules
- Or add matching pods to enable scheduling

---

**Cause 6: Image Pull Issues**

**Symptom:**
Pod might be in ImagePullBackOff after scheduling

**Investigation:**
```bash
# Check image pull secrets
kubectl get pod <pod-name> -o yaml | grep imagePullSecrets

# Test image pull manually
kubectl run test --image=<image> --dry-run=client
```

---

**Cause 7: Admission Webhook Blocking**

**Symptom:**
No events, pod just stuck

**Investigation:**
```bash
# Check admission webhooks
kubectl get validatingwebhookconfigurations
kubectl get mutatingwebhookconfigurations

# Check API server logs
kubectl logs -n kube-system <api-server-pod>
```

---

**Step 2: Check Scheduler Logs (if issue not obvious)**
```bash
# Find scheduler pod
kubectl get pods -n kube-system | grep scheduler

# Check scheduler logs
kubectl logs -n kube-system kube-scheduler-<node> | grep <pod-name>
```

---

**Step 3: Check Node Conditions**
```bash
# Nodes might be NotReady, have disk pressure, etc.
kubectl get nodes
kubectl describe node <node-name>

# Check for:
# - Ready status
# - DiskPressure
# - MemoryPressure
# - PIDPressure
# - NetworkUnavailable
```

---

**Real Production Example from Birlasoft:**

**Scenario:** StatefulSet pod stuck in Pending

**Investigation:**
```bash
kubectl describe pod postgres-0
# Event: pod has unbound PersistentVolumeClaims

kubectl get pvc
# STATUS: Pending

kubectl describe pvc postgres-pvc-0
# Event: waiting for first consumer to be created before binding
```

**Root Cause:** 
- StorageClass had `volumeBindingMode: WaitForFirstConsumer`
- Node didn't have compatible topology (was using wrong AZ)

**Solution:**
```bash
# Added topology constraints to StatefulSet
volumeClaimTemplates:
  spec:
    storageClassName: gp2
    # Added allowed topologies
    allowedTopologies:
    - matchLabelExpressions:
      - key: topology.kubernetes.io/zone
        values:
        - us-east-1a
```

---

**Preventive Measures:**

1. **Resource Management:**
   - Set resource quotas and limit ranges
   - Monitor cluster capacity
   - Implement cluster autoscaling

2. **Validation:**
   - Use admission controllers (OPA/Gatekeeper)
   - Pre-deployment validation in CI/CD
   - Dry-run deployments: `kubectl apply --dry-run=server`

3. **Monitoring:**
   - Alert on pending pods > 5 minutes
   - Dashboard showing pod scheduling rates
   - Node resource utilization alerts

4. **Documentation:**
   - Maintain runbook for common scheduling issues
   - Document cluster-specific constraints

This comprehensive approach ensures quick resolution regardless of the root cause."

---

### 3. Advanced Terraform

**Q: You're managing Terraform state for a large infrastructure with multiple teams. Team A accidentally deleted the Terraform state file from S3. Team B just applied changes 5 minutes ago. Team C is planning to apply changes now. How do you handle this disaster scenario? Walk me through your recovery process and what safeguards you'd implement.**

**Answer:**
"This is a nightmare scenario but recoverable. Here's my approach:

**IMMEDIATE ACTIONS (Next 5 minutes):**

**Step 1: STOP ALL TERRAFORM OPERATIONS**
```bash
# Emergency broadcast to all teams
# Slack/Email: "STOP - Terraform state disaster - Do NOT run any terraform commands"

# If possible, temporarily revoke S3 permissions
aws s3api put-bucket-policy --bucket terraform-state --policy deny-all.json

# Lock state in DynamoDB if not already locked
aws dynamodb put-item --table-name terraform-locks \
  --item '{"LockID": {"S": "emergency-lock"}}'
```

**Step 2: ASSESS THE DAMAGE**
```bash
# Check S3 bucket versioning (THIS IS CRITICAL)
aws s3api list-object-versions --bucket terraform-state --prefix prod/

# Find latest state file version before deletion
aws s3api list-object-versions --bucket terraform-state \
  --prefix prod/terraform.tfstate \
  --query 'Versions[?IsLatest==`false`] | [0]'
```

---

**RECOVERY PROCESS:**

**Scenario A: S3 Versioning Enabled (Best Case)**

```bash
# Step 1: Restore from previous version
aws s3api get-object --bucket terraform-state \
  --key prod/terraform.tfstate \
  --version-id <version-id-before-deletion> \
  restored-state.tfstate

# Step 2: Verify state file integrity
terraform show restored-state.tfstate | head -50
# Check serial number, last update time

# Step 3: Compare with Team B's changes
# If Team B's changes were applied, we need their serial number
# Contact Team B: "What was the serial number after your apply?"

# Step 4: Restore appropriate version
# If Team B's state (serial 45) is newer than deleted state (serial 43)
# Find Team B's state version
aws s3api list-object-versions --bucket terraform-state \
  --query 'Versions[?VersionId==`<team-b-version-id>`]'

# Step 5: Restore to S3
aws s3 cp restored-state.tfstate s3://terraform-state/prod/terraform.tfstate
```

---

**Scenario B: No Versioning / Need to Rebuild State**

This is painful but doable:

```bash
# Step 1: Import existing infrastructure
# Create a script to reimport all resources

# Get list of all resources from actual infrastructure
# For AWS:
aws ec2 describe-instances --filters "Name=tag:ManagedBy,Values=Terraform"
aws rds describe-db-instances
# ... etc for all resource types

# Step 2: Initialize new empty state
terraform init

# Step 3: Import each resource
# Create import script:
cat > import_all.sh << 'EOF'
#!/bin/bash

# VPCs
terraform import aws_vpc.main vpc-12345678

# Subnets
terraform import aws_subnet.public_a subnet-abcd1234
terraform import aws_subnet.public_b subnet-efgh5678

# EC2 Instances
terraform import aws_instance.app_server[0] i-0123456789abcdef0
terraform import aws_instance.app_server[1] i-0123456789abcdef1

# RDS
terraform import aws_db_instance.postgres mydb-instance-id

# S3 Buckets
terraform import aws_s3_bucket.data my-data-bucket

# ... continue for ALL resources
EOF

chmod +x import_all.sh
./import_all.sh
```

**Challenge with importing:**
- Must import in dependency order
- Count and for_each resources are tricky
- Module resources need full path: `module.network.aws_vpc.main`

---

**Step 4: Reconcile Team B's Changes**

```bash
# Team B applied changes - we need to incorporate them

# Option 1: If Team B's code is in Git
git log --since="2 hours ago" -- terraform/
# Find their commit, cherry-pick if needed

# Option 2: Ask Team B for their state backup
# They should have local .terraform/terraform.tfstate backup

# Option 3: Compare infrastructure current state vs code
terraform plan
# Review differences
# Some resources might show as drift (Team B's changes)
```

---

**Step 5: Validation**

```bash
# Critical validation before allowing team access
terraform plan -detailed-exitcode
# Exit code 0: No changes needed (perfect)
# Exit code 2: Changes needed (investigate each)

# Validate no resources will be destroyed inadvertently
terraform plan -out=recovery.tfplan
terraform show -json recovery.tfplan | \
  jq '.resource_changes[] | select(.change.actions[] == "delete")'

# If any deletes show up - STOP and investigate
# Real infrastructure should not be deleted
```

---

**SAFEGUARDS TO IMPLEMENT (Post-Incident):**

**1. S3 State Backend Hardening**

```hcl
# terraform/backend.tf
terraform {
  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    
    # Critical: Enable versioning
    # Set in S3 bucket configuration
  }
}
```

```bash
# Enable versioning on state bucket
aws s3api put-bucket-versioning \
  --bucket terraform-state-prod \
  --versioning-configuration Status=Enabled

# Enable MFA Delete (ultimate protection)
aws s3api put-bucket-versioning \
  --bucket terraform-state-prod \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::123456789:mfa/root-account-mfa-device 123456"

# Lifecycle policy to retain versions
aws s3api put-bucket-lifecycle-configuration \
  --bucket terraform-state-prod \
  --lifecycle-configuration file://lifecycle.json
```

```json
// lifecycle.json
{
  "Rules": [
    {
      "Id": "RetainStateVersions",
      "Status": "Enabled",
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    }
  ]
}
```

---

**2. Access Controls**

```json
// S3 Bucket Policy - Deny delete without MFA
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDeleteWithoutMFA",
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteObjectVersion"
      ],
      "Resource": "arn:aws:s3:::terraform-state-prod/*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

---

**3. State Backup Automation**

```bash
# Automated state backup to separate location
# Run as cron job or Lambda

#!/bin/bash
# backup-terraform-state.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_BUCKET="terraform-state-backups-prod"

# Copy current state to backup bucket
aws s3 cp s3://terraform-state-prod/prod/terraform.tfstate \
  s3://$BACKUP_BUCKET/backups/$TIMESTAMP/terraform.tfstate

# Keep backups for 1 year
aws s3api put-bucket-lifecycle-configuration \
  --bucket $BACKUP_BUCKET \
  --lifecycle-configuration '{
    "Rules": [{
      "Id": "ExpireOldBackups",
      "Status": "Enabled",
      "Expiration": {"Days": 365}
    }]
  }'
```

---

**4. State Locking Monitoring**

```python
# Monitor for state locks held too long
import boto3
from datetime import datetime, timedelta

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('terraform-state-lock')

# Alert if lock held > 30 minutes
def check_stale_locks():
    response = table.scan()
    for item in response['Items']:
        created = datetime.fromisoformat(item['Created'])
        if datetime.now() - created > timedelta(minutes=30):
            send_alert(f"Stale lock: {item['LockID']}")
```

---

**5. Workspace Isolation**

```hcl
# Separate state files per environment/team
terraform {
  backend "s3" {
    bucket = "terraform-state-prod"
    key    = "team-a/${terraform.workspace}/terraform.tfstate"
    # Each team/workspace gets separate state
  }
}
```

---

**6. Pre-Apply Validation**

```bash
# CI/CD Pipeline safeguards
# .gitlab-ci.yml or Jenkinsfile

validate:
  script:
    # Download current state
    - terraform init
    # Create plan
    - terraform plan -out=tfplan
    # Validate no unexpected deletions
    - |
      DELETIONS=$(terraform show -json tfplan | \
        jq '[.resource_changes[] | select(.change.actions[] == "delete")] | length')
      if [ $DELETIONS -gt 0 ]; then
        echo "WARNING: Plan includes $DELETIONS deletions"
        # Require manual approval
        exit 1
      fi
```

---

**7. Change Approval Process**

```yaml
# Terraform Cloud / Enterprise
# Require approvals for applies

organization:
  terraform-prod:
    # Policies
    policy_sets:
      - name: "prevent-deletions"
        policies:
          - deletion-protection.sentinel
    
    # VCS integration
    vcs_repo: "github.com/company/terraform"
    
    # Apply requires approval
    require_manual_apply: true
    
    # Speculative plans for PRs
    speculative_enabled: true
```

---

**8. Regular State Audits**

```bash
# Weekly state audit script
#!/bin/bash

# Compare state with actual infrastructure
terraform refresh
terraform plan -detailed-exitcode

# Exit code 2 means drift detected
if [ $? -eq 2 ]; then
  # Generate drift report
  terraform show -no-color > drift-report.txt
  # Send to team
  mail -s "Terraform Drift Detected" team@company.com < drift-report.txt
fi
```

---

**9. Documentation & Runbooks**

```markdown
# Runbook: Terraform State Disaster Recovery

## Symptoms
- Missing state file
- Corrupted state
- Concurrent modification

## Recovery Steps
1. STOP all Terraform operations
2. Check S3 versioning
3. Restore from version history
4. Validate restored state
5. Run terraform plan
6. Document incident

## Contacts
- On-call: +1-555-0100
- Team Lead: alice@company.com
- Cloud Admin: bob@company.com
```

---

**10. Team Training**

```yaml
# Quarterly Training Topics
- State file architecture
- Disaster recovery drill
- Proper git workflow
- State file do's and don'ts
- When to ask for help
```

---

**Post-Incident Review:**

1. **Timeline Documentation**:
   - When was state deleted?
   - Who was affected?
   - How long was recovery?

2. **Root Cause Analysis**:
   - How did deletion occur?
   - Why didn't safeguards prevent it?
   - What was the human factor?

3. **Action Items**:
   - [ ] Enable S3 MFA Delete (Owner: DevOps Lead, Due: 1 week)
   - [ ] Implement automated backups (Owner: SRE, Due: 2 weeks)
   - [ ] Team training session (Owner: Manager, Due: 1 month)
   - [ ] Update runbooks (Owner: DevOps, Due: 3 days)

4. **Metrics**:
   - Detection time: 0 minutes (immediate)
   - Recovery time: X hours
   - Business impact: None (caught in time)

This comprehensive approach ensures both recovery capability and prevention of recurrence."

---

### 4. Complex Production Scenario

**Q: Production is down. You have 100 microservices in Kubernetes. The monitoring dashboard shows elevated latency across ALL services starting exactly 15 minutes ago. No deployments happened. How do you troubleshoot this? What's your methodology?**

**Answer:**
"This is a systematic incident response scenario. Time is critical, but panic leads to mistakes. Here's my approach:

**INCIDENT COMMAND (Minute 0-2):**

```bash
# Declare P1 Incident
# Open incident bridge
# Assign roles:
# - Incident Commander (me)
# - Communication lead
# - Technical investigators

# First message to stakeholders:
"P1 Incident: Elevated latency across all services. 
Team investigating. ETA 15 minutes for initial findings."
```

---

**INITIAL ASSESSMENT (Minute 2-5):**

**Key Principle: Common cause across ALL services = infrastructure issue, NOT application issue**

```bash
# Quick health checks - work from bottom to top of stack

# 1. CLUSTER HEALTH
kubectl get nodes
kubectl describe nodes | grep -i "condition\|pressure"

# Look for:
# - Node NotReady
# - MemoryPressure
# - DiskPressure
# - PIDPressure

# 2. CONTROL PLANE
kubectl get componentstatuses  # Deprecated but useful
kubectl get pods -n kube-system | grep -v Running

# 3. CLUSTER METRICS
kubectl top nodes
kubectl top pods --all-namespaces | sort -k 3 -nr | head -20
```

---

**PARALLEL INVESTIGATION TRACKS (Minute 5-15):**

**Track 1: Network Layer**

```bash
# Check network plugin
kubectl get pods -n kube-system | grep -E "calico|cilium|flannel|weave"

# Check CNI issues
kubectl logs -n kube-system -l k8s-app=calico-node --tail=100

# DNS issues (common culprit)
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=100

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# Check CoreDNS config
kubectl get configmap -n kube-system coredns -o yaml
```

**Track 2: Infrastructure Events**

```bash
# Cluster events (often revealing)
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -50

# Look for patterns:
# - OOMKilled events
# - ImagePullBackOff
# - Node state changes
# - Volume issues

# Check for recent changes in infrastructure
# AWS example:
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ModifyDBInstance \
  --start-time $(date -u -d '30 minutes ago' '+%Y-%m-%dT%H:%M:%S') \
  --max-results 50
```

**Track 3: External Dependencies**

```bash
# Database connection pool exhaustion?
# Check RDS metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name DatabaseConnections \
  --dimensions Name=DBInstanceIdentifier,Value=prod-db \
  --start-time $(date -u -d '30 minutes ago' '+%Y-%m-%dT%H:%M:%S') \
  --end-time $(date -u '+%Y-%m-%dT%H:%M:%S') \
  --period 300 \
  --statistics Maximum

# Redis/ElastiCache issues?
aws cloudwatch get-metric-statistics \
  --namespace AWS/ElastiCache \
  --metric-name CPUUtilization \
  --dimensions Name=CacheClusterId,Value=prod-redis \
  --start-time $(date -u -d '30 minutes ago' '+%Y-%m-%dT%H:%M:%S') \
  --end-time $(date -u '+%Y-%m-%dT%H:%M:%S') \
  --period 300 \
  --statistics Maximum
```

**Track 4: Monitoring Deep Dive**

```bash
# Dynatrace - Check distributed traces
# Look for:
# - Where is latency being added?
# - External API calls timing out?
# - Database query slow downs?

# Prometheus queries
# Network latency
sum(rate(http_request_duration_seconds_sum[5m])) 
  by (service, status_code)

# Pod restarts
sum(kube_pod_container_status_restarts_total) by (namespace, pod)

# Resource saturation
node_filesystem_avail_bytes / node_filesystem_size_bytes < 0.1
```

---

**LIKELY ROOT CAUSES & DETECTION:**

**Root Cause 1: DNS Resolution Issues (MOST COMMON)**

**Symptoms:**
```bash
# CoreDNS pods OOMKilled or crashed
kubectl get pods -n kube-system -l k8s-app=kube-dns

# High DNS query rate
kubectl logs -n kube-system -l k8s-app=kube-dns | grep -i "cache"
```

**Fix:**
```bash
# Scale up CoreDNS
kubectl scale deployment -n kube-system coredns --replicas=5

# Increase memory limits
kubectl edit deployment -n kube-system coredns
# Change memory limit from 170Mi to 512Mi

# Add caching to application level (long-term)
```

---

**Root Cause 2: Node Resource Exhaustion**

**Detection:**
```bash
kubectl describe node <node-name>

# Shows:
Conditions:
  MemoryPressure   True    NodeHasInsufficientMemory
  DiskPressure     True    NodeHasDiskPressure
```

**Impact:**
- Kubelet throttles pods
- Evicts pods randomly
- Causes cascading failures

**Fix:**
```bash
# Emergency: Cordon node
kubectl cordon <node-name>

# Drain non-critical workloads
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Add more nodes (autoscaling should do this)
# Or manually add nodes

# Long-term: Right-size resource requests/limits
```

---

**Root Cause 3: Database Connection Pool Exhaustion**

**Detection:**
```bash
# Check RDS connections
aws rds describe-db-instances --db-instance-identifier prod-db \
  --query 'DBInstances[0].DBParameterGroups[0].DBParameterGroupName'

# Get connection count
mysql -h prod-db.xxx.rds.amazonaws.com -e "SHOW STATUS LIKE 'Threads_connected';"
mysql -h prod-db.xxx.rds.amazonaws.com -e "SHOW VARIABLES LIKE 'max_connections';"
```

**Symptoms:**
- All services slow (all need DB)
- Started at specific time (connection pool filled up)
- Application logs showing connection timeouts

**Fix:**
```bash
# Immediate: Increase max_connections
aws rds modify-db-parameter-group \
  --db-parameter-group-name prod-params \
  --parameters "ParameterName=max_connections,ParameterValue=500,ApplyMethod=immediate"

# Reboot DB instance if needed (last resort)
aws rds reboot-db-instance --db-instance-identifier prod-db

# Long-term:
# - Implement connection pooling properly
# - Review slow queries
# - Add read replicas
# - Implement caching layer
```

---

**Root Cause 4: Cluster Autoscaler Issues**

**Detection:**
```bash
# Check cluster autoscaler logs
kubectl logs -n kube-system -l app=cluster-autoscaler --tail=100

# Common issues:
# "scale up failed: instance type not available in AZ"
# "max node count reached"
# "insufficient capacity"

# Check pending pods
kubectl get pods --all-namespaces --field-selector=status.phase=Pending
```

**Fix:**
```bash
# Manual scale up
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name eks-node-group \
  --desired-capacity 20

# Review autoscaler config
kubectl edit deployment -n kube-system cluster-autoscaler
```

---

**Root Cause 5: Certificate Expiration**

**Detection:**
```bash
# Check cert expiry
kubectl get secrets --all-namespaces -o json | \
  jq -r '.items[] | select(.type=="kubernetes.io/tls") | 
  {namespace: .metadata.namespace, name: .metadata.name, 
  cert: .data."tls.crt"}' | \
  while read secret; do
    echo $secret | openssl x509 -noout -enddate
  done

# Check ingress TLS
kubectl describe ingress --all-namespaces | grep -i tls
```

**Fix:**
```bash
# Renew cert-manager certificates
kubectl annotate certificate <cert-name> \
  -n <namespace> \
  cert-manager.io/issue-temporary-certificate="true"

# Manual cert renewal if needed
```

---

**REAL PRODUCTION EXAMPLE:**

At Birlasoft, we had exactly this scenario:

**Timeline:**
- **14:23**: All services latency spike
- **14:25**: P1 incident declared
- **14:30**: Investigation ongoing

**Investigation:**
```bash
# Checked DNS - normal
# Checked nodes - normal
# Checked database - AH HA!

aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name DatabaseConnections \
  --dimensions Name=DBInstanceIdentifier,Value=prod-postgres \
  --start-time 14:00 --end-time 14:30 \
  --period 60 --statistics Maximum

# Output: Connections jumped from 200 to 495 (max: 500)
```

**Root Cause:**
- New marketing campaign launched at 14:20
- 5x traffic increase
- Database connection pool saturated
- All services waiting for connections

**Immediate Fix:**
```bash
# Increased max_connections
aws rds modify-db-parameter-group \
  --db-parameter-group-name prod-params \
  --parameters "ParameterName=max_connections,ParameterValue=1000,ApplyMethod=immediate"

# Killed long-running queries
psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE state = 'idle in transaction' AND query_start < now() - interval '5 minutes';"
```

**Recovery:**
- **14:42**: Latency returning to normal
- **14:50**: All metrics green
- **MTTR: 27 minutes**

---

**POST-INCIDENT ACTIONS:**

**Immediate (Same Day):**
```bash
# Document timeline
# Capture all metrics/logs
# Write incident summary
```

**Short-term (This Week):**
```markdown
1. Implement RDS connection monitoring alerts
2. Add connection pool metrics to dashboards
3. Load test database layer
4. Review connection pool configs in all services
5. Add circuit breakers to prevent cascade
```

**Long-term (This Month):**
```markdown
1. Read replica setup for read traffic
2. Caching layer (Redis) for frequent queries
3. Database query optimization
4. Implement rate limiting
5. Chaos engineering to test failure scenarios
```

**Blameless Postmortem:**
```markdown
# What Went Well:
- Fast incident detection (2 minutes)
- Clear incident command structure
- Systematic troubleshooting

# What Didn't Go Well:
- No alerting for DB connection saturation
- No load testing for marketing campaigns
- Reactive rather than proactive

# Action Items:
- [ ] Add DB connection alerts (Owner: DevOps, Due: 2 days)
- [ ] Implement chaos testing (Owner: SRE, Due: 2 weeks)
- [ ] Marketing campaign checklist (Owner: PM, Due: 1 week)
- [ ] Load testing automation (Owner: DevOps, Due: 2 weeks)
```

---

**METHODOLOGY SUMMARY:**

1. **Stay Calm** - Panic causes mistakes
2. **Incident Command** - Clear roles and communication
3. **Work Bottom-Up** - Infrastructure → Platform → Application
4. **Look for Common Causes** - All services = infrastructure
5. **Parallel Investigation** - Multiple tracks simultaneously
6. **Time-based Correlation** - "Started 15 min ago" is a clue
7. **External Dependencies** - Often the culprit
8. **Metrics Over Speculation** - Data-driven decisions
9. **Document Everything** - For postmortem
10. **Learn and Improve** - Every incident is a learning opportunity

This systematic approach has served me well in multiple production incidents at both Birlasoft and Infosys."

---

### 5. Advanced Security & Compliance

**Q: You need to implement a security scanning pipeline that fails builds if critical vulnerabilities are found, but your developers are complaining that this is blocking legitimate work because there are always some CVEs in base images. How do you balance security with velocity? Design a practical solution.**

**Answer:**
"This is a classic security vs. velocity tension. Here's my pragmatic approach:

**The Problem:**
- Security wants zero critical vulnerabilities
- Developers want to ship features fast
- Base images always have some CVEs
- Blanket blocking creates friction and workarounds

**My Solution: Risk-Based Tiered Approach**

---

**Tier 1: BLOCK - Critical Exploitable Vulnerabilities**

Block only if ALL these conditions are met:
1. **Critical or High severity** (CVSS >= 7.0)
2. **Exploitability: Active exploitation in the wild** OR **Public exploit exists**
3. **Reachability: Vulnerable component is actually used** (not just present)
4. **Fix available** (not waiting for upstream)

**Implementation:**
```yaml
# .gitlab-ci.yml security scan stage
security_scan:
  stage: security
  script:
    - |
      # Scan image
      trivy image --severity CRITICAL,HIGH --format json \
        --output scan-results.json $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      
      # Custom filtering logic
      python3 << 'EOF'
      import json
      import sys
      
      with open('scan-results.json') as f:
          results = json.load(f)
      
      blocking_vulns = []
      
      for result in results.get('Results', []):
          for vuln in result.get('Vulnerabilities', []):
              # Check our blocking criteria
              severity = vuln.get('Severity')
              fixed_version = vuln.get('FixedVersion')
              vuln_id = vuln.get('VulnerabilityID')
              
              # Query exploit database
              has_exploit = check_exploit_db(vuln_id)
              is_reachable = check_reachability(vuln.get('PkgName'))
              
              if (severity in ['CRITICAL', 'HIGH'] and 
                  has_exploit and 
                  is_reachable and 
                  fixed_version):
                  blocking_vulns.append(vuln)
      
      if blocking_vulns:
          print(f"BLOCKING: {len(blocking_vulns)} exploitable vulnerabilities")
          for v in blocking_vulns:
              print(f"  - {v['VulnerabilityID']}: {v['Title']}")
          sys.exit(1)
      else:
          print("Security scan passed")
      EOF
  allow_failure: false
```

---

**Tier 2: WARN - Other High/Critical Issues**

Don't block, but make developers aware:

```yaml
security_report:
  stage: security
  script:
    - trivy image --severity CRITICAL,HIGH $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  allow_failure: true
  artifacts:
    reports:
      container_scanning: gl-container-scanning-report.json
```

Post results to:
- PR comments (automated bot)
- Security dashboard
- Weekly team email digest

---

**Tier 3: Track - Medium/Low Issues**

```yaml
# Nightly full scan
nightly_security_scan:
  stage: security
  script:
    - trivy image --severity MEDIUM,LOW,CRITICAL,HIGH \
        --format json --output full-scan.json \
        $CI_REGISTRY_IMAGE:latest
    - upload-to-security-tracking-system full-scan.json
  only:
    - schedules
```

---

**Exception Process:**

Create a vulnerability exception workflow:

```yaml
# exceptions.yaml - version controlled
exceptions:
  - cve: CVE-2024-12345
    package: openssl
    reason: "No fix available, waiting for openssl 3.1.5"
    approved_by: security-team@company.com
    expiry_date: 2024-03-31
    compensating_controls:
      - "Running behind WAF with specific rule"
      - "Network segmentation prevents exploit"
    ticket: SEC-1234
  
  - cve: CVE-2024-67890
    package: libxml2
    reason: "Not exploitable - we don't parse untrusted XML"
    approved_by: security-team@company.com
    expiry_date: 2024-04-15
    ticket: SEC-1235
```

**Exception workflow:**
1. Developer creates PR with exception request
2. Security team reviews (async, SLA: 4 hours)
3. If approved, merged to exceptions.yaml
4. Pipeline checks exceptions file before failing

```python
# check-exceptions.py
def check_exception(vuln_id, package):
    with open('exceptions.yaml') as f:
        exceptions = yaml.safe_load(f)
    
    for exc in exceptions.get('exceptions', []):
        if (exc['cve'] == vuln_id and 
            exc['package'] == package and
            datetime.now() < datetime.fromisoformat(exc['expiry_date'])):
            print(f"Exception granted: {exc['reason']}")
            return True
    return False
```

---

**Base Image Strategy:**

**1. Curated Base Images:**
```dockerfile
# Instead of:
FROM node:18

# Use company-maintained:
FROM company-registry.com/hardened/node:18-2024-02-01
# Pre-scanned, patched, approved base images
# Updated weekly with security patches
```

**2. Base Image Pipeline:**
```yaml
# base-images/.gitlab-ci.yml
build_base_image:
  script:
    # Start with minimal image
    - docker build -t base-node:18 .
    
    # Scan
    - trivy image base-node:18
    
    # Patch what we can
    - docker run base-node:18 apt-get update && apt-get upgrade -y
    
    # Re-scan
    - trivy image base-node:18 --format json > scan.json
    
    # Approve if acceptable
    - |
      if python check-scan.py scan.json; then
        docker tag base-node:18 company-registry.com/approved/node:18-$(date +%Y-%m-%d)
        docker push company-registry.com/approved/node:18-$(date +%Y-%m-%d)
      fi
  schedule:
    - cron: "0 2 * * 1"  # Weekly Monday 2 AM
```

**3. Mandate Approved Base Images:**
```yaml
# OPA Policy (Open Policy Agent)
package docker.base_images

deny[msg] {
  input.Stages[_].From
  not startswith(input.Stages[_].From, "company-registry.com/approved/")
  msg = "Use only approved base images from company-registry.com/approved/"
}
```

---

**Developer Experience Improvements:**

**1. Fast Feedback:**
```bash
# Pre-commit hook
# .git/hooks/pre-commit
#!/bin/bash
if git diff --cached --name-only | grep -q Dockerfile; then
  echo "Dockerfile changed, running quick security scan..."
  docker build -t temp-image .
  trivy image --severity CRITICAL --exit-code 1 temp-image
fi
```

**2. IDE Integration:**
```yaml
# VS Code extension settings
# Scan on file save
"docker.scan.onSave": true
"docker.scan.severity": "CRITICAL,HIGH"
```

**3. Developer Dashboard:**
```
┌─────────────────────────────────────────┐
│ Your Security Posture                   │
├─────────────────────────────────────────┤
│ ✅ service-a: No critical issues        │
│ ⚠️  service-b: 2 high issues (excepted) │
│ ❌ service-c: 1 critical - BLOCKED      │
│    └─ CVE-2024-12345: RCE in libcurl   │
│       Fix: Update to libcurl 8.5.0      │
│       Command: apt-get install libcurl4 │
└─────────────────────────────────────────┘
```

---

**Metrics & Governance:**

**1. SLA Tracking:**
```yaml
Metrics:
  - Mean time to patch critical CVE: < 48 hours
  - % images with zero critical: > 95%
  - Security scan success rate: > 98%
  - Exception approval time: < 4 hours
```

**2. Monthly Security Review:**
```markdown
# February 2024 Security Report
- Total scans: 1,247
- Blocked builds: 12 (0.96%)
- Exceptions granted: 8
- Average remediation time: 18 hours
- Top vulnerabilities:
  1. CVE-2024-XXX (libssl): 23 occurrences
  2. CVE-2024-YYY (node): 15 occurrences
```

**3. Gamification:**
```
🏆 Security Champions Leaderboard
1. Backend Team: 45 days CVE-free
2. Frontend Team: 38 days CVE-free
3. Data Team: 22 days CVE-free
```

---

**Continuous Improvement:**

**Quarterly Reviews:**
```markdown
1. Review exception list
   - Are long-standing exceptions still needed?
   - Can we update dependencies?

2. Adjust blocking criteria
   - Are we too strict?
   - Are we missing exploitable vulns?

3. Base image refresh
   - New minimal images available?
   - Dependencies needing updates?

4. Developer feedback
   - Survey: Is security pipeline helpful or hindrance?
   - Incorporate feedback
```

---

**Communication Strategy:**

**1. Transparent Communication:**
```markdown
# Weekly Team Email
Subject: Security Scan Update - Week of Feb 5

Hi Team,

Security scan stats this week:
- 247 builds scanned
- 2 critical issues found and fixed
- 1 exception requested (approved)

🎉 Shoutout to @alice for quickly patching CVE-2024-12345

Common issue this week:
- Outdated npm packages
- Fix: Run `npm update` before committing

Questions? #security-help Slack channel
```

**2. Runbooks:**
```markdown
# Runbook: Security Scan Failed

## Symptom
Pipeline failed with "Security scan blocking critical CVE"

## Resolution
1. Review the scan output
2. Check if fix is available
   - Yes: Update dependency, rebuild
   - No: Request exception (see process below)

## Requesting Exception
1. Create ticket: https://jira/SEC
2. Include:
   - CVE ID
   - Why not fixable now
   - Compensating controls
   - Timeline for fix
3. Tag @security-team
4. SLA: 4 hour response

## Emergency Override
If production is down:
- Use override: `CI_SECURITY_OVERRIDE=true`
- Post-incident exception required within 24h
```

---

**Success Criteria:**

After implementing this system at Birlasoft:

**Before:**
- Developers bypassing security scans
- 40+ critical CVEs in production
- Adversarial relationship with security team
- Slow vulnerability remediation (weeks)

**After:**
- 98% scan adoption
- <5 critical CVEs in production (all excepted)
- Collaborative security culture
- Fast remediation (median 18 hours)

**Key Success Factors:**
1. **Balanced approach** - not too strict, not too loose
2. **Fast exception process** - removes blocker for legit cases
3. **Developer-friendly** - automated, fast feedback
4. **Metrics-driven** - track and improve
5. **Curated base images** - solve problem at source
6. **Clear communication** - no surprises

This approach maintains security posture while enabling developer velocity."

---

## CLOSING QUESTIONS

### For You to Ask the Interviewer

**About the Role:**
1. "What's the current state of CI/CD maturity? Are you looking to build from scratch or optimize existing pipelines?"

2. "What's the team structure? Will I be working with a dedicated DevOps team or embedded with application teams?"

3. "What's your deployment frequency currently, and what's the goal?"

**About Technology:**
4. "I see GCP is preferred - what's the current cloud footprint? Multi-cloud or GCP-focused?"

5. "What's the Kubernetes setup? Self-managed or GKE? How many clusters?"

6. "What's the current observability stack? Are you using Google Cloud Operations or third-party tools?"

**About Culture:**
7. "How does the organization handle the on-call rotation? What's the incident frequency?"

8. "What's the biggest DevOps challenge the team is facing right now?"

9. "How do you balance feature velocity with reliability/SRE practices?"

**About Growth:**
10. "What does success look like for this role in the first 90 days? First year?"

---

## KEY TAKEAWAYS FOR YOUR INTERVIEW

**Your Strengths to Emphasize:**
1. ✅ **5 years relevant experience** (meets requirement)
2. ✅ **Strong CI/CD experience** (Jenkins, automation)
3. ✅ **Container orchestration** (Kubernetes production experience)
4. ✅ **IaC expertise** (Terraform, Ansible)
5. ✅ **Multi-cloud** (AWS + VMware, can adapt to GCP)
6. ✅ **Incident response** (production troubleshooting)
7. ✅ **ITIL knowledge** (ServiceNow, release management)

**Areas to Address:**
1. **GCP Experience**: "While my primary cloud experience is AWS, I'm certified in AWS and have strong fundamentals that translate across clouds. I'm eager to deepen GCP expertise."

2. **Harness**: "I haven't used Harness specifically, but I have extensive CI/CD experience with Jenkins and GitHub Actions. The concepts are transferable, and I'm a quick learner."

3. **Shift Work**: "I'm comfortable with shift work and have experience with on-call rotations. Reliability is critical, and I'm committed to supporting 24/7 operations."

**Your Competitive Advantages:**
1. Recent experience at financial services (Northern Trust) - understanding of compliance/governance
2. Production incident response experience - real troubleshooting under pressure
3. Automation mindset - reduced manual effort by 30%
4. Cross-functional collaboration - worked with dev, QA, security teams
5. Observability implementation - Dynatrace, Prometheus expertise

**Confidence Builders:**
- You have ALL the core requirements
- Your experience aligns well with the JD
- You've solved real production problems
- You understand both technical and process aspects (ITIL + SRE)

**Good Luck! You've got this! 🚀**
