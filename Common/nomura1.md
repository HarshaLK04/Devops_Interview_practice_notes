# TOC Automation Engineer - Scenario-Based Interview Prep
## Nomura Role | Based on Resume: Harshal Kalamkar

---

## SECTION 1: OPERATIONAL SCENARIOS

### Scenario 1: High-Volume Ticket Reduction

**Interviewer:** "Your TOC team receives 2,000 tickets per month. 60% are repetitive tasks like server restarts, log collection, and password resets. How would you reduce this manual load?"

**Your Answer:**

"I'd approach this systematically using data analysis and prioritization:

**Week 1-2: Analysis**
- Pull 3 months of ServiceNow ticket data
- Categorize by type, frequency, resolution time
- Identify top 5 ticket types consuming most effort

**Example from Birlasoft:**
```
Top Ticket Types (from analysis):
1. Application restart: 350/month × 15min = 87.5 hours
2. Log collection: 280/month × 10min = 46.7 hours
3. Disk cleanup: 180/month × 20min = 60 hours
4. User access: 150/month × 25min = 62.5 hours
5. Health checks: 120/month × 5min = 10 hours
Total: 266.7 hours/month of manual work
```

**Week 3-4: Quick Wins Implementation**

**Priority 1: Application Restart (highest volume)**
- Create ServiceNow catalog item
- Backend: Jenkins job → Ansible playbook
- Safety: Pre-checks, health validation, rollback
- Result: 15min → 2min, 87% reduction

**Priority 2: Log Collection**
- AWX job template accessible via ServiceNow
- User selects timeframe, log type
- Automated collection, packaging, upload to shared location
- Result: 10min → 1min, 90% reduction

**Priority 3: Disk Cleanup**
- Monitoring alert triggers auto-remediation
- Cleans logs >30 days, temp files, package cache
- If successful, auto-closes ticket
- If fails, escalates to L2
- Result: 80% automated, 20% manual review

**Month 2-3: Measure & Optimize**
```
Results after 3 months:
- Ticket volume: 2,000 → 1,200 (40% reduction)
- Manual hours: 267 → 80 (70% reduction)
- Team capacity freed: 187 hours/month
- ROI: $14,000/month saved
```

**Key Success Factors:**
1. Data-driven prioritization
2. Start with highest ROI items
3. Maintain safety guardrails
4. Measure everything
5. Iterate based on feedback

At Birlasoft, this approach freed our team to focus on infrastructure improvements rather than routine tasks."

---

### Scenario 2: Production Incident During Deployment

**Interviewer:** "You're deploying a critical trading application to production at 2 AM. Halfway through the deployment, monitoring shows a spike in errors. What do you do?"

**Your Answer:**

"This is a time-critical decision. Here's my response:

**Immediate Actions (First 2 Minutes):**

1. **Stop deployment progression** - Don't deploy to remaining servers
2. **Assess impact:**
   - How many servers deployed? (e.g., 5 of 20)
   - What's the error rate? (e.g., 15% of requests failing)
   - Are customers impacted? (Check user-facing metrics)

3. **Check monitoring:**
   - Application logs: Any exceptions?
   - Infrastructure: CPU/Memory normal?
   - Database: Connection pool exhausted?
   - Network: Any latency spikes?

**Decision Point (Minute 2-5):**

**If errors are severe (>10% error rate):**
- **ROLLBACK immediately** on deployed servers
- Restore previous version using automated rollback
- Verify health after rollback

**If errors are minor (<5% and stable):**
- **Hold position** - don't rollback yet
- Investigate root cause while monitoring
- Keep rollback ready

**Example from Birlasoft:**
```
Scenario: Deployed microservice v2.1.0
Issue: 12% error rate after 5 servers deployed
Investigation (3 minutes):
  - Logs showed: "Connection timeout to Redis cache"
  - New version had different connection pool config
  - Redis was fine, config was wrong
  
Decision: ROLLBACK
Action taken:
  - Rolled back 5 servers to v2.0.9
  - Error rate dropped to 0.5% (normal)
  - Total downtime: 8 minutes
  
Post-mortem:
  - Config validation missed in pre-prod
  - Added: Config diff check in pipeline
  - Added: Smoke test for Redis connectivity
```

**Communication Protocol:**

**Immediate (during incident):**
- Slack #incidents: "Deployment halted due to errors, investigating"
- Page on-call manager if customer-impacting
- Update ServiceNow change request

**Post-resolution:**
- Slack update: Root cause + resolution
- Email to stakeholders
- Schedule post-mortem within 24 hours

**Rollback Procedure:**
```bash
# Automated rollback (30 seconds)
ansible-playbook rollback.yml -e "target_version=2.0.9" --limit "deployed_servers"

# Verify
curl https://api/health | jq .version
# Expected: "2.0.9"
```

**Prevention for Next Time:**
1. Deploy to canary group first (2-3 servers)
2. Monitor for 15 minutes before proceeding
3. Automated rollback if error threshold exceeded
4. Enhanced pre-deployment validation

**Key Principle:** In production, **availability > new features**. Always err on the side of rollback if in doubt. We can always deploy again after fixing the issue."

---

### Scenario 3: Conflicting SLO Requirements

**Interviewer:** "The business wants 99.99% availability (52 minutes downtime/year) but also wants daily deployments. Engineering says this is impossible. How do you handle this?"

**Your Answer:**

"This is a common tension between availability and velocity. I'd facilitate a data-driven discussion:

**Step 1: Translate Business Requirements**
```
99.99% availability = 4.38 minutes downtime/month
Daily deployments = 30 deployments/month
Per-deployment budget = 4.38 ÷ 30 = 8.76 seconds maximum downtime per deployment
```

**Reality check:** Zero-downtime deployment is the only option.

**Step 2: Present Options with Trade-offs**

**Option A: Maintain 99.99%, Reduce Deployment Frequency**
- Deploy weekly instead of daily
- Each deployment can have ~1 minute downtime
- Pros: Achievable with current architecture
- Cons: Slower feature delivery, larger batch sizes (more risk)

**Option B: Daily Deployments, Accept 99.95% SLO**
```
99.95% = 21.6 minutes downtime/month
Per-deployment budget = 21.6 ÷ 30 = 43 seconds per deployment
```
- Pros: Fast feature delivery
- Cons: Lower availability guarantee
- Impact: What's the business cost of extra 17 minutes downtime/month?

**Option C: Invest in Zero-Downtime Architecture** (Recommended)
- Implement blue-green or rolling deployments
- Load balancer-based traffic shifting
- Database migration compatibility (backward compatible changes)
- Timeline: 2-3 months investment
- Outcome: 99.99% availability + daily deployments

**Step 3: Recommend Based on Business Priority**

"At Birlasoft, we faced this exact scenario:

**Initial State:**
- SLA: 99.9% (43 min/month)
- Deployments: Weekly
- Deployment downtime: 5-10 minutes

**Business Pressure:**
- Trading desk wanted daily updates
- But couldn't tolerate more downtime

**Our Solution:**
1. **Month 1-2:** Implemented rolling deployments
   - Deploy to 20% of servers at a time
   - Health checks between batches
   - Automated rollback on failure

2. **Month 2-3:** Added blue-green capability
   - Two identical production environments
   - Switch traffic using load balancer
   - Instant rollback possible

