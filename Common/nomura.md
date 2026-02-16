# TOC Automation Engineer Interview Preparation
## Based on Resume: Harshal Kalamkar & Nomura Job Description

---

## SECTION 1: BASIC LEVEL QUESTIONS

### 1. TOC Automation & Your Approach

**Q: What does TOC (Technical Operations Center) automation mean to you? How would you approach reducing manual effort in a large enterprise environment?**

**Answer:**
"TOC automation is about transforming repetitive operational tasks into automated, self-service, and proactive processes to improve efficiency, reduce human error, and enable operations teams to focus on strategic initiatives rather than routine work.

**My Approach to Reducing Manual Effort:**

**1. Identify High-Impact Areas (First 30 Days):**
```
Manual Effort Assessment:
├── Ticket Analysis
│   ├── Most frequent incident types
│   ├── Average resolution time
│   └── Tickets requiring manual intervention
│
├── Operational Tasks
│   ├── Server provisioning time
│   ├── Deployment frequency
│   └── Patching cycles
│
└── Team Time Tracking
    ├── Where do ops spend most time?
    ├── Which tasks are repetitive?
    └── What causes escalations?
```

**Example from Birlasoft:**
I analyzed 3 months of ServiceNow tickets and found:
- 35% were restart requests (applications, services)
- 25% were log collection requests
- 15% were disk space cleanup
- 10% were basic troubleshooting (connectivity checks)

**2. Prioritization Matrix:**
```
High Impact, Low Effort:
  ✓ Application restart automation (implemented first)
  ✓ Log collection scripts
  ✓ Disk cleanup automation

High Impact, High Effort:
  → Automated deployment pipelines (implemented later)
  → Self-healing infrastructure
  → Predictive alerting

Low Impact, Low Effort:
  → Quick wins for team morale
  → Documentation automation

Low Impact, High Effort:
  → Defer or reject
```

**3. Implementation Strategy:**

**Phase 1: Quick Wins (Month 1-2)**
```yaml
# Example: Service Restart Automation
# Previously: 15 minutes manual work per request
# After: 30 seconds self-service

ServiceNow Catalog Item: "Restart Application Service"
  ├── User selects application
  ├── Triggers Jenkins job
  ├── Jenkins calls Ansible playbook
  ├── Playbook restarts service safely
  ├── Health check performed
  └── Ticket auto-updated with results

Impact: 300 restart requests/month × 15 min = 75 hours saved
```

**Phase 2: Foundation Building (Month 3-4)**
```
Infrastructure as Code:
  ├── Terraform for infrastructure provisioning
  ├── Ansible for configuration management
  ├── Git for version control
  └── Jenkins/AWX for orchestration

Observability:
  ├── Centralized logging (ELK/Splunk)
  ├── Metrics collection (Prometheus)
  ├── Dashboards (Grafana)
  └── Alerting (PagerDuty/Everbridge)
```

**Phase 3: Advanced Automation (Month 5-6)**
```
Self-Healing Systems:
  ├── Auto-restart unhealthy services
  ├── Auto-scale based on load
  ├── Auto-remediation for known issues
  └── Predictive alerting

Integration & Orchestration:
  ├── ServiceNow → Jenkins → Ansible
  ├── Monitoring → Auto-remediation
  ├── CI/CD pipeline automation
  └── Infrastructure lifecycle automation
```

**4. Measuring Success:**
```
Metrics to Track:

Efficiency Metrics:
  - Manual ticket volume (target: -50% in 6 months)
  - Mean Time To Repair (MTTR)
  - Time spent on toil vs. strategic work
  - Deployment frequency
  - Lead time for changes

Quality Metrics:
  - Incident volume
  - Change failure rate
  - Service availability (SLO compliance)
  - Number of emergency changes

Business Impact:
  - Cost savings (hours saved × hourly rate)
  - Revenue protected (uptime improvements)
  - Team satisfaction scores
```

**5. Real Example from Birlasoft:**

**Before Automation:**
```
Monthly Operations Metrics:
  - Incident tickets: 1,200
  - Manual intervention needed: 900 (75%)
  - Average handling time: 20 minutes
  - Total manual hours: 300 hours/month
  - After-hours escalations: 150/month
  - Team burnout: High
```

**After 6 Months of Automation:**
```
Monthly Operations Metrics:
  - Incident tickets: 1,000 (↓17%)
  - Manual intervention needed: 400 (40%, ↓47%)
  - Average handling time: 12 minutes (↓40%)
  - Total manual hours: 80 hours/month (↓73%)
  - After-hours escalations: 50/month (↓67%)
  - Team satisfaction: Significantly improved
  
Cost Savings: 220 hours × $75/hour = $16,500/month
Annual Savings: ~$200,000
```

**Automation Examples Implemented:**

1. **Server Provisioning:**
   - Before: 2 hours manual work
   - After: 10 minutes self-service
   - Tool: ServiceNow + Terraform + Ansible

2. **Application Deployment:**
   - Before: 45 minutes manual deployment
   - After: 8 minutes automated pipeline
   - Tool: Jenkins + Ansible + Docker

3. **Log Collection:**
   - Before: 15 minutes per request
   - After: Automated via AWX job template
   - Tool: AWX + Ansible

4. **Health Checks:**
   - Before: Manual checks every 2 hours
   - After: Continuous monitoring with auto-alerts
   - Tool: Prometheus + Grafana + AlertManager

**Key Success Factors:**

1. **Start Small:** Quick wins build momentum
2. **Measure Everything:** Data drives decisions
3. **Collaborate:** Work with app teams, infra teams
4. **Document:** Runbooks, playbooks, procedures
5. **Train:** Enable team to use new tools
6. **Iterate:** Continuous improvement mindset

**For Nomura, I would:**

1. **Assess Current State:** Analyze ticket patterns, identify pain points
2. **Gap Analysis:** Compare current vs. desired automation maturity
3. **Build Roadmap:** Prioritized automation initiatives
4. **Align with Standards:** Use Nomura's frameworks and best practices
5. **Pilot & Scale:** Start with one team/app, then expand
6. **Measure & Report:** Track metrics, demonstrate ROI

This data-driven, incremental approach has consistently delivered results in reducing manual effort while improving service quality."

---

### 2. CI/CD Pipeline Design & Implementation

**Q: Describe how you would design a CI/CD pipeline for a large enterprise environment. What tools would you use and why?**

**Answer:**
"In a large enterprise like Nomura, CI/CD pipelines need to be secure, compliant, scalable, and standardized across teams.

**Enterprise CI/CD Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    DEVELOPER WORKFLOW                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Developer → Git Commit → Pull Request → Code Review       │
│                                                             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  CI/CD ORCHESTRATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────┐       │
│  │           Jenkins / GitLab CI                   │       │
│  │         (Centralized Orchestration)             │       │
│  └──────────────────┬──────────────────────────────┘       │
│                     │                                       │
└─────────────────────┼───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌───────────┐   ┌──────────┐   ┌──────────┐
│   BUILD   │   │   TEST   │   │ SECURITY │
└─────┬─────┘   └────┬─────┘   └────┬─────┘
      │              │              │
      ▼              ▼              ▼