3. **Month 3:** Enhanced deployment pipeline
   - Automated smoke tests
   - Gradual traffic shifting (5% → 25% → 100%)
   - Automatic rollback on error rate threshold

**Results:**
```
After implementation:
- Achieved: 99.97% availability (12 min downtime/month)
- Deployments: Daily (sometimes 2-3 times/day)
- Average deployment time: 8 minutes (zero downtime)
- Rollbacks: 2% of deployments (automatically triggered)
```

**Key Takeaway:** Rather than choosing between velocity and reliability, invest in engineering practices that enable both. Present the business case with timeline and resource requirements."

---

### Scenario 4: Cross-Team Automation Conflicts

**Interviewer:** "The Infrastructure team uses Terraform for provisioning, the Application team wants to use CloudFormation, and Security team mandates all changes through ServiceNow. How do you align these teams?"

**Your Answer:**

"This is about establishing common frameworks while respecting team autonomy:

**My Approach:**

**Step 1: Understand Each Team's Constraints**

**Infrastructure Team (Terraform preference):**
- Why: Existing expertise, multi-cloud support
- Valid concern: Retraining cost, tool switching overhead

**Application Team (CloudFormation preference):**
- Why: AWS-native, tightly integrated with their apps
- Valid concern: AWS-specific features, faster iteration

**Security Team (ServiceNow mandate):**
- Why: Audit trail, compliance requirements
- Valid concern: Governance, change control

**Step 2: Propose Common Framework**

**Standardization Layer:**
```
ServiceNow (Change Control - All teams)
    ↓
CI/CD Pipeline (Jenkins/GitLab - Common orchestration)
    ↓
Infrastructure as Code (Team choice: Terraform OR CloudFormation)
    ↓
Target Infrastructure (AWS, VMware, etc.)
```

**Key Principle:** Standardize interfaces, not implementations.

**Step 3: Define Integration Points**

**Example Implementation:**
```yaml
# Common pipeline template (Jenkins shared library)
- Stage: Change Request
  Action: Create/validate ServiceNow CR
  Mandatory: Yes
  
- Stage: Infrastructure Code Validation
  Action: terraform validate OR cfn-lint
  Flexible: Team choice of tool
  
- Stage: Security Scan
  Action: checkov/tfsec (works with both tools)
  Mandatory: Yes
  
- Stage: Deployment
  Action: terraform apply OR CloudFormation deploy
  Flexible: Team choice
  
- Stage: Compliance Check
  Action: Update ServiceNow, tag resources
  Mandatory: Yes
```

**Step 4: Real Example from Birlasoft**

**Problem We Faced:**
- Dev team: Kubernetes YAML manifests
- Ops team: Ansible playbooks
- Platform team: Terraform
- Security: "Everything must be audited"

**Solution We Implemented:**

**Common Gateway Pattern:**
```
All teams → GitLab CI/CD → Common validation → Team-specific tools → Common reporting
```

**Specific Rules:**
1. **All changes via Git** (single source of truth)
2. **All deployments via CI/CD** (no manual changes)
3. **All changes logged to ServiceNow** (automated)
4. **Teams choose IaC tool** (Terraform, Ansible, Helm, etc.)
5. **Common security scanning** (regardless of tool)

**Implementation:**
```python
# Common wrapper script (called by all teams)
def deploy_infrastructure(tool, environment, change_request):
    # 1. Validate ServiceNow CR
    if not validate_change_request(change_request):
        raise Exception("Invalid CR")
    
    # 2. Security scan (tool-agnostic)
    scan_results = run_security_scan(tool)
    if scan_results.critical_issues > 0:
        raise Exception("Security issues found")
    
    # 3. Execute deployment (tool-specific)
    if tool == "terraform":
        execute_terraform(environment)
    elif tool == "cloudformation":
        execute_cloudformation(environment)
    
    # 4. Update ServiceNow (common)
    update_servicenow(change_request, status="deployed")
    
    # 5. Tag resources (compliance)
    tag_resources(environment, change_request)
```

**Results:**
- Infrastructure team: Happy (kept Terraform)
- App team: Happy (kept CloudFormation)
- Security team: Happy (full audit trail, compliance)
- Reduced friction, increased adoption

**Key Success Factors:**
1. **Listen first** - Understand each team's constraints
2. **Standardize minimally** - Only what's necessary
3. **Automate the common parts** - Integration, validation, reporting
4. **Allow flexibility** - Where it doesn't impact governance
5. **Document clearly** - What's mandatory vs. optional

**For Nomura:** I'd start with a pilot project involving one team from each domain, prove the concept, then scale across the organization."

---

## SECTION 2: TECHNICAL SCENARIOS

### Scenario 5: Monitoring Alert Fatigue

**Interviewer:** "Your team gets 500 alerts per day. 90% are false positives or non-actionable. The team ignores most alerts. How do you fix this?"

**Your Answer:**

"Alert fatigue is a serious problem that erodes trust in monitoring. Here's my systematic approach:

**Phase 1: Immediate Triage (Week 1)**

**Classify all alerts:**
```
Critical + Actionable: Keep (5%)
  - Example: Database down, API error rate >10%
  - Action: Must page on-call

Warning + Actionable: Keep but don't page (10%)
  - Example: Disk space at 80%, CPU high
  - Action: Email, create ticket

Informational: Disable paging (20%)
  - Example: Deployment started, backup completed
  - Action: Log only, maybe Slack

False Positive: Fix or disable (65%)
  - Example: Threshold too sensitive, flapping metrics
  - Action: Tune or remove
```

**Phase 2: Alert Quality Improvement (Week 2-4)**

**Example: CPU High Alert (False Positive)**
```yaml
# Before (bad alert)
- alert: HighCPU
  expr: cpu_usage > 80%
  for: 1m
  # Problem: Fires during normal traffic spikes

# After (improved)
- alert: SustainedHighCPU
  expr: avg_over_time(cpu_usage[15m]) > 85%
  for: 15m
  # Better: Only fires if sustained high CPU for 15+ minutes
  
  annotations:
    summary: "Sustained high CPU on {{ $labels.instance }}"
    description: "CPU has been above 85% for 15+ minutes"
    runbook: "https://wiki/runbooks/high-cpu"
    dashboard: "https://grafana/d/cpu-dashboard"
```

**Phase 3: Alert Consolidation**

**Before:** Multiple alerts for related issues
```
- Database connection failed
- Application timeout errors
- Health check failing
- API response time high
```
All these fire when database is down = alert storm

**After:** Parent-child relationships
```yaml
# Primary alert
- alert: DatabaseDown
  expr: up{job="postgres"} == 0
  severity: critical

# Dependent alerts (suppressed when parent fires)
- alert: ApplicationErrors
  expr: error_rate > 0.05
  severity: warning
  # Only alert if database is UP
  expr: error_rate > 0.05 AND up{job="postgres"} == 1
```

**Phase 4: Actionable Alerts Only**

**Bad Alert (non-actionable):**
```
Alert: Disk usage high
What to do: ???
```

**Good Alert (actionable):**
```
Alert: Disk usage 85% on /var/log
Action: Run log cleanup: ansible-playbook cleanup-logs.yml
Runbook: https://wiki/disk-cleanup
Auto-remediation: Available via ServiceNow
```

**Real Example from Birlasoft:**

**Before Optimization:**
```
Daily alerts: 500
Pages to on-call: 120/day
Actionable: 6/day (5%)
Team response: Ignored 95% of alerts
Real incidents missed: 2 in 3 months
```

**After Optimization (3 months):**
```
Daily alerts: 45 (91% reduction)
Pages to on-call: 8/day
Actionable: 7/day (87%)
False positive rate: <10%
Real incidents missed: 0
Team trust in monitoring: Restored
```

**How We Did It:**

**Week 1: Quick Wins**
- Disabled 200 alerts that hadn't fired in 30 days
- Increased threshold on 100 noisy alerts
- Consolidated 50 duplicate alerts
Result: 500 → 150 alerts/day

**Week 2-4: Systematic Review**
- Each alert reviewed by team
- Question: "If this alert fires at 2 AM, would I take action?"
- If no: Disable or convert to warning
- If yes: Ensure runbook exists

**Week 5-8: SLO-Based Alerting**
```yaml
# Instead of: Alert on every 5xx error
# Do: Alert on SLO burn rate

- alert: ErrorBudgetBurnRateFast
  expr: |
    error_rate > (14.4 * slo_target_error_rate)
  for: 5m
  # Only fires if burning through monthly error budget in <2 days
  
  annotations:
    summary: "Fast error budget burn - investigate immediately"
    impact: "At this rate, monthly error budget exhausted in 2 days"
```

**Key Principles:**

1. **Every alert must be actionable** - If you can't act on it, it shouldn't page
2. **Context matters** - Include runbook, dashboard, suggested action
3. **Tune thresholds based on reality** - Not arbitrary numbers
4. **Reduce noise** - Better to miss 1 edge case than ignore everything
5. **Alert on symptoms, not causes** - Alert on "API slow" not "CPU high"

**For Nomura:** I'd start with a 2-week sprint to audit top 50 noisiest alerts, fix those, then systematically work through the rest."

---

### Scenario 6: Multi-Region Deployment Failure

**Interviewer:** "You're deploying to 3 regions (US, EU, APAC) simultaneously. EU deployment fails halfway through. What's your recovery strategy?"

**Your Answer:**

"Multi-region deployments require careful orchestration and blast radius control:

**My Deployment Strategy:**

**Approach: Staged Regional Rollout** (not simultaneous)
```
1. Deploy to APAC first (smallest user base, low-traffic hours)
   - Monitor for 2 hours
   
2. If APAC successful, deploy to EU
   - Monitor for 2 hours
   
3. If EU successful, deploy to US (largest user base)
   
4. Total deployment window: 6-8 hours (safer than 1 hour blast)
```

**But if EU already failed halfway:**

**Immediate Response (First 5 Minutes):**

**1. Halt Further Deployments**
- STOP US deployment immediately
- Don't make the problem bigger

**2. Assess EU Impact**
```
Questions:
- How many EU servers deployed? (e.g., 15 of 30)
- What percentage of EU traffic affected? (e.g., 50%)
- Is APAC still healthy? (Check)
- Error rate in EU? (e.g., 8%)
```

**3. Decision Matrix:**

**If APAC is affected too:**
- **Global issue** with new version
- **Action:** Rollback APAC and EU deployed servers
- **Reason:** Version has fundamental problem

**If only EU affected:**
- **Regional or deployment issue**
- **Action:** Investigate while holding position
- **Possible causes:** 
  - EU-specific config error
  - Database migration incompatibility (EU DB version different)
  - Network/firewall rules
  - EU-specific integration (payment gateway, etc.)

**Recovery Procedure:**

**Scenario: EU deployment failed, APAC healthy**

**Step 1: Contain (5 minutes)**
```bash
# Mark unhealthy EU servers out of load balancer rotation
# Traffic shifts to healthy servers (50% capacity)

# If EU load balancer supports:
aws elbv2 deregister-targets --target-group-arn $EU_TG \
  --targets Id=i-deployed-server-1 Id=i-deployed-server-2
```

**Step 2: Investigate (10 minutes)**
```bash
# Check EU deployed servers
- Application logs: Any exceptions?
- Database connectivity: OK?
- Config diff: Any EU-specific settings?
- Integration tests: EU payment gateway failing?

# Compare APAC (working) vs EU (failing)
diff apac-config.yml eu-config.yml
```

**Step 3: Fix or Rollback (15 minutes)**

**If quick fix available (config error):**
```bash
# Fix config, redeploy just to failed servers
ansible-playbook deploy.yml \
  --limit eu-deployed-servers \
  -e fix_config=true
```

**If no quick fix (code issue):**
```bash
# Rollback EU deployed servers
ansible-playbook rollback.yml \
  --limit eu-deployed-servers \
  -e target_version=previous

# Verify health
curl https://eu-api/health | jq .version
```

**Step 4: Post-Incident**
```
- All regions on same version? Or EU on old version?
- If EU rolled back: Schedule fix + redeploy
- If fixed: Continue to US deployment (after validation)
```

**Real Example from Infosys:**

**Scenario:**
- Deploying payment microservice to 3 regions
- APAC successful, EU failed at 60% deployed
- Error: "Payment gateway timeout"

**Investigation:**
- EU uses different payment provider (Adyen vs Stripe)
- New version had hardcoded Stripe config
- APAC worked because it uses Stripe

**Resolution:**
```
Immediate:
  - Rolled back EU deployed servers (10 minutes)
  - EU back to previous version, stable
  
Fix:
  - Made payment provider configurable
  - Added EU-specific integration tests
  - Redeployed next day, successful
  
Prevention:
  - Added: Region-specific smoke tests in pipeline
  - Added: Config validation for each region
  - Changed: Deploy to smallest region first, validate, then scale
```

**Better Deployment Pattern:**

```yaml
# Progressive Regional Deployment
stages:
  - stage: Deploy APAC
    validation:
      - smoke_tests
      - health_checks
      - monitor: 2 hours
    rollback_on_failure: auto
  
  - stage: Deploy EU (after APAC validation)
    canary: 10%  # Deploy to 10% of EU servers first
    validation:
      - smoke_tests
      - region_specific_tests  # EU payment gateway
      - monitor: 30 minutes
    progressive_rollout:
      - 10% → 50% → 100%
      - pause: 30 minutes between stages
    rollback_on_failure: auto
  
  - stage: Deploy US (after EU validation)
    # Same progressive approach
```

**Key Takeaways:**

1. **Never deploy to all regions simultaneously** - Staged rollout limits blast radius
2. **Each region needs specific validation** - Not all integrations are global
3. **Monitor between regions** - Use APAC as canary for EU/US
4. **Have region-specific rollback** - Independent rollback capability
5. **Time zones matter** - Deploy during low-traffic hours for each region

**For Nomura:** Given financial services criticality, I'd recommend even more conservative approach - deploy to one region per day, with full 24-hour monitoring before next region."

---

### Scenario 7: Conflicting Priorities During Incident

**Interviewer:** "You're working on a P1 production incident. Your manager asks you to attend a vendor demo meeting in 15 minutes. The business wants a status update. What do you do?"

**Your Answer:**

"Clear prioritization and communication are critical during incidents:

**My Response:**

**Immediate (30 seconds):**

**To Manager (Slack/Teams):**
```
"I'm leading response to P1 incident (customer-impacting database issue). 
Cannot attend vendor demo.
Can you attend or should we reschedule?
ETA for incident resolution: 45 minutes"
```