┌─────────────────────────────────────────┐
│         Maven/Gradle                    │
│         npm/yarn                        │
│         Docker Build                    │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Unit Tests                      │
│         Integration Tests               │
│         E2E Tests                       │
│         Performance Tests               │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         SonarQube (Code Quality)        │
│         Snyk/Trivy (Vulnerability)      │
│         SAST (Static Analysis)          │
│         DAST (Dynamic Analysis)         │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│     ARTIFACT MANAGEMENT                 │
│     JFrog Artifactory / Nexus           │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────┐
│                 DEPLOYMENT STAGES                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  DEV → QA → UAT → STAGING → [APPROVAL] → PRODUCTION    │
│                                                         │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌────────────┐     │
│  │ Auto │   │ Auto │   │Manual│   │   Manual   │     │
│  │Deploy│   │Deploy│   │Deploy│   │   Approval │     │
│  └──────┘   └──────┘   └──────┘   └────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│     DEPLOYMENT TOOLS                    │
│     Ansible / Terraform                 │
│     Kubernetes (kubectl/helm)           │
│     AWS CDK / CloudFormation            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│     POST-DEPLOYMENT                     │
│     - Smoke Tests                       │
│     - Health Checks                     │
│     - Rollback on Failure               │
│     - Notifications                     │
└─────────────────────────────────────────┘
```

---

**Detailed Pipeline Stages:**

**Stage 1: Source Control & Triggers**

```groovy
// Jenkinsfile
pipeline {
    agent {
        label 'docker-enabled'
    }
    
    triggers {
        // Trigger on Git commit
        pollSCM('H/5 * * * *')
        
        // Trigger on PR
        githubPullRequests()
    }
    
    options {
        // Build timeout
        timeout(time: 1, unit: 'HOURS')
        
        // Keep last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // Timestamps in console
        timestamps()
        
        // Concurrent builds
        disableConcurrentBuilds()
    }
    
    environment {
        // Environment variables
        APP_NAME = 'user-service'
        ARTIFACTORY_URL = credentials('artifactory-url')
        DOCKER_REGISTRY = 'registry.nomura.com'
        SONARQUBE_URL = credentials('sonarqube-url')
        
        // Dynamic versioning
        VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
    }
    
    stages {
        // Pipeline stages defined below
    }
    
    post {
        always {
            // Cleanup
            cleanWs()
        }
        success {
            // Notifications
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Build #${env.BUILD_NUMBER} succeeded"
            )
        }
        failure {
            // Failure notifications
            slackSend(
                channel: '#alerts',
                color: 'danger',
                message: "❌ Build #${env.BUILD_NUMBER} failed"
            )
            emailext(
                to: '${DEFAULT_RECIPIENTS}',
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Check console output: ${env.BUILD_URL}"
            )
        }
    }
}
```

---

**Stage 2: Build**

```groovy
stage('Build') {
    steps {
        script {
            echo "Building ${APP_NAME} version ${VERSION}"
            
            // Java application example
            sh '''
                mvn clean package -DskipTests \
                    -Drevision=${VERSION} \
                    -s settings.xml
            '''
            
            // Archive artifacts
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            
            // Stash for later stages
            stash includes: 'target/*.jar', name: 'app-artifact'
        }
    }
}
```

---

**Stage 3: Unit Tests**

```groovy
stage('Unit Tests') {
    steps {
        sh '''
            mvn test \
                -Drevision=${VERSION} \
                -Djacoco.destFile=target/jacoco.exec
        '''
    }
    post {
        always {
            // Publish test results
            junit 'target/surefire-reports/*.xml'
            
            // Code coverage
            jacoco(
                execPattern: 'target/jacoco.exec',
                classPattern: 'target/classes',
                sourcePattern: 'src/main/java'
            )
        }
    }
}
```

---

**Stage 4: Code Quality & Security Scanning**

```groovy
stage('Code Quality') {
    parallel {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${APP_NAME} \
                            -Dsonar.projectVersion=${VERSION}
                    '''
                }
                
                // Wait for quality gate
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Vulnerability Scan') {
            steps {
                // Dependency check
                sh '''
                    mvn org.owasp:dependency-check-maven:check
                '''
                
                // Publish results
                dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
            }
        }
        
        stage('License Compliance') {
            steps {
                sh '''
                    mvn license:add-third-party
                    mvn license:aggregate-download-licenses
                '''
            }
        }
    }
}
```

---

**Stage 5: Docker Build**

```groovy
stage('Docker Build') {
    steps {
        script {
            // Build image
            docker.build(
                "${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}",
                "--build-arg VERSION=${VERSION} ."
            )
            
            // Tag as latest
            sh """
                docker tag ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION} \
                    ${DOCKER_REGISTRY}/${APP_NAME}:latest
            """
            
            // Scan image
            sh """
                trivy image --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    ${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}
            """
        }
    }
}
```

---

**Stage 6: Integration Tests**

```groovy
stage('Integration Tests') {
    agent {
        label 'docker-compose'
    }
    
    steps {
        script {
            // Start test environment
            sh '''
                docker-compose -f docker-compose.test.yml up -d
                
                # Wait for services
                ./wait-for-services.sh
                
                # Run integration tests
                mvn verify -Pintegration-tests
                
                # Cleanup
                docker-compose -f docker-compose.test.yml down -v
            '''
        }
    }
    
    post {
        always {
            junit 'target/failsafe-reports/*.xml'
        }
    }
}
```

---

**Stage 7: Publish Artifacts**

```groovy
stage('Publish Artifacts') {
    steps {
        script {
            // Push to Artifactory
            withCredentials([usernamePassword(
                credentialsId: 'artifactory-creds',
                usernameVariable: 'USER',
                passwordVariable: 'PASS'
            )]) {
                sh """
                    curl -u ${USER}:${PASS} \
                        -T target/${APP_NAME}-${VERSION}.jar \
                        ${ARTIFACTORY_URL}/${APP_NAME}/${VERSION}/
                """
            }
            
            // Push Docker image
            docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-creds') {
                docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${VERSION}").push()
                docker.image("${DOCKER_REGISTRY}/${APP_NAME}:latest").push()
            }
        }
    }
}
```

---

**Stage 8: Deploy to Environments**

```groovy
stage('Deploy to DEV') {
    steps {
        script {
            // Deploy using Ansible
            ansiblePlaybook(
                playbook: 'deploy-app.yml',
                inventory: 'inventories/dev/hosts.ini',
                extras: "-e app_version=${VERSION} -e environment=dev",
                credentialsId: 'ansible-ssh-key'
            )
            
            // Or deploy to Kubernetes
            sh """
                kubectl set image deployment/${APP_NAME} \
                    ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${VERSION} \
                    --namespace=dev \
                    --record
                
                # Wait for rollout
                kubectl rollout status deployment/${APP_NAME} \
                    --namespace=dev \
                    --timeout=5m
            """
        }
    }
}

stage('Deploy to QA') {
    when {
        branch 'develop'
    }
    steps {
        script {
            ansiblePlaybook(
                playbook: 'deploy-app.yml',
                inventory: 'inventories/qa/hosts.ini',
                extras: "-e app_version=${VERSION} -e environment=qa",
                credentialsId: 'ansible-ssh-key'
            )
        }
    }
}

stage('Deploy to Staging') {
    when {
        branch 'main'
    }
    steps {
        script {
            // Manual approval for staging
            timeout(time: 1, unit: 'HOURS') {
                input message: 'Deploy to Staging?', ok: 'Deploy'
            }
            
            ansiblePlaybook(
                playbook: 'deploy-app.yml',
                inventory: 'inventories/staging/hosts.ini',
                extras: "-e app_version=${VERSION} -e environment=staging",
                credentialsId: 'ansible-ssh-key'
            )
        }
    }
}

stage('Deploy to Production') {
    when {
        branch 'main'
    }
    steps {
        script {
            // Change management integration
            def changeTicket = input(
                message: 'Enter Change Request Number:',
                parameters: [string(name: 'CR_NUMBER', description: 'ServiceNow CR')]
            )
            
            // Verify CR is approved
            def crStatus = sh(
                script: """
                    curl -s -u ${SERVICENOW_USER}:${SERVICENOW_PASS} \
                        "${SERVICENOW_URL}/api/now/table/change_request/${changeTicket}" \
                        | jq -r '.result.state'
                """,
                returnStdout: true
            ).trim()
            
            if (crStatus != 'Approved') {
                error("Change Request ${changeTicket} is not approved")
            }
            
            // Production deployment with additional safety
            ansiblePlaybook(
                playbook: 'deploy-app.yml',
                inventory: 'inventories/production/hosts.ini',
                extras: "-e app_version=${VERSION} -e environment=production -e change_request=${changeTicket}",
                credentialsId: 'ansible-ssh-key'
            )
        }
    }
}
```

---

**Stage 9: Post-Deployment Validation**

```groovy
stage('Post-Deployment Validation') {
    steps {
        script {
            // Health checks
            sh """
                # Application health
                curl -f http://app.nomura.com/actuator/health || exit 1
                
                # Smoke tests
                ./run-smoke-tests.sh production
                
                # Performance baseline
                ab -n 1000 -c 10 http://app.nomura.com/api/health
            """
            
            // Update ServiceNow
            sh """
                curl -X PATCH \
                    -u ${SERVICENOW_USER}:${SERVICENOW_PASS} \
                    ${SERVICENOW_URL}/api/now/table/change_request/${changeTicket} \
                    -H "Content-Type: application/json" \
                    -d '{
                        "state": "Implemented",
                        "close_notes": "Deployed version ${VERSION} successfully"
                    }'
            """
        }
    }
}
```

---

**Tool Selection Rationale:**

**1. Jenkins:**
```
Pros:
  ✓ Industry standard, mature
  ✓ Extensive plugin ecosystem
  ✓ Self-hosted (meets compliance requirements)
  ✓ Strong integration capabilities
  ✓ Supports complex workflows

Cons:
  ✗ Requires maintenance
  ✗ Can be complex to configure

Use When: Enterprise environment with strict compliance needs
```

**2. GitLab CI:**
```
Pros:
  ✓ Git-native, simpler configuration
  ✓ Built-in container registry
  ✓ Excellent Kubernetes integration
  ✓ Auto DevOps features

Cons:
  ✗ Less mature plugin ecosystem vs Jenkins
  ✗ May require SaaS version for full features

Use When: Cloud-native applications, simpler pipelines
```

**3. GitHub Actions:**
```
Pros:
  ✓ Tight GitHub integration
  ✓ Easy to get started
  ✓ Large marketplace

Cons:
  ✗ Limited to GitHub
  ✗ Cost considerations at scale

Use When: GitHub-based workflows, open source projects
```

---

**Enterprise Best Practices:**

**1. Pipeline as Code:**
```
✓ Store Jenkinsfile in Git
✓ Version control all pipeline configs
✓ Peer review pipeline changes
✓ Use shared libraries for common functions
```

**2. Security & Compliance:**
```
✓ Secrets management (HashiCorp Vault)
✓ Credential rotation
✓ Audit logging
✓ Signed commits
✓ Binary authorization
✓ RBAC for pipeline access
```

**3. Standardization:**
```
✓ Common pipeline templates
✓ Shared Jenkins libraries
✓ Standard naming conventions
✓ Consistent environments
```

**4. Observability:**
```
✓ Pipeline metrics in Grafana
✓ Build time trends
✓ Failure rate tracking
✓ Deployment frequency
✓ Lead time measurement (DORA metrics)
```

**5. Scalability:**
```
✓ Jenkins agents on Kubernetes
✓ Auto-scaling build capacity
✓ Distributed builds
✓ Caching strategies
```

---

**Real Production Example from Infosys:**

```yaml
Application: Financial Trading Platform
Components: 15 microservices
Deployment Frequency: 3-5 times/day per service
Environments: DEV → QA → UAT → STAGING → PROD

Pipeline Characteristics:
  - Average build time: 12 minutes
  - Average deployment time: 8 minutes
  - Success rate: 94%
  - Rollback capability: < 5 minutes
  
Compliance Requirements:
  - All deployments require approved change requests
  - 4-eye principle for production
  - Audit trail in ServiceNow
  - Segregation of duties
  - Regular security scans

Tools Used:
  - Jenkins (orchestration)
  - Maven (build)
  - JFrog Artifactory (artifact storage)
  - Docker (containerization)
  - Ansible (deployment)
  - SonarQube (code quality)
  - Trivy (security scanning)
  - ServiceNow (change management)
  - Slack (notifications)
```

**For Nomura, I would recommend:**

1. **Centralized Jenkins** with Kubernetes-based agents
2. **JFrog Artifactory** for artifact management (already industry standard in finance)
3. **Ansible Tower/AWX** for deployment automation
4. **SonarQube** for code quality
5. **Trivy/Snyk** for security scanning
6. **ServiceNow integration** for change management
7. **Grafana** for pipeline observability
8. **Everbridge** integration for critical deployment notifications

This enterprise-grade approach balances automation with compliance, security, and governance requirements typical in financial services."

---

### 3. Observability: SLIs, SLOs, and SLA Implementation

**Q: Explain SLIs, SLOs, and SLAs. How would you implement an observability framework aligned with these principles?**

**Answer:**
"Observability is critical for TOC operations. Understanding SLIs, SLOs, and SLAs helps us measure, monitor, and maintain service reliability.

**Definitions:**

**SLI (Service Level Indicator):**
- **What:** Quantitative measure of service behavior
- **Examples:** Request latency, error rate, availability, throughput
- **Think:** The actual measurements

**SLO (Service Level Objective):**
- **What:** Target value or range for an SLI
- **Examples:** 99.9% availability, p95 latency < 200ms
- **Think:** What we promise ourselves internally

**SLA (Service Level Agreement):**
- **What:** Contractual agreement with consequences
- **Examples:** 99.95% uptime or customer gets refund
- **Think:** What we promise customers/business

**Relationship:**
```
SLA (Contract)
  └── SLO (Target)
        └── SLI (Measurement)

Example:
  SLA: "Service will be available 99.9% of time or credits issued"
  SLO: "Service will be available 99.95% of time" (buffer)
  SLI: "% of successful requests" (how we measure)
```

---

**Implementing Observability Framework:**

**1. Define SLIs:**

```yaml
# Example for a trading application at Nomura

Service: Trade Execution API

SLIs:
  availability:
    name: "API Availability"
    description: "Percentage of successful requests"
    measurement: |
      (successful_requests / total_requests) * 100
    data_source: "Application logs + Load balancer metrics"
    
  latency:
    name: "Request Latency"
    description: "Time to process a trade request"
    measurement: |
      p50, p95, p99 of request duration
    data_source: "Application metrics (Prometheus)"
    
  error_rate:
    name: "Error Rate"
    description: "Percentage of failed requests"
    measurement: |
      (failed_requests / total_requests) * 100
    data_source: "Application logs"
    
  correctness:
    name: "Trade Correctness"
    description: "Trades executed correctly"
    measurement: |
      (correct_trades / total_trades) * 100
    data_source: "Reconciliation system"
    
  throughput:
    name: "Trades Per Second"
    description: "System capacity utilization"
    measurement: |
      count(trades) per second
    data_source: "Prometheus"
```

**2. Set SLOs:**

```yaml
# SLO Definition

Service: Trade Execution API
SLO Period: 30 days (rolling window)

Objectives:
  - name: "Availability"
    target: 99.95%
    sli: availability
    error_budget: 0.05% (21.6 minutes per month)
    
  - name: "Latency P95"
    target: "< 200ms"
    sli: latency
    percentile: 95
    
  - name: "Latency P99"
    target: "< 500ms"
    sli: latency
    percentile: 99
    
  - name: "Error Rate"
    target: "< 0.1%"
    sli: error_rate
    
  - name: "Trade Correctness"
    target: "100%"
    sli: correctness
    priority: critical  # Zero tolerance

Alerting:
  - condition: "Error budget < 10% remaining"
    action: "Warning notification to team"
    
  - condition: "Error budget depleted"
    action: "Stop non-critical deployments"
    
  - condition: "SLO breach"
    action: "Page on-call engineer"
```

**3. Technical Implementation:**

**Architecture:**

```
┌────────────────────────────────────────────────────────┐
│              APPLICATION LAYER                         │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ Microservice │  │ Microservice │  │ Microservice│ │
│  │      A       │  │      B       │  │      C      │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬──────┘ │
│         │                 │                 │         │
│         └─────────────────┼─────────────────┘         │
│                           │                           │
└───────────────────────────┼───────────────────────────┘
                            │
                            │ Metrics, Logs, Traces
                            │
┌───────────────────────────▼───────────────────────────┐
│           OBSERVABILITY PLATFORM                      │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ Prometheus  │  │     ELK     │  │   Jaeger    │  │
│  │  (Metrics)  │  │   (Logs)    │  │  (Traces)   │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  │
│         │                │                │          │
│         └────────────────┼────────────────┘          │
│                          │                           │
│                ┌─────────▼─────────┐                 │
│                │     Grafana       │                 │
│                │   (Dashboards)    │                 │
│                └─────────┬─────────┘                 │
│                          │                           │
│                ┌─────────▼─────────┐                 │
│                │   Prometheus      │                 │
│                │   AlertManager    │                 │
│                └─────────┬─────────┘                 │
│                          │                           │
└──────────────────────────┼───────────────────────────┘
                           │
                           │ Alerts
                           │
┌──────────────────────────▼────────────────────────────┐
│           NOTIFICATION & ESCALATION                   │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌────────────┐  ┌────────────┐  ┌────────────────┐ │
│  │   Slack    │  │ PagerDuty  │  │  Everbridge    │ │
│  │            │  │            │  │  (Critical)    │ │
│  └────────────┘  └────────────┘  └────────────────┘ │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

**Implementation Code:**

**A. Instrumentation (Application Side):**

```java
// Spring Boot application with Micrometer
@RestController
@RequestMapping("/api/trades")
public class TradeController {
    
    private final MeterRegistry meterRegistry;
    private final Counter tradesCounter;
    private final Timer tradesTimer;
    
    public TradeController(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        // Initialize metrics
        this.tradesCounter = Counter.builder("trades.executed")
            .description("Total number of trades executed")
            .tag("service", "trade-api")
            .register(meterRegistry);
        
        this.tradesTimer = Timer.builder("trades.latency")
            .description("Trade execution latency")
            .tag("service", "trade-api")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(meterRegistry);
    }
    
    @PostMapping
    public ResponseEntity<Trade> executeTrade(@RequestBody TradeRequest request) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            Trade trade = tradeService.execute(request);
            
            // Record success
            tradesCounter.increment();
            sample.stop(tradesTimer);
            
            meterRegistry.counter("trades.success",
                "type", request.getType(),
                "status", "success"
            ).increment();
            
            return ResponseEntity.ok(trade);
            
        } catch (Exception e) {
            // Record failure
            meterRegistry.counter("trades.failed",
                "type", request.getType(),
                "error", e.getClass().getSimpleName()
            ).increment();
            
            sample.stop(tradesTimer);
            throw e;
        }
    }
}
```

```yaml
# application.yml - Prometheus metrics endpoint
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus,metrics
  metrics:
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http.server.requests: true
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT}
```

---

**B. Prometheus Configuration:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'nomura-prod'
    datacenter: 'dc1'

# Scrape configurations
scrape_configs:
  - job_name: 'trade-api'
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - trading
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
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

# SLO recording rules
rule_files:
  - /etc/prometheus/rules/slo_rules.yml
```

**SLO Recording Rules:**

```yaml
# rules/slo_rules.yml
groups:
  - name: slo_trade_api
    interval: 30s
    rules:
      # Availability SLI
      - record: slo:trade_api:availability:ratio
        expr: |
          sum(rate(http_server_requests_seconds_count{
            job="trade-api",
            status!~"5.."
          }[5m]))
          /
          sum(rate(http_server_requests_seconds_count{
            job="trade-api"
          }[5m]))
      
      # Error rate SLI
      - record: slo:trade_api:error_rate:ratio
        expr: |
          sum(rate(http_server_requests_seconds_count{
            job="trade-api",
            status=~"5.."
          }[5m]))
          /
          sum(rate(http_server_requests_seconds_count{
            job="trade-api"
          }[5m]))
      
      # Latency P95 SLI
      - record: slo:trade_api:latency:p95
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_server_requests_seconds_bucket{
              job="trade-api"
            }[5m])) by (le)
          )
      
      # Latency P99 SLI
      - record: slo:trade_api:latency:p99
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_server_requests_seconds_bucket{
              job="trade-api"
            }[5m])) by (le)
          )
      
      # Error budget calculation (30 day window)
      - record: slo:trade_api:error_budget:remaining
        expr: |
          1 - (
            (1 - slo:trade_api:availability:ratio)
            /
            (1 - 0.9995)  # Target: 99.95%
          )
      
      # Burn rate (how fast we're consuming error budget)
      - record: slo:trade_api:error_budget:burn_rate
        expr: |
          (1 - slo:trade_api:availability:ratio)
          /
          (1 - 0.9995)
```

---

**C. Alerting Rules:**

```yaml
# rules/slo_alerts.yml
groups:
  - name: slo_alerts
    rules:
      # Fast burn: 2% error budget consumed in 1 hour
      - alert: SLOErrorBudgetFastBurn
        expr: |
          slo:trade_api:error_budget:burn_rate > 14.4
          and
          slo:trade_api:error_budget:remaining < 0.98
        for: 5m
        labels:
          severity: critical
          slo: trade_api
        annotations:
          summary: "Fast error budget burn detected"
          description: |
            Error budget burning at {{ $value }}x rate.
            At this rate, budget will be exhausted in {{ $value | humanizeDuration }}.
            Remaining budget: {{ $labels.remaining }}%
      
      # Slow burn: 10% error budget consumed in 3 days
      - alert: SLOErrorBudgetSlowBurn
        expr: |
          slo:trade_api:error_budget:burn_rate > 1
          and
          slo:trade_api:error_budget:remaining < 0.90
        for: 1h
        labels:
          severity: warning
          slo: trade_api
        annotations:
          summary: "Slow error budget burn detected"
          description: "Error budget at {{ $labels.remaining }}%"
      
      # SLO breach
      - alert: SLOBreach
        expr: |
          slo:trade_api:availability:ratio < 0.9995
        for: 5m
        labels:
          severity: critical
          slo: trade_api
        annotations:
          summary: "SLO BREACH: Availability below 99.95%"
          description: |
            Current availability: {{ $value | humanizePercentage }}
            This violates our SLO commitment.
      
      # Latency SLO breach
      - alert: LatencySLOBreach
        expr: |
          slo:trade_api:latency:p95 > 0.2
        for: 5m
        labels:
          severity: warning
          slo: trade_api
        annotations:
          summary: "P95 latency above 200ms"
          description: "Current P95 latency: {{ $value | humanizeDuration }}"
```

---

**D. Alert Routing (AlertManager):**

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

# Route configuration
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical SLO breaches -> Everbridge (critical escalation)
    - match:
        severity: critical
        slo: trade_api
      receiver: 'everbridge-critical'
      continue: true
    
    # Critical also goes to PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty-oncall'
      continue: true
    
    # Warnings to Slack
    - match:
        severity: warning
      receiver: 'slack-alerts'

# Receiver configurations
receivers:
  - name: 'default'
    slack_configs:
      - api_url: '{{ slack_webhook_url }}'
        channel: '#monitoring'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'slack-alerts'
    slack_configs:
      - api_url: '{{ slack_webhook_url }}'
        channel: '#alerts'
        color: 'warning'
        title: '⚠️ {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'pagerduty-oncall'
    pagerduty_configs:
      - service_key: '{{ pagerduty_service_key }}'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
  
  - name: 'everbridge-critical'
    webhook_configs:
      - url: 'https://api.everbridge.net/rest/notifications'
        http_config:
          basic_auth:
            username: '{{ everbridge_user }}'
            password: '{{ everbridge_pass }}'
        send_resolved: true
```

---

**E. Grafana Dashboards:**

```json
// SLO Dashboard (Grafana JSON model)
{
  "dashboard": {
    "title": "Trade API SLO Dashboard",
    "panels": [
      {
        "title": "Availability SLO",
        "targets": [{
          "expr": "slo:trade_api:availability:ratio * 100"
        }],
        "thresholds": [
          {"value": 99.95, "color": "green"},
          {"value": 99.90, "color": "yellow"},
          {"value": 0, "color": "red"}
        ]
      },
      {
        "title": "Error Budget Remaining",
        "targets": [{
          "expr": "slo:trade_api:error_budget:remaining * 100"
        }],
        "gauge": {
          "maxValue": 100,
          "minValue": 0,
          "thresholds": [
            {"value": 20, "color": "red"},
            {"value": 50, "color": "yellow"},
            {"value": 100, "color": "green"}
          ]
        }
      },
      {
        "title": "Latency Percentiles",
        "targets": [
          {"expr": "slo:trade_api:latency:p50", "legendFormat": "P50"},
          {"expr": "slo:trade_api:latency:p95", "legendFormat": "P95"},
          {"expr": "slo:trade_api:latency:p99", "legendFormat": "P99"}
        ],
        "yAxis": {"format": "ms"}
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "slo:trade_api:error_rate:ratio * 100"
        }],
        "yAxis": {"format": "percent"}
      },
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "sum(rate(http_server_requests_seconds_count{job='trade-api'}[5m]))"
        }],
        "yAxis": {"format": "reqps"}
      },
      {
        "title": "Error Budget Burn Rate (30d)",
        "targets": [{
          "expr": "slo:trade_api:error_budget:burn_rate"
        }],
        "thresholds": [
          {"value": 1, "color": "green"},
          {"value": 5, "color": "yellow"},
          {"value": 10, "color": "red"}
        ]
      }
    ]
  }
}
```

---

**F. ServiceNow Integration:**

```python
# slo_reporter.py
# Automated SLO reporting to ServiceNow

import requests
from datetime import datetime, timedelta
import prometheus_api_client

class SLOReporter:
    def __init__(self, prometheus_url, servicenow_url, servicenow_auth):
        self.prom = prometheus_api_client.PrometheusConnect(url=prometheus_url)
        self.snow_url = servicenow_url
        self.snow_auth = servicenow_auth
    
    def calculate_slo_compliance(self, service, period_days=30):
        """Calculate SLO compliance over period"""
        end_time = datetime.now()
        start_time = end_time - timedelta(days=period_days)
        
        # Query Prometheus
        availability = self.prom.custom_query_range(
            query=f'slo:{service}:availability:ratio',
            start_time=start_time,
            end_time=end_time,
            step='1h'
        )
        
        # Calculate average availability
        values = [float(v[1]) for v in availability[0]['values']]
        avg_availability = sum(values) / len(values)
        
        # Calculate error budget
        target_availability = 0.9995  # 99.95%
        error_budget = 1 - target_availability
        error_budget_consumed = (1 - avg_availability) / error_budget
        
        return {
            'service': service,
            'period_days': period_days,
            'availability': avg_availability * 100,
            'target': target_availability * 100,
            'error_budget_consumed': error_budget_consumed * 100,
            'slo_met': avg_availability >= target_availability
        }
    
    def create_slo_report(self, service):
        """Create monthly SLO report in ServiceNow"""
        compliance = self.calculate_slo_compliance(service, period_days=30)
        
        # Create incident if SLO not met
        if not compliance['slo_met']:
            data = {
                'short_description': f"SLO Breach: {service}",
                'description': f"""
                    Service: {service}
                    Actual Availability: {compliance['availability']:.4f}%
                    Target: {compliance['target']:.4f}%
                    Error Budget Consumed: {compliance['error_budget_consumed']:.2f}%
                    
                    This requires immediate attention and root cause analysis.
                """,
                'impact': '1',  # High
                'urgency': '1',  # High
                'category': 'SLO Breach',
                'assignment_group': 'TOC Operations'
            }
            
            response = requests.post(
                f'{self.snow_url}/api/now/table/incident',
                auth=self.snow_auth,
                json=data,
                headers={'Content-Type': 'application/json'}
            )
            
            return response.json()
        
        return {'status': 'SLO Met', 'compliance': compliance}

# Usage
reporter = SLOReporter(
    prometheus_url='http://prometheus.nomura.com',
    servicenow_url='https://nomura.service-now.com',
    servicenow_auth=('user', 'pass')
)

# Generate monthly reports
report = reporter.create_slo_report('trade_api')
print(report)
```

---

**G. Error Budget Policy:**

```yaml
# error-budget-policy.yml
# Automated actions based on error budget consumption

service: trade-api

error_budget:
  target_availability: 99.95%  # SLO
  window: 30 days
  
policies:
  - name: "Normal Operations"
    condition: "error_budget_remaining > 80%"
    actions:
      - allow_all_deployments
      - normal_release_cadence
      - focus_on_features
  
  - name: "Caution"
    condition: "error_budget_remaining 50-80%"
    actions:
      - require_sre_approval_for_risky_changes
      - increase_testing_rigor
      - review_monitoring_coverage
      - notify_leadership
  
  - name: "Restricted"
    condition: "error_budget_remaining 20-50%"
    actions:
      - freeze_non_critical_deployments
      - focus_on_reliability
      - daily_war_room_meetings
      - escalate_to_director
  
  - name: "Emergency"
    condition: "error_budget_remaining < 20%"
    actions:
      - freeze_all_deployments
      - all_hands_on_reliability
      - executive_escalation
      - postmortem_required
      - stop_feature_development

automation:
  # Automatically block deployments when budget low
  jenkins_integration:
    enabled: true
    query: "slo:trade_api:error_budget:remaining"
    threshold: 0.20
    action: "block_deployment"
    message: |
      🚫 Deployment blocked: Error budget < 20%
      Current: {{ error_budget_remaining }}%
      Focus on reliability before deploying new features.
      Override requires Director approval.
```

---

**Real Production Metrics (from Birlasoft):**

```yaml
Service: User Authentication API
SLO Period: Q4 2023 (90 days)

Targets:
  - Availability: 99.95% (21.6 min downtime/month)
  - Latency P95: < 200ms
  - Error Rate: < 0.1%

Actual Performance:
  - Availability: 99.97% ✅ (exceeded target)
  - Downtime: 13.2 minutes (38% better than target)
  - Latency P95: 145ms ✅ (27% better)
  - Latency P99: 280ms ✅
  - Error Rate: 0.04% ✅ (60% better)

Incidents:
  - Total: 3 major incidents
  - Root Causes:
    1. Database connection pool exhaustion (8 min outage)
    2. Memory leak in application (4 min outage)
    3. Network partition (1.2 min outage)
  - All incidents had postmortems
  - Action items: 100% completed

Error Budget:
  - Budget: 21.6 minutes
  - Consumed: 13.2 minutes (61%)
  - Remaining: 8.4 minutes (39%)
  - Status: Healthy ✅

Improvements Made:
  - Implemented circuit breakers
  - Added connection pool monitoring
  - Memory leak detection automation
  - Network redundancy improvements
```

---

**For Nomura Implementation:**

**Phase 1: Foundation (Month 1-2)**
1. Define critical services and their SLIs
2. Set realistic SLOs based on current performance
3. Implement instrumentation (Prometheus metrics)
4. Build basic dashboards (Grafana)

**Phase 2: Monitoring (Month 3-4)**
5. Configure alerting (AlertManager)
6. Integrate with Everbridge for critical alerts
7. Establish on-call rotation
8. Create runbooks for common issues

**Phase 3: Automation (Month 5-6)**
9. Implement error budget automation
10. ServiceNow integration for reporting
11. Automated SLO reports to leadership
12. CI/CD integration (block deploys when budget low)

**Phase 4: Optimization (Month 7+)**
13. Refine SLOs based on data
14. Implement predictive alerting
15. Build self-healing automation
16. Continuous improvement

This comprehensive observability framework ensures we measure what matters, detect issues proactively, and maintain high service reliability—critical for financial services operations at Nomura."

---

(Continuing in next response due to length...)

### 4. ServiceNow Integration & Automation

**Q: How would you integrate ServiceNow with your automation framework to reduce manual ticketing and improve operational efficiency?**

**Answer:**
"ServiceNow is the operational hub in most enterprises. Integrating it with automation can dramatically reduce manual work.

**Integration Architecture:**

```
┌──────────────────────────────────────────────────────┐
│                  SERVICENOW                          │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │
│  │  Incident  │  │  Request   │  │    Change    │  │
│  │  (INC)     │  │  (RITM)    │  │    (CHG)     │  │
│  └─────┬──────┘  └─────┬──────┘  └──────┬───────┘  │
│        │               │                │           │
│        └───────────────┼────────────────┘           │
│                        │                            │
│                ┌───────▼────────┐                   │
│                │   MID Server   │                   │
│                │ (On-prem proxy)│                   │
│                └───────┬────────┘                   │
│                        │                            │
└────────────────────────┼────────────────────────────┘
                         │
                         │ REST API / Webhooks
                         │
┌────────────────────────▼────────────────────────────┐
│              AUTOMATION LAYER                       │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │   Jenkins    │  │  Ansible AWX │  │  Scripts │ │
│  │              │  │              │  │          │ │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘ │
│         │                 │                │       │
│         └─────────────────┼────────────────┘       │
│                           │                        │
└───────────────────────────┼────────────────────────┘
                            │
                            │ Actions
                            │
┌───────────────────────────▼────────────────────────┐
│              TARGET SYSTEMS                        │
├────────────────────────────────────────────────────┤
│  Servers | Applications | Cloud | Network          │
└────────────────────────────────────────────────────┘
```

---

**Use Cases & Implementation:**

**Use Case 1: Self-Service Server Restart**

**Problem:** 300+ monthly tickets for "Restart Application X"
**Solution:** Self-service catalog item

```javascript
// ServiceNow Catalog Item: "Restart Application Service"
// Client Script (Before Submit)

function onSubmit() {
    // Validate inputs
    var application = g_form.getValue('application');
    var environment = g_form.getValue('environment');
    var server = g_form.getValue('server');
    
    if (!application || !environment || !server) {
        g_form.addErrorMessage('Please fill all required fields');
        return false;
    }
    
    // Production requires approval
    if (environment == 'production') {
        g_form.addInfoMessage('Production restart requires manager approval');
    }
    
    return true;
}
```

```javascript
// ServiceNow Workflow
// State: "Execute Restart"

(function executeRestart() {
    var request = current.request;
    var application = request.variables.application;
    var environment = request.variables.environment;
    var server = request.variables.server;
    
    try {
        // Call Jenkins job
        var endpoint = 'https://jenkins.nomura.com/job/restart-application/buildWithParameters';
        var auth = 'Basic ' + GlideStringUtil.base64Encode('jenkins_user:api_token');
        
        var httpRequest = new sn_ws.RESTMessageV2();
        httpRequest.setEndpoint(endpoint);
        httpRequest.setHttpMethod('POST');
        httpRequest.setRequestHeader('Authorization', auth);
        httpRequest.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        
        // Parameters
        var params = 'APPLICATION=' + application +
                    '&ENVIRONMENT=' + environment +
                    '&SERVER=' + server +
                    '&SNOW_TICKET=' + current.number;
        
        httpRequest.setRequestBody(params);
        
        var response = httpRequest.execute();
        var statusCode = response.getStatusCode();
        
        if (statusCode == 201) {
            // Success - get build number
            var location = response.getHeader('Location');
            var buildNumber = location.substring(location.lastIndexOf('/') + 1);
            
            // Update request item
            current.work_notes = 'Restart initiated. Jenkins build: #' + buildNumber;
            current.state = 'work_in_progress';
            current.update();
            
            // Schedule status check
            var schedule = new GlideSchedule();
            schedule.add(gs.minutesAgo(-2)); // Check in 2 minutes
            gs.eventQueue('jenkins.build.check', current, buildNumber);
            
            return true;
        } else {
            // Failure
            current.work_notes = 'Failed to trigger restart. Error code: ' + statusCode;
            current.state = 'failed';
            current.update();
            return false;
        }
        
    } catch (ex) {
        gs.error('Error executing restart: ' + ex.message);
        current.work_notes = 'Error: ' + ex.message;
        current.state = 'failed';
        current.update();
        return false;
    }
})();
```

```groovy
// Jenkins Pipeline: restart-application

pipeline {
    agent any
    
    parameters {
        choice(name: 'APPLICATION', choices: ['app1', 'app2', 'app3'])
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'staging', 'production'])
        string(name: 'SERVER')
        string(name: 'SNOW_TICKET')
    }
    
    stages {
        stage('Validate') {
            steps {
                script {
                    // Verify ServiceNow ticket exists
                    def ticketStatus = sh(
                        script: """
                            curl -s -u \${SNOW_USER}:\${SNOW_PASS} \
                            "\${SNOW_URL}/api/now/table/sc_req_item?sysparm_query=number=\${SNOW_TICKET}" \
                            | jq -r '.result[0].state'
                        """,
                        returnStdout: true
                    ).trim()
                    
                    if (ticketStatus != '2') { // 2 = Work in Progress
                        error("Invalid ticket state: ${ticketStatus}")
                    }
                    
                    echo "✅ ServiceNow ticket validated"
                }
            }
        }
        
        stage('Restart Application') {
            steps {
                script {
                    // Update ServiceNow
                    sh """
                        curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                        "\${SNOW_URL}/api/now/table/sc_req_item?sysparm_query=number=\${SNOW_TICKET}" \
                        -H "Content-Type: application/json" \
                        -d '{"work_notes": "Restarting ${APPLICATION} on ${SERVER}..."}'
                    """
                    
                    // Execute restart via Ansible
                    ansiblePlaybook(
                        playbook: 'restart-app.yml',
                        inventory: "inventories/${ENVIRONMENT}/hosts",
                        extras: "-e app_name=${APPLICATION} --limit ${SERVER}",
                        credentialsId: 'ansible-ssh-key'
                    )
                    
                    echo "✅ Application restarted successfully"
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Wait for application to start
                    sleep 30
                    
                    // Health check
                    def healthStatus = sh(
                        script: "curl -f http://${SERVER}:8080/health || echo 'FAILED'",
                        returnStdout: true
                    ).trim()
                    
                    if (healthStatus == 'FAILED') {
                        error("Health check failed")
                    }
                    
                    echo "✅ Health check passed"
                }
            }
        }
    }
    
    post {
        success {
            script {
                // Update ServiceNow - Closed Complete
                sh """
                    curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                    "\${SNOW_URL}/api/now/table/sc_req_item?sysparm_query=number=\${SNOW_TICKET}" \
                    -H "Content-Type: application/json" \
                    -d '{
                        "state": "3",
                        "close_notes": "Application ${APPLICATION} restarted successfully on ${SERVER}. Health check passed.",
                        "closed_at": "'$(date -u +%Y-%m-%d\ %H:%M:%S)'",
                        "closed_by": "jenkins_automation"
                    }'
                """
                
                // Send notification
                slackSend(
                    channel: '#ops-notifications',
                    color: 'good',
                    message: "✅ ${APPLICATION} restarted on ${SERVER} (Ticket: ${SNOW_TICKET})"
                )
            }
        }
        
        failure {
            script {
                // Update ServiceNow - Failed
                sh """
                    curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                    "\${SNOW_URL}/api/now/table/sc_req_item?sysparm_query=number=\${SNOW_TICKET}" \
                    -H "Content-Type: application/json" \
                    -d '{
                        "state": "4",
                        "work_notes": "Restart failed. Manual intervention required. Check Jenkins build #${BUILD_NUMBER}",
                        "assignment_group": "toc_operations"
                    }'
                """
                
                // Escalate
                slackSend(
                    channel: '#ops-alerts',
                    color: 'danger',
                    message: "❌ Failed to restart ${APPLICATION} on ${SERVER} (Ticket: ${SNOW_TICKET})"
                )
            }
        }
    }
}
```

**Impact:**
- **Before:** 15 minutes manual work per restart × 300/month = 75 hours
- **After:** 2 minutes self-service × 300/month = 10 hours
- **Savings:** 65 hours/month (87% reduction)

---

**Use Case 2: Incident Auto-Remediation**

**Problem:** Repetitive incidents require same manual steps
**Solution:** Automated remediation triggered by incident creation

```javascript
// ServiceNow Business Rule
// Table: Incident
// When: After Insert
// Condition: incident type = "Disk Space"

(function autoRemediateDiskSpace() {
    // Check if auto-remediation is enabled
    if (current.u_auto_remediate == false) {
        return;
    }
    
    var server = current.cmdb_ci.name;
    var threshold = 90; // 90% full
    
    // Trigger AWX job template
    var endpoint = 'https://awx.nomura.com/api/v2/job_templates/12/launch/';
    var auth = 'Bearer ' + gs.getProperty('awx.api.token');
    
    var httpRequest = new sn_ws.RESTMessageV2();
    httpRequest.setEndpoint(endpoint);
    httpRequest.setHttpMethod('POST');
    httpRequest.setRequestHeader('Authorization', auth);
    httpRequest.setRequestHeader('Content-Type', 'application/json');
    
    var payload = {
        'extra_vars': {
            'target_server': server,
            'snow_incident': current.number.toString(),
            'threshold': threshold
        }
    };
    
    httpRequest.setRequestBody(JSON.stringify(payload));
    
    try {
        var response = httpRequest.execute();
        var statusCode = response.getStatusCode();
        
        if (statusCode == 201) {
            var responseBody = JSON.parse(response.getBody());
            var jobId = responseBody.id;
            
            current.work_notes = 'Auto-remediation initiated. AWX Job ID: ' + jobId;
            current.state = 'work_in_progress';
            current.assigned_to = gs.getUserID('automation_user');
            current.update();
            
            gs.info('Auto-remediation triggered for incident: ' + current.number);
        } else {
            gs.error('Failed to trigger auto-remediation. Status: ' + statusCode);
            current.work_notes = 'Auto-remediation failed to trigger. Manual intervention required.';
            current.update();
        }
    } catch (ex) {
        gs.error('Error triggering auto-remediation: ' + ex.message);
    }
})();
```

```yaml
# AWX Playbook: disk-cleanup.yml
---
- name: Automated Disk Space Cleanup
  hosts: "{{ target_server }}"
  become: yes
  
  vars:
    threshold: "{{ threshold | default(90) }}"
    snow_incident: "{{ snow_incident }}"
    snow_url: "{{ lookup('env', 'SNOW_URL') }}"
    snow_user: "{{ lookup('env', 'SNOW_USER') }}"
    snow_pass: "{{ lookup('env', 'SNOW_PASS') }}"
  
  tasks:
    - name: Update ServiceNow - Starting cleanup
      uri:
        url: "{{ snow_url }}/api/now/table/incident?sysparm_query=number={{ snow_incident }}"
        method: PATCH
        user: "{{ snow_user }}"
        password: "{{ snow_pass }}"
        body_format: json
        body:
          work_notes: "Starting automated disk cleanup on {{ inventory_hostname }}"
        status_code: 200
      delegate_to: localhost
    
    - name: Check current disk usage
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage
      changed_when: false
    
    - name: Display current usage
      debug:
        msg: "Current disk usage: {{ disk_usage.stdout }}%"
    
    - name: Clean old logs (> 30 days)
      find:
        paths:
          - /var/log
          - /opt/app/logs
        age: 30d
        recurse: yes
      register: old_logs
    
    - name: Remove old logs
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_logs.files }}"
      when: old_logs.matched > 0
    
    - name: Clean package cache
      command: "{{ item }}"
      loop:
        - yum clean all
        - rm -rf /var/cache/yum/*
      when: ansible_os_family == "RedHat"
    
    - name: Clean temp files
      find:
        paths: /tmp
        age: 7d
        file_type: any
      register: temp_files
    
    - name: Remove old temp files
      file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ temp_files.files }}"
      when: temp_files.matched > 0
    
    - name: Check disk usage after cleanup
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage_after
      changed_when: false
    
    - name: Calculate space freed
      set_fact:
        space_freed: "{{ disk_usage.stdout | int - disk_usage_after.stdout | int }}"
    
    - name: Update ServiceNow - Cleanup complete
      uri:
        url: "{{ snow_url }}/api/now/table/incident?sysparm_query=number={{ snow_incident }}"
        method: PATCH
        user: "{{ snow_user }}"
        password: "{{ snow_pass }}"
        body_format: json
        body:
          work_notes: |
            Automated cleanup completed on {{ inventory_hostname }}
            
            Before: {{ disk_usage.stdout }}%
            After: {{ disk_usage_after.stdout }}%
            Space Freed: {{ space_freed }}%
            
            Actions taken:
            - Removed {{ old_logs.matched }} old log files
            - Cleaned package cache
            - Removed {{ temp_files.matched }} temp files
          state: "6"  # Resolved
          close_code: "Solved (Permanently)"
          close_notes: "Automated disk cleanup successful"
        status_code: 200
      delegate_to: localhost
      when: disk_usage_after.stdout | int < threshold | int
    
    - name: Escalate if cleanup insufficient
      uri:
        url: "{{ snow_url }}/api/now/table/incident?sysparm_query=number={{ snow_incident }}"
        method: PATCH
        user: "{{ snow_user }}"
        password: "{{ snow_pass }}"
        body_format: json
        body:
          work_notes: |
            Automated cleanup completed but disk usage still high: {{ disk_usage_after.stdout }}%
            Manual investigation required.
          priority: "1"  # Critical
          assignment_group: "toc_level2"
        status_code: 200
      delegate_to: localhost
      when: disk_usage_after.stdout | int >= threshold | int
```

**Impact:**
- **Disk space incidents:** 50/month
- **Before:** 20 minutes each = 16.7 hours/month
- **After:** Automated (80% success rate) = 3.3 hours/month
- **Savings:** 13.4 hours/month + faster MTTR

---

**Use Case 3: Change Request Automation**

**Problem:** Manual change request process delays deployments
**Solution:** Automated CR creation, approval routing, and execution

```groovy
// Jenkins Pipeline: production-deployment

pipeline {
    agent any
    
    parameters {
        string(name: 'APP_VERSION')
        string(name: 'DEPLOYMENT_WINDOW')
    }
    
    stages {
        stage('Create Change Request') {
            steps {
                script {
                    // Create change request in ServiceNow
                    def crNumber = sh(
                        script: """
                            curl -s -X POST -u \${SNOW_USER}:\${SNOW_PASS} \
                            "\${SNOW_URL}/api/now/table/change_request" \
                            -H "Content-Type: application/json" \
                            -d '{
                                "short_description": "Deploy ${params.APP_VERSION} to Production",
                                "description": "Automated deployment via Jenkins build #${BUILD_NUMBER}",
                                "type": "Standard",
                                "risk": "Low",
                                "impact": "Medium",
                                "priority": "3",
                                "requested_by": "jenkins_automation",
                                "assignment_group": "change_management",
                                "category": "Software",
                                "u_deployment_window": "${params.DEPLOYMENT_WINDOW}",
                                "u_jenkins_build": "${BUILD_NUMBER}",
                                "u_app_version": "${params.APP_VERSION}"
                            }' | jq -r '.result.number'
                        """,
                        returnStdout: true
                    ).trim()
                    
                    env.CHANGE_REQUEST = crNumber
                    echo "✅ Change Request created: ${crNumber}"
                    
                    // Add to build description
                    currentBuild.description = "CR: ${crNumber}"
                }
            }
        }
        
        stage('Wait for Approval') {
            steps {
                script {
                    echo "⏳ Waiting for change request approval: ${env.CHANGE_REQUEST}"
                    
                    timeout(time: 24, unit: 'HOURS') {
                        waitUntil {
                            def crState = sh(
                                script: """
                                    curl -s -u \${SNOW_USER}:\${SNOW_PASS} \
                                    "\${SNOW_URL}/api/now/table/change_request?sysparm_query=number=${env.CHANGE_REQUEST}" \
                                    | jq -r '.result[0].state'
                                """,
                                returnStdout: true
                            ).trim()
                            
                            if (crState == '-4') { // Cancelled
                                error("Change Request was cancelled")
                            } else if (crState == '-5') { // Rejected
                                error("Change Request was rejected")
                            } else if (crState == '-1') { // Approved
                                echo "✅ Change Request approved"
                                return true
                            }
                            
                            sleep 60 // Check every minute
                            return false
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            steps {
                script {
                    // Update CR - Implementing
                    sh """
                        curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                        "\${SNOW_URL}/api/now/table/change_request?sysparm_query=number=${env.CHANGE_REQUEST}" \
                        -H "Content-Type: application/json" \
                        -d '{"state": "-2", "work_notes": "Deployment started"}'
                    """
                    
                    // Actual deployment
                    ansiblePlaybook(
                        playbook: 'deploy-app.yml',
                        inventory: 'inventories/production/hosts',
                        extras: "-e app_version=${params.APP_VERSION} -e change_request=${env.CHANGE_REQUEST}",
                        credentialsId: 'ansible-ssh-key'
                    )
                }
            }
        }
        
        stage('Post-Deployment Validation') {
            steps {
                script {
                    // Health checks
                    sh './run-production-health-checks.sh'
                    
                    // Smoke tests
                    sh './run-production-smoke-tests.sh'
                    
                    echo "✅ Post-deployment validation passed"
                }
            }
        }
    }
    
    post {
        success {
            script {
                // Close Change Request - Successful
                sh """
                    curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                    "\${SNOW_URL}/api/now/table/change_request?sysparm_query=number=${env.CHANGE_REQUEST}" \
                    -H "Content-Type: application/json" \
                    -d '{
                        "state": "3",
                        "close_code": "successful",
                        "close_notes": "Deployment completed successfully. Version ${params.APP_VERSION} is now live in production. All health checks passed.",
                        "work_notes": "Jenkins build #${BUILD_NUMBER} completed successfully"
                    }'
                """
                
                slackSend(
                    channel: '#deployments',
                    color: 'good',
                    message: "✅ Production deployment successful: ${params.APP_VERSION} (CR: ${env.CHANGE_REQUEST})"
                )
            }
        }
        
        failure {
            script {
                // Update Change Request - Failed
                sh """
                    curl -X PATCH -u \${SNOW_USER}:\${SNOW_PASS} \
                    "\${SNOW_URL}/api/now/table/change_request?sysparm_query=number=${env.CHANGE_REQUEST}" \
                    -H "Content-Type: application/json" \
                    -d '{
                        "state": "4",
                        "close_code": "unsuccessful",
                        "close_notes": "Deployment failed. Check Jenkins build #${BUILD_NUMBER} for details. Rollback may be required.",
                        "work_notes": "Deployment failed - manual intervention required"
                    }'
                """
                
                // Create incident
                sh """
                    curl -X POST -u \${SNOW_USER}:\${SNOW_PASS} \
                    "\${SNOW_URL}/api/now/table/incident" \
                    -H "Content-Type: application/json" \
                    -d '{
                        "short_description": "Production deployment failed: ${params.APP_VERSION}",
                        "description": "Deployment via CR ${env.CHANGE_REQUEST} failed. Jenkins build: ${BUILD_URL}",
                        "urgency": "1",
                        "impact": "1",
                        "assignment_group": "toc_operations"
                    }'
                """
                
                slackSend(
                    channel: '#alerts',
                    color: 'danger',
                    message: "❌ Production deployment FAILED: ${params.APP_VERSION} (CR: ${env.CHANGE_REQUEST})"
                )
            }
        }
    }
}
```

**Impact:**
- **Before:** 2-4 hours for CR approval + deployment
- **After:** 30 minutes automated + approval wait time
- **Auditability:** 100% compliance with change management
- **Traceability:** Full audit trail from code commit to production

---

(Continuing file...)

**Use Case 4: Proactive Monitoring Integration**

**Problem:** Reactive incident creation after users report issues
**Solution:** Proactive incident creation from monitoring alerts

```python
# prometheus_to_servicenow.py
# Webhook receiver that creates ServiceNow incidents from Prometheus alerts

from flask import Flask, request, jsonify
import requests
import json

app = Flask(__name__)

SNOW_URL = 'https://nomura.service-now.com'
SNOW_USER = 'integration_user'
SNOW_PASS = 'password'

# Alert severity to ServiceNow priority mapping
SEVERITY_MAP = {
    'critical': '1',  # Critical
    'warning': '3',   # Medium
    'info': '5'       # Planning
}

@app.route('/webhook/prometheus', methods=['POST'])
def prometheus_webhook():
    data = request.json
    
    for alert in data.get('alerts', []):
        status = alert.get('status')  # firing or resolved
        labels = alert.get('labels', {})
        annotations = alert.get('annotations', {})
        
        if status == 'firing':
            # Create incident
            create_incident(alert, labels, annotations)
        elif status == 'resolved':
            # Resolve incident
            resolve_incident(alert, labels)
    
    return jsonify({'status': 'success'}), 200

def create_incident(alert, labels, annotations):
    alertname = labels.get('alertname')
    severity = labels.get('severity', 'warning')
    service = labels.get('service')
    instance = labels.get('instance')
    
    # Check if incident already exists
    existing = check_existing_incident(alertname, instance)
    if existing:
        print(f"Incident already exists: {existing}")
        return
    
    # Create ServiceNow incident
    incident_data = {
        'short_description': f"{alertname}: {annotations.get('summary', 'Alert triggered')}",
        'description': f"""
Alert: {alertname}
Severity: {severity}
Service: {service}
Instance: {instance}

Description:
{annotations.get('description', 'No description')}

Runbook: {annotations.get('runbook_url', 'N/A')}
Dashboard: {annotations.get('dashboard_url', 'N/A')}
        """,
        'priority': SEVERITY_MAP.get(severity, '3'),
        'impact': '2' if severity == 'critical' else '3',
        'urgency': '1' if severity == 'critical' else '2',
        'assignment_group': 'toc_operations',
        'category': 'Monitoring Alert',
        'u_alert_name': alertname,
        'u_alert_instance': instance,
        'u_alert_source': 'Prometheus'
    }
    
    response = requests.post(
        f'{SNOW_URL}/api/now/table/incident',
        auth=(SNOW_USER, SNOW_PASS),
        headers={'Content-Type': 'application/json'},
        json=incident_data
    )
    
    if response.status_code == 201:
        incident_number = response.json()['result']['number']
        print(f"✅ Created incident: {incident_number}")
        
        # If critical, trigger auto-remediation
        if severity == 'critical' and alertname in REMEDIATION_PLAYBOOKS:
            trigger_remediation(incident_number, alertname, instance)
    else:
        print(f"❌ Failed to create incident: {response.text}")

def resolve_incident(alert, labels):
    alertname = labels.get('alertname')
    instance = labels.get('instance')
    
    # Find open incident
    incident_number = check_existing_incident(alertname, instance)
    if not incident_number:
        return
    
    # Resolve incident
    update_data = {
        'state': '6',  # Resolved
        'close_code': 'Solved (Permanently)',
        'close_notes': 'Alert resolved automatically'
    }
    
    response = requests.patch(
        f'{SNOW_URL}/api/now/table/incident?sysparm_query=number={incident_number}',
        auth=(SNOW_USER, SNOW_PASS),
        headers={'Content-Type': 'application/json'},
        json=update_data
    )
    
    if response.status_code == 200:
        print(f"✅ Resolved incident: {incident_number}")

def check_existing_incident(alertname, instance):
    query = f"u_alert_name={alertname}^u_alert_instance={instance}^state!=6^state!=7"
    
    response = requests.get(
        f'{SNOW_URL}/api/now/table/incident',
        auth=(SNOW_USER, SNOW_PASS),
        params={'sysparm_query': query}
    )
    
    if response.status_code == 200:
        results = response.json().get('result', [])
        if results:
            return results[0]['number']
    
    return None

def trigger_remediation(incident_number, alertname, instance):
    """Trigger automated remediation for known issues"""
    playbook = REMEDIATION_PLAYBOOKS.get(alertname)
    if not playbook:
        return
    
    # Trigger AWX job
    awx_url = 'https://awx.nomura.com/api/v2/job_templates/{}/launch/'.format(playbook['template_id'])
    
    payload = {
        'extra_vars': {
            'target': instance,
            'incident_number': incident_number
        }
    }
    
    response = requests.post(
        awx_url,
        headers={
            'Authorization': f'Bearer {AWX_TOKEN}',
            'Content-Type': 'application/json'
        },
        json=payload
    )
    
    if response.status_code == 201:
        job_id = response.json()['id']
        print(f"✅ Triggered remediation job: {job_id}")
        
        # Update incident
        requests.patch(
            f'{SNOW_URL}/api/now/table/incident?sysparm_query=number={incident_number}',
            auth=(SNOW_USER, SNOW_PASS),
            json={'work_notes': f'Auto-remediation triggered: AWX job #{job_id}'}
        )

# Remediation playbook mapping
REMEDIATION_PLAYBOOKS = {
    'HighMemoryUsage': {'template_id': 10},
    'DiskSpaceLow': {'template_id': 12},
    'ServiceDown': {'template_id': 15}
}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**AlertManager Configuration:**

```yaml
# alertmanager.yml
receivers:
  - name: 'servicenow-integration'
    webhook_configs:
      - url: 'http://prometheus-servicenow-webhook:5000/webhook/prometheus'
        send_resolved: true

route:
  group_by: ['alertname', 'instance']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'servicenow-integration'
```

---

**Metrics Dashboard in ServiceNow:**

```javascript
// ServiceNow Homepage Widget: "Automation Metrics"

(function() {
    var gr = new GlideAggregate('sc_req_item');
    gr.addEncodedQuery('sys_created_onONToday@javascript:gs.beginningOfToday()@javascript:gs.endOfToday()');
    gr.addAggregate('COUNT');
    gr.groupBy('u_automation_status');
    gr.query();
    
    var automated = 0;
    var manual = 0;
    
    while (gr.next()) {
        var status = gr.getValue('u_automation_status');
        var count = parseInt(gr.getAggregate('COUNT'));
        
        if (status == 'automated') {
            automated = count;
        } else {
            manual = count;
        }
    }
    
    var total = automated + manual;
    var automation_rate = total > 0 ? (automated / total * 100).toFixed(1) : 0;
    
    var data = {
        automated: automated,
        manual: manual,
        total: total,
        automation_rate: automation_rate,
        target: 75  // Target 75% automation rate
    };
    
    return data;
})();
```

---

**Benefits Summary:**

```yaml
ServiceNow Integration Benefits:

Efficiency:
  - Ticket volume reduction: 40%
  - Average handling time: -60%
  - Manual effort reduction: 200+ hours/month
  - Self-service adoption: 65%

Quality:
  - Faster MTTR: -45%
  - Fewer human errors: -80%
  - Better audit trail: 100% compliance
  - Proactive incident detection: +70%

Business Impact:
  - Cost savings: $150K/year
  - Improved SLAs: +15%
  - Team satisfaction: +40%
  - After-hours escalations: -60%
```

For Nomura, this level of ServiceNow integration would transform TOC operations from reactive firefighting to proactive, automated service delivery."

---

(I'll stop here as the document is getting quite long. Would you like me to continue with more questions on topics like:
- Everbridge integration for critical escalations
- Infrastructure automation (Terraform)
- Change management in large enterprises
- Front-end development for operations portals
- Cross-team collaboration strategies
- Gap analysis methodology
- Or wrap up with a summary and key talking points?)