**To Business/Stakeholders (via incident channel):**
```
"Status Update - 10:15 AM
Incident: Database performance degradation
Impact: 30% of API requests timing out
Team: 3 engineers investigating
Current action: Rolling back recent DB config change
ETA: 30-45 minutes
Next update: 10:30 AM (15 minutes)
```

**Priority Order (Clear in my mind):**
```
1. P1 Incident Resolution (customer impact)
2. Communication to stakeholders
3. Documentation for handoff
4. Vendor demo (can be rescheduled)
```

**Handling the Incident:**

**If I'm the only expert:**
- Stay on incident, delegate demo
- Brief update every 15 minutes
- Resolve incident first

**If others can handle:**
- Delegate to senior engineer
- Stay available on Slack
- Quick handoff: "Here's what we've tried, next step is X"
- Attend demo if absolutely critical
- Return to incident after

**Real Example from Birlasoft:**

**Scenario:**
- P1: Trading API down during market hours
- Manager: "Critical steering committee meeting in 10 minutes"
- Business: "When will trading resume?"

**My Actions:**

**1. Assessed situation (1 minute):**
```
- Incident severity: P1 (trading halted)
- Team capacity: 2 engineers (myself + junior)
- Junior capability: Can execute, needs guidance
- Meeting importance: High (steering committee)
- Meeting urgency: Can I be 30 min late?
```

**2. Delegated to junior (2 minutes):**
```
"Here's the situation:
- Trading API down due to database lock
- We've identified the problematic query
- Next step: Kill the query session, restart API
- Command: [showed exact command]
- Expected result: API comes back in 5 minutes
- If issue: Call me immediately, I'll come back
- I'll be in meeting but phone on loud"
```

**3. Communicated (2 minutes):**

**To Manager:**
```
"P1 trading incident. Delegated to Rahul with clear instructions.
Can attend meeting, but may need to step out if escalates.
Phone on loud."
```

**To Business (Incident Slack channel):**
```
"10:05 AM Update:
Root cause identified: Long-running database query
Fix in progress: Terminating query, restarting API
Expected resolution: 10:10 AM
Monitoring: Rahul (backup: myself)
Next update: 10:15 AM"
```

**4. Attended meeting (with phone):**
- Updated steering committee on incident
- Gave 5-minute status on quarterly automation goals
- Left meeting after 15 minutes when Rahul confirmed resolution

**Outcome:**
- Incident resolved: 10:12 AM (7 minutes)
- Meeting attended: Provided update on automation initiatives
- Business satisfied: Quick communication + resolution
- Junior engineer: Learned incident response

**Key Principles:**

**1. Prioritize by Impact:**
```
P1 customer-impacting > P2 internal > Meetings > Email > Demos
```

**2. Communicate Clearly:**
- Set expectations (ETA, next update time)
- Don't go dark - regular updates even if "still investigating"
- Be honest if you don't know

**3. Delegate When Possible:**
- Empower team members
- Give clear instructions
- Stay available as backup

**4. Know When to Push Back:**
- "I understand the meeting is important, but we have customers unable to trade. 
   I need to resolve this first. Can I join in 30 minutes?"
- Most managers will understand and support

**5. Document as You Go:**
- Incident timeline in shared doc
- Others can pick up if you need to step away
- Helps post-mortem later

**Bad Responses (Don't Do This):**

❌ Drop incident to attend meeting without handoff
❌ Ignore manager's meeting request without response
❌ Work on incident silently without status updates
❌ Try to do both (attend meeting while debugging)

**What I'd Tell Nomura Interviewers:**

"In financial services, incident response is paramount. I'd always prioritize customer-impacting issues over internal meetings. However, I'd communicate clearly so stakeholders know the situation and expected timeline. If the meeting is truly critical and can't be missed, I'd ensure proper handoff to another engineer with clear instructions and remain available as backup. The key is clear communication and knowing when to delegate vs. when to stay hands-on."

---

## SECTION 3: COLLABORATION & LEADERSHIP SCENARIOS

### Scenario 8: Resistance to Automation

**Interviewer:** "The operations team is resistant to automation. They say 'We've always done it manually and it works fine. Automation will make us obsolete.' How do you handle this?"

**Your Answer:**

"This is about change management and addressing fears:

**Understanding the Resistance:**

**Common fears I've encountered:**
1. **Job security:** "Will automation replace me?"
2. **Loss of control:** "What if automation makes mistakes?"
3. **Complexity:** "I don't know Python/Ansible, can't maintain it"
4. **Past failures:** "We tried automation before, it didn't work"
5. **Expertise devaluation:** "My value is in knowing how to do X manually"

**My Approach:**

**Phase 1: Listen & Empathize (Week 1)**

**Individual conversations:**
```
Me: "I heard you have concerns about the automation initiative. 
     Tell me more about what worries you."

Common responses:
- "I've been doing this for 10 years. Automation will make my skills worthless."
- "What if the automation breaks? Who will fix it?"
- "Management will use this to reduce headcount."

My response:
- "Your expertise is exactly what we need to build good automation.
   You know all the edge cases that can go wrong.
   Without your input, automation will fail."
```

**Phase 2: Reframe Automation (Week 1-2)**

**Key messages:**

**From:** "Automation replaces you"
**To:** "Automation removes toil, elevates your work"

```
Manual work today:
- 60% repetitive tasks (restarts, log collection, password resets)
- 30% firefighting incidents
- 10% proactive improvements

With automation:
- 10% handling automation exceptions
- 30% improving automation
- 60% strategic work (architecture, optimization, innovation)
```

**Real example from Birlasoft:**

"I had an ops engineer, Suresh, who was skeptical:

**His concern:**
'I restart applications 20 times a day. If you automate this, what's my job?'

**My response:**
'You spend 5 hours/day restarting apps. That's 100 hours/month.
What if instead, you:
- Build self-healing systems so apps don't need restarts
- Analyze why apps crash and fix root causes
- Design better deployment strategies
- Mentor junior engineers

Which would you prefer: Being a restart button, or being the architect 
who designs systems that don't need restarting?'

**His reaction:**
Initially skeptical, but intrigued.

**What I did:**
1. Involved him in designing the automation
2. He documented all the edge cases (his expertise)
3. He became the 'automation owner' for his domain
4. 6 months later, he was leading a team building self-healing infrastructure

**His quote:**
'I used to dread Monday mornings because of all the manual work. 
Now I actually enjoy coming to work because I'm solving interesting problems.'"

**Phase 3: Show Quick Wins (Month 1)**

**Start with pain points they hate:**
```
Survey results from Birlasoft ops team:
"What manual task do you hate most?"

1. After-hours emergency restarts (40% voted)
2. Collecting logs from 50+ servers (25%)
3. Password reset requests (20%)

Strategy: Automate #1 first (highest pain)
Result: Team saw immediate quality of life improvement
Response: "This is actually helpful"
```

**Phase 4: Involve Team in Building (Month 1-3)**

**Don't build automation *for* them, build it *with* them:**

```
Bad approach:
- DevOps team builds automation in isolation
- Presents completed solution to ops team
- "Here, use this"
- Result: Not adopted, lacks edge cases

Good approach:
- Ops team identifies automation candidates
- Ops provides requirements and edge cases
- DevOps provides tooling/framework
- Build together in sprints
- Ops owns the automation
- Result: Adopted, comprehensive, maintained
```

**Example workshop format:**
```
Week 1: Automation Workshop
- Ops team: Documents manual process
- DevOps: Shows automation possibilities
- Together: Design automation approach

Week 2-4: Build Together
- Ops: Write requirements, test cases
- DevOps: Build automation framework
- Pair programming sessions

Week 5: Ops Team Owns It
- Ops deploys and maintains
- DevOps supports as needed
```

**Phase 5: Address Job Security Fears (Ongoing)**

**Manager's message (critical):**
```
"Automation is not about reducing headcount.
It's about:
1. Reducing toil
2. Improving service quality
3. Freeing you for higher-value work
4. Making on-call less painful

No one will lose their job due to automation.
In fact, automation skills make you more valuable."
```

**Reskilling program:**
```
Offered at Birlasoft:
- Ansible training (2 weeks)
- Python fundamentals (4 weeks)
- Git & CI/CD (1 week)
- Monitoring & Observability (1 week)

Result:
- 80% of ops team completed training
- 5 team members became automation SMEs
- 2 promoted to senior roles (automation focus)
```

**Metrics to Track:**

**Before automation:**
```
Team satisfaction: 5/10
After-hours pages: 150/month
Toil percentage: 60%
Incident resolution time: 45 minutes
```

**After automation (6 months):**
```
Team satisfaction: 8/10
After-hours pages: 40/month (73% reduction)
Toil percentage: 15%
Incident resolution time: 12 minutes
Team quote: "I don't dread on-call anymore"
```

**Key Success Factors:**

1. **Empathy first** - Understand their fears
2. **Involve, don't impose** - Build with them, not for them
3. **Show value quickly** - Automate their biggest pain points
4. **Reskill, don't replace** - Invest in training
5. **Leadership support** - Manager must reinforce job security
6. **Celebrate success** - Public recognition for automation contributions

**For Nomura:**
'Given the scale of operations at Nomura, I'd start with a pilot team (10-15 people), prove the value, showcase the career growth opportunities, and let success stories spread organically. The best evangelists for automation are ops engineers who've experienced the benefits firsthand.'"

---

### Scenario 9: Limited Budget, Unlimited Asks

**Interviewer:** "You have 3 team members and $50K budget for tools. Business wants: better monitoring, faster deployments, reduced incidents, and self-service portal. How do you prioritize?"

**Your Answer:**

"Classic resource constraint scenario. I'd use data to prioritize ROI:

**Step 1: Quantify Current State (Week 1)**

```
Current Metrics:
- Incident MTTR: 45 minutes
- Deployments: 2/week (manual, 2 hours each)
- Manual tickets: 800/month (200 hours team effort)
- Monitoring coverage: 40%
- After-hours escalations: 80/month

Team capacity: 3 people × 160 hours = 480 hours/month
Current allocation:
  - Manual tickets: 200 hours (42%)
  - Deployment support: 16 hours (3%)
  - Incident response: 80 hours (17%)
  - Project work: 184 hours (38%)
```

**Step 2: Business Impact Analysis**

**Calculate impact of each initiative:**

**Option 1: Better Monitoring**
```
Problem: 60% of incidents discovered by users (not monitoring)
Impact:
  - Earlier detection = -30% MTTR (save 240 hours/year)
  - Fewer escalations = -40 pages/month
  - Better sleep for team = Improved retention
Cost: $15K (Prometheus + Grafana) + 120 hours setup
ROI: High (improved incident response)
```

**Option 2: Deployment Automation**
```
Problem: Manual deployments (2 hours × 2/week = 4 hours/week)
Impact:
  - Automated: 15 min × 2/week = 0.5 hours/week
  - Save: 182 hours/year
  - Enable: Daily deployments (faster feature delivery)
Cost: $5K (Jenkins plugins) + 80 hours setup
ROI: Medium (saves time + enables velocity)
```

**Option 3: Incident Reduction**
```
Problem: Same incidents repeating (20% are duplicates)
Impact:
  - Auto-remediation for top 5 incident types
  - Reduce incident volume by 30%
  - Save: 624 hours/year
Cost: $10K (Ansible Tower) + 160 hours build
ROI: Highest (massive time savings)
```

**Option 4: Self-Service Portal**
```
Problem: 800 tickets/month for routine requests
Impact:
  - Self-service for 50% of tickets
  - Save: 1,200 hours/year
  - Improve: User satisfaction
Cost: $20K (Portal solution) + 240 hours build
ROI: Highest (huge time savings)
```

**Step 3: Prioritization Matrix**

```
Initiative          | Impact | Cost  | Time | ROI   | Priority
--------------------|--------|-------|------|-------|----------
Self-Service Portal | High   | $20K  | 240h | 5.0x  | 1
Incident Reduction  | High   | $10K  | 160h | 6.2x  | 2
Better Monitoring   | High   | $15K  | 120h | 2.0x  | 3
Deploy Automation   | Medium | $5K   | 80h  | 2.3x  | 4

Budget remaining: $50K
Team capacity: 480 hours/month
```

**Step 4: My Recommendation**

**Phase 1 (Month 1-3): Foundation**

**Priority 1: Incident Reduction ($10K)**
```
Why first: Highest ROI, frees team capacity
Tools: Ansible Tower/AWX ($10K)
Time: 160 hours

Auto-remediation for:
1. Application restarts (250/month)
2. Log collection (150/month)
3. Disk cleanup (100/month)

Expected outcome:
  - Reduce tickets: 500/month → 200/month
  - Free up: 100 hours/month
  - Cumulative capacity: +1,200 hours/year
```

**Priority 2: Better Monitoring ($15K)**
```
Why: Enable proactive operations
Tools: Prometheus + Grafana ($15K for enterprise support)
Time: 120 hours

Deploy:
  - Application metrics (RED: Rate, Errors, Duration)
  - Infrastructure metrics
  - SLO dashboards
  - Alerting (on SLO violations, not thresholds)

Expected outcome:
  - Detect 80% of issues before users
  - Reduce MTTR by 30%
  - Save: 240 hours/year
```

**Budget used: $25K
Budget remaining: $25K (save for Phase 2)**

**Phase 2 (Month 4-6): Optimization**

**Priority 3: Self-Service Portal ($20K)**
```
Why: Biggest long-term time saver
Tools: ServiceNow catalog expansion ($20K licensing)
Time: 240 hours (spread over 3 months)

Build self-service for:
  - Server restarts
  - Log access
  - Account provisioning
  - Certificate renewals

Expected outcome:
  - Reduce tickets: 200/month → 80/month
  - Save: 1,200 hours/year
  - Improve: User satisfaction scores
```

**Priority 4: Deployment Automation ($5K)**
```
Why: Lower ROI, but enables velocity
Tools: Jenkins plugins + scripting ($5K)
Time: 80 hours

Automate:
  - Build → Test → Deploy pipeline
  - Automated rollback
  - Deployment notifications

Expected outcome:
  - Deployments: 2/week → daily
  - Time per deployment: 2 hours → 15 min
  - Save: 182 hours/year
  - Enable: Faster feature delivery
```

**Budget used: $50K
Team capacity: Fully allocated but ROI positive**

**Step 5: Communicate Trade-offs**

**To Business:**
```
"Given budget constraints, here's what we can deliver:

Quarter 1:
  ✅ Incident reduction (auto-remediation)
  ✅ Improved monitoring & alerting
  📊 Result: 60% reduction in manual tickets, proactive issue detection

Quarter 2:
  ✅ Self-service portal for common requests
  ✅ Automated deployments
  📊 Result: 70% reduction in manual tickets, daily deployments

What we're NOT doing this year:
  ❌ Advanced analytics platform (would need $100K)
  ❌ Full AIOps implementation (would need 2 more team members)

We can revisit these next fiscal year based on results."
```

**Real Example from Birlasoft:**

**Similar scenario:**
- Team: 4 people
- Budget: $75K
- Asks: Everything

**Our approach:**
```
Quarter 1: AWX + Prometheus/Grafana ($30K)
  Result: 40% ticket reduction, better visibility

Quarter 2: ServiceNow expansion ($25K)
  Result: Self-service adoption 55%

Quarter 3: Used saved capacity (not budget) for:
  - Custom automation scripts
  - Integration development
  - Documentation

Quarter 4: Saved budget ($20K) for:
  - Training & certifications
  - Conference attendance
  - Innovation time
```

**Outcome after 1 year:**
```
Tickets: 1,000/month → 250/month (75% reduction)
Deployments: 2/week → 8/week (4x increase)
Incidents: 45 min MTTR → 15 min MTTR (67% improvement)
Team morale: Significantly improved
Budget used: $55K of $75K (under budget!)

Business response: "This exceeded expectations.
                    Here's $150K for next year."
```

**Key Takeaways:**

1. **Data over opinions** - Quantify impact of each option
2. **ROI focus** - Highest return initiatives first
3. **Build capacity first** - Automation frees time for more automation
4. **Under-promise, over-deliver** - Better to exceed conservative goals
5. **Save budget** - Don't spend just because you have it
6. **Communicate clearly** - Set realistic expectations

**For Nomura:**
'I'd start with a detailed current-state analysis, quantify the impact of each initiative, and propose a phased approach that delivers measurable value each quarter. I'd also look for opportunities to use open-source tools where appropriate to maximize budget efficiency.'"

---

### Scenario 10: Knowledge Transfer & Documentation

**Interviewer:** "Your team's senior engineer (who knows all the legacy systems) is leaving in 2 weeks. How do you capture that knowledge?"

**Your Answer:**

"Two weeks isn't enough, but we can capture critical knowledge systematically:

**Emergency Knowledge Transfer Plan:**

**Week 1: Inventory & Priority**

**Day 1-2: Knowledge Mapping**
```
Meet with departing engineer:
"Let's map out what you know that's NOT documented"

Categories:
1. Critical systems (production impact if knowledge lost)
2. Tribal knowledge (known only to you)
3. Workarounds and gotchas (not in official docs)
4. Escalation paths and contacts
5. Undocumented procedures

Output: Prioritized list of knowledge gaps
```

**Example from Birlasoft:**
```
When Amit left, we identified:

Critical knowledge:
1. Legacy trading system restart procedure (specific sequence)
2. Database failover process (manual steps)
3. Vendor escalation contacts (not in ServiceNow)
4. Configuration quirks (why settings are set this way)
5. Historical incident patterns (recurring issues)

Medium priority:
6-10. Various system integrations, backup procedures, etc.
```

**Day 3-5: Rapid Documentation Sessions**

**Format: 90-minute focused sessions**
```
Session 1: Legacy Trading System
- Record video: Walkthrough of restart procedure
- Document: Step-by-step runbook
- Capture: All edge cases and error scenarios
- Demo: Show someone else, they try, document issues

Session 2: Database Failover
- Live demonstration (in test environment)
- Screen recording
- Document decision points ("How do you know when to failover?")
- Create checklist

Session 3: Vendor Relationships
- List all vendor contacts
- Document escalation procedures
- Share email threads with critical context
- Introduce team to vendor contacts
```

**Week 2: Practice & Validation**

**Day 6-8: Hands-on Practice**
```
Assign scenarios to team members:
"Pretend the trading system is down. Walk through restart procedure."

Departing engineer observes:
- What did they miss?
- What wasn't clear in documentation?
- Update docs in real-time

Repeat for all critical procedures
```

**Day 9-10: Documentation Finalization**
```
Create knowledge repository:

/knowledge-base/
  /runbooks/
    - trading-system-restart.md
    - database-failover.md
    - vendor-escalations.md
  /videos/
    - trading-restart-demo.mp4
    - db-failover-walkthrough.mp4
  /diagrams/
    - system-architecture.png
    - network-topology.png
  /contacts/
    - vendor-contacts.xlsx
    - escalation-matrix.xlsx
```

**Documentation Template I Use:**

```markdown
# System: Legacy Trading Platform
## Last Updated: 2024-02-16
## Owner: Harshal Kalamkar (after Amit's departure)

## Quick Links
- [Runbook](#restart-procedure)
- [Troubleshooting](#common-issues)
- [Escalation](#when-to-escalate)

## System Overview
- Purpose: Processes equity trades for APAC region
- Criticality: HIGH (revenue impact)
- Uptime SLA: 99.9%
- Peak hours: 9 AM - 5 PM IST

## Architecture
[Diagram showing components, dependencies]

## Restart Procedure
### When to restart:
- Application becomes unresponsive
- Error rate > 5% for 10+ minutes
- Memory usage > 90% and not recovering

### Pre-checks:
- [ ] Check if market is open (don't restart during trading hours unless critical)
- [ ] Notify #trading-ops Slack channel
- [ ] Create ServiceNow incident

### Steps:
1. SSH to primary server: `ssh trade-prod-01.nomura.com`
2. Check application status: `systemctl status trading-app`
3. Stop application: `systemctl stop trading-app`
4. IMPORTANT: Wait 60 seconds (allows in-flight transactions to complete)
5. Verify no processes running: `ps aux | grep trading`
6. Start application: `systemctl start trading-app`
7. Monitor startup: `tail -f /var/log/trading/startup.log`
8. Wait for: "Application started successfully" (usually 2-3 minutes)
9. Health check: `curl http://localhost:8080/health`
10. Smoke test: Submit test trade via UI

### What can go wrong:
**Issue:** App won't stop cleanly
**Solution:** Force kill: `kill -9 $(cat /var/run/trading.pid)`
**Warning:** This may cause transaction loss. Only use if absolutely necessary.

**Issue:** App won't start - "Port 8080 already in use"
**Solution:** 
```bash
# Find process using port 8080
lsof -i :8080
# Kill it
kill -9 <PID>
# Restart app
```

### Escalation:
If restart doesn't work after 2 attempts:
1. Page on-call: Use PagerDuty "Trading Platform" escalation
2. Call vendor: Acme Corp, 24/7 hotline: +1-800-555-1234
   - Reference number: NOMURA-12345
   - Contact: John Smith (john.smith@acmecorp.com)

## Historical Context
### Why is restart sequence so specific?
- 60-second wait: Discovered after 2019 incident where restart caused data loss
- Port conflict: Common issue due to zombie processes
- Health check critical: Once started without green status, caused 2-hour outage

## Related Systems
- Database: Oracle RAC cluster (db-prod-01, db-prod-02)
- Message queue: IBM MQ
- Reporting: PowerBI dashboard

## Automation Status
- Automated: Health checks, basic monitoring
- Not automated: Restart (too risky, requires human judgment)
- Future: Considering auto-restart for non-market hours

## Questions?
Contact: Harshal Kalamkar (harshal.kalamkar@nomura.com)
Backup: Sarah Johnson (sarah.johnson@nomura.com)
```

**Beyond Documentation: Other Strategies**

**1. Shadowing Sessions**
```
Days 1-10: Team members shadow departing engineer
- Follow along during routine tasks
- Observe decision-making process
- Ask questions in real-time
```

**2. Recorded Sessions**
```
Screen recordings of:
- Incident response
- Troubleshooting sessions
- Configuration changes
- Deployment procedures

Store in shared repository
```

**3. Create Decision Trees**
```
For complex troubleshooting:

"Trading system slow" →
  Is CPU > 80%? 
    Yes → Check for runaway processes
    No → Is memory > 80%?
      Yes → Memory leak, restart needed
      No → Is database slow?
        Yes → Check DB locks
        No → Check network latency
```

**4. Pair with Vendor**
```
If critical knowledge is vendor-specific:
- Arrange knowledge transfer session with vendor
- Get vendor documentation
- Establish direct contacts
- Consider retainer for post-departure support
```

**Long-term Prevention:**

**Culture change:**
```
Rule: "If you're the only person who knows how to do something,
       that's a P1 incident waiting to happen"

Requirements:
- All critical procedures documented
- At least 2 people trained on each system
- Quarterly knowledge validation sessions
- Documentation updates mandatory for any changes
```

**Real Example from Birlasoft:**

**When Rajesh left (payment system expert):**

**What we did (his last 2 weeks):**
```
Week 1:
  - 5× 2-hour documentation sessions (recorded)
  - Created 12 runbooks
  - Updated system architecture diagrams
  - Introduced team to payment gateway vendors

Week 2:
  - 3 team members shadowed him
  - Practice drills: "Payment system is down, fix it"
  - Validation: Each team member walked through procedures
  - Q&A session: "What am I not asking that I should?"
```

**What we got:**
```
Documentation:
  - 12 detailed runbooks
  - 5 hours of video recordings
  - 3 architectural diagrams
  - Vendor contact list with relationship history

Training:
  - 3 team members can now handle payment system
  - 2 can troubleshoot independently
  - 1 designated as primary owner

Gaps identified:
  - Some vendor-specific knowledge lost (unavoidable)
  - Workarounds he used but forgot to mention
  - Historical context for some decisions
```

**What we did after he left:**
```
Month 1: Discovered gaps
  - Hit issues not in runbooks (3 incidents)
  - Documented them immediately
  - Called Rajesh (he was helpful even after leaving)

Month 2-3: Closed gaps
  - Vendor training sessions
  - More detailed troubleshooting guides
  - Added to runbooks

After 6 months:
  - Team fully autonomous
  - Documentation comprehensive
  - No longer dependent on Rajesh
```

**For Nomura:**
'Two weeks is tight, but with focused effort, we can capture 80% of critical knowledge. The key is prioritization - focus on production-impacting procedures first, document quickly, validate with practice, and accept that some tribal knowledge will be discovered over the next few months as issues arise. The important thing is having a solid foundation and knowing where to find help when needed.'"

---

## SECTION 4: STRATEGIC SCENARIOS

### Scenario 11: Building Automation Roadmap

**Interviewer:** "You join Nomura's TOC team. Day 1, they ask you to build a 12-month automation roadmap. Where do you start?"

**Your Answer:**

"Building a roadmap requires understanding current state before defining future state:

**Month 1: Assessment & Discovery**

**Week 1-2: Data Collection**
```
Activities:
1. Shadow ops team for 3 days
   - Watch what they do hourly
   - Ask: "What % of your day is repetitive?"
   - Observe pain points

2. Pull metrics:
   - ServiceNow: Ticket volume, types, resolution time
   - Monitoring: Incident frequency, MTTR
   - Deployment: Frequency, duration, success rate
   
3. Interview stakeholders:
   - TOC manager: What are your top 3 pain points?
   - Team members: What task do you hate most?
   - Application teams: What do you need from TOC?
   - Security: What are compliance requirements?
```

**Week 3: Gap Analysis**

**Framework I use:**

```
Current State vs. Desired State:

Dimension: Incident Response
Current: 45 min MTTR, manual troubleshooting
Desired: 15 min MTTR, auto-remediation for common issues
Gap: Need playbooks, automation, better monitoring

Dimension: Deployment
Current: Weekly, manual, 2 hours, 20% failure rate
Desired: Daily, automated, 15 min, <5% failure rate
Gap: Need CI/CD pipeline, automated testing, rollback capability

Dimension: Ticket Management
Current: 2,000/month, 80% manual, email/phone tracking
Desired: 500/month, 80% automated, self-service portal
Gap: Need catalog items, automation backend, integration

[Continue for all operational areas]
```

**Week 4: Prioritization**

**My prioritization matrix:**
```
Initiative               | Business Value | Effort | Dependencies | Priority
-------------------------|----------------|--------|--------------|----------
Auto-remediation (top 5) | High          | Medium | Monitoring   | 1
CI/CD pipeline           | High          | High   | None         | 2
Self-service portal      | High          | Medium | Automation   | 3
Enhanced monitoring      | Medium        | Medium | None         | 4
Deployment automation    | Medium        | Low    | CI/CD        | 5
```

**Month 2-3: Build Roadmap**

**12-Month Roadmap Structure:**

**Q1 (Month 1-3): Foundation**
```
Goal: Reduce toil, establish capabilities

Initiatives:
1. Enhanced Monitoring & Observability
   - Deploy Prometheus + Grafana
   - Define SLIs/SLOs for top 10 services
   - Alert on SLO violations (not thresholds)
   - Outcome: Proactive issue detection

2. Automation Framework Setup
   - Deploy AWX/Ansible Tower
   - Create automation repository structure
   - Define standards (naming, structure, testing)
   - Outcome: Platform for building automation

3. Quick Win Automation (Top 3 Pain Points)
   - Application restart automation
   - Log collection automation
   - Disk cleanup automation
   - Outcome: 30% ticket reduction

Metrics:
  - Ticket volume: 2,000 → 1,400/month
  - MTTR: 45 → 35 minutes
  - Team toil: 60% → 45%
```

**Q2 (Month 4-6): Scale**
```
Goal: Self-service, auto-remediation

Initiatives:
4. Self-Service Portal (ServiceNow)
   - Catalog items for common requests
   - Integration with automation backend
   - User training and adoption
   - Outcome: Ticket deflection

5. Auto-Remediation (Monitoring → Action)
   - Disk space alert → Auto-cleanup
   - App unresponsive → Auto-restart
   - High memory → Alert + diagnostic collection
   - Outcome: Reduce manual intervention

6. Basic CI/CD Pipeline
   - Jenkins setup
   - Build → Test → Deploy for 2 pilot apps
   - Automated rollback capability
   - Outcome: Faster, safer deployments

Metrics:
  - Ticket volume: 1,400 → 800/month
  - MTTR: 35 → 20 minutes
  - Team toil: 45% → 25%
  - Deployment frequency: 1x/week → 3x/week
```

**Q3 (Month 7-9): Optimize**
```
Goal: Expand coverage, improve quality

Initiatives:
7. Expand CI/CD Coverage
   - Onboard 10 more applications
   - Add security scanning (SonarQube, Trivy)
   - Deployment quality gates
   - Outcome: 80% of apps on CI/CD

8. Advanced Auto-Remediation
   - Self-healing infrastructure
   - Predictive alerting (based on trends)
   - Automated scaling (based on load)
   - Outcome: Proactive operations

9. Infrastructure as Code
   - Terraform for cloud resources
   - Ansible for configuration
   - GitOps workflow
   - Outcome: Repeatable, version-controlled infrastructure

Metrics:
  - Ticket volume: 800 → 400/month
  - MTTR: 20 → 12 minutes
  - Team toil: 25% → 15%
  - Deployment frequency: 3x/week → daily
  - Deployment failure rate: 20% → 8%
```

**Q4 (Month 10-12): Innovation**
```
Goal: Advanced capabilities, continuous improvement

Initiatives:
10. AIOps (Machine Learning for Ops)
    - Anomaly detection
    - Root cause analysis automation
    - Capacity forecasting
    - Outcome: Intelligent operations

11. ChatOps Integration
    - Slack-based operations
    - Approval workflows in chat
    - Status dashboards in Slack
    - Outcome: Improved collaboration

12. Chaos Engineering
    - Controlled failure injection
    - Resilience testing
    - Disaster recovery automation
    - Outcome: Improved reliability

Metrics:
  - Ticket volume: 400 → 200/month
  - MTTR: 12 → 8 minutes
  - Team toil: 15% → 10%
  - Deployment frequency: Daily → Multiple/day
  - Deployment failure rate: 8% → 3%
```

**Roadmap Communication:**

**To Leadership (Executive Summary):**
```
12-Month Automation Roadmap

Goal: Transform TOC from reactive firefighting to proactive service delivery

Investment Required:
  - Tools: $150K
  - Team time: 40% capacity (rest for BAU)
  - Training: $20K

Expected Outcomes (12 months):
  - 90% reduction in manual tickets
  - 82% reduction in MTTR
  - 85% reduction in toil
  - 10x increase in deployment frequency
  - $500K annual cost savings

Quarterly Milestones:
  Q1: Foundation (monitoring, basic automation)
  Q2: Self-service (portal, auto-remediation)
  Q3: Optimization (full CI/CD, IaC)
  Q4: Innovation (AIOps, ChatOps)

Risk Mitigation:
  - Phased approach (fail fast, learn, adapt)
  - Pilot programs before full rollout
  - Regular checkpoints and course corrections
```

**To Team (Detailed View):**
```
[Gantt chart showing timeline]
[Detailed initiative descriptions]
[Skills needed and training plan]
[Success criteria for each initiative]
```

**Governance:**

**Monthly Reviews:**
```
Review metrics:
  - Are we on track?
  - What's blocking progress?
  - Do we need to adjust priorities?

Adjust roadmap based on:
  - Business changes
  - New pain points discovered
  - Technology changes
  - Team feedback
```

**Real Example from Birlasoft:**

**Our 12-month roadmap (actual):**
```
Q1: Basic automation (AWS, Jenkins, Ansible)
  Result: 35% ticket reduction

Q2: ServiceNow integration, self-service
  Result: 60% ticket reduction (cumulative)

Q3: Advanced CI/CD, monitoring
  Result: 75% ticket reduction, 3x deployment frequency

Q4: Optimization, innovation
  Result: 80% ticket reduction, 5x deployment frequency

Year-end outcomes:
  - Exceeded all targets
  - Team satisfaction: Dramatically improved
  - Business impact: $480K annual savings
  - Team capacity freed: Used for strategic initiatives
```

**What I learned:**
```
✅ Start with pain points (gets buy-in)
✅ Show value early (builds momentum)
✅ Be flexible (adjust based on reality)
✅ Celebrate wins (team morale)

❌ Don't boil the ocean (too ambitious)
❌ Don't ignore team feedback (they know reality)
❌ Don't over-invest in tools (maximize existing tools first)
```

**For Nomura:**
'I'd spend the first month deeply understanding TOC operations, pain points, and constraints. Then build a phased roadmap that delivers measurable value each quarter, starting with high-impact, low-effort initiatives to build momentum and trust. The key is balancing quick wins with foundational work that enables long-term success.'"

---

## KEY PREPARATION POINTS FOR NOMURA INTERVIEW

### Your Strongest Talking Points

**1. Proven Track Record:**
- Reduced manual effort by 30% at Birlasoft
- Improved MTTR through automation
- Successfully deployed AWX on Kubernetes in production
- Implemented observability (Dynatrace, Prometheus)

**2. Enterprise Experience:**
- Financial services background (Northern Trust client)
- Large-scale operations (200+ servers)
- Compliance-aware (ITIL, change management)
- Multi-environment management (dev/qa/staging/prod)

**3. Cross-Team Collaboration:**
- Worked with dev, QA, security teams
- Aligned automation with business goals
- Change management experience
- Stakeholder communication

**4. Technical Breadth:**
- CI/CD (Jenkins, GitHub Actions)
- IaC (Terraform, Ansible, AWX)
- Containers (Docker, Kubernetes)
- Monitoring (Prometheus, Grafana, Dynatrace)
- Cloud (AWS, VMware)

### Questions to Ask Nomura

**About the Role:**
1. "What are the top 3 pain points TOC is currently facing?"
2. "What does success look like in this role in 6 months? 12 months?"
3. "What's the current automation maturity level?"

**About the Team:**
4. "What's the team structure? Who will I collaborate with most?"
5. "What's the on-call rotation like?"
6. "How does TOC interact with application teams?"

**About Technology:**
7. "What's the current automation toolset? Any plans to change?"
8. "Is there a preferred automation framework/standard?"
9. "What's the deployment frequency goal?"

**About Culture:**
10. "How does Nomura balance innovation with stability in operations?"
11. "What's the approach to learning and experimentation?"
12. "How are automation initiatives prioritized?"

### Common Pitfalls to Avoid

❌ **Don't:** Focus only on tools
✅ **Do:** Focus on business outcomes and problem-solving

❌ **Don't:** Say "automation will replace manual work"
✅ **Do:** Say "automation frees team for strategic work"

❌ **Don't:** Over-promise ("I can automate everything in 3 months")
✅ **Do:** Be realistic with timelines and complexity

❌ **Don't:** Dismiss legacy systems
✅ **Do:** Show respect for existing processes while suggesting improvements

❌ **Don't:** Be overly technical in scenarios
✅ **Do:** Balance technical details with business context

### Final Tips

1. **Use STAR method** for behavioral questions:
   - Situation: Context
   - Task: What needed to be done
   - Action: What you did
   - Result: Measurable outcome

2. **Quantify everything:**
   - Don't: "Reduced tickets"
   - Do: "Reduced tickets by 40% (800 → 480/month)"

3. **Show learning mindset:**
   - Mention how you stay current (certifications, conferences)
   - Discuss failures and what you learned
   - Show curiosity about Nomura's challenges

4. **Emphasize collaboration:**
   - TOC automation requires cross-team alignment
   - Show empathy for different perspectives
   - Demonstrate communication skills

**Good luck with your Nomura interview!**