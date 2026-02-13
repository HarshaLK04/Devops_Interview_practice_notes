
# DevOps & SRE Interview Questions (TSYS/Grok)

This document provides a curated list of DevOps and SRE interview questions, ranging from foundational to advanced, with sample answers and real-world context. Use this as a study guide or reference for technical interviews.

---

## Basic Level Questions

These questions test core DevOps concepts such as CI/CD, automation, containers, IaC, cloud, and agile practices. When answering, be concise, use real examples, and relate your experience to team outcomes.

### 1. What is CI/CD, and why is it important in DevOps?
**Answer:**
Continuous Integration (CI) involves frequently merging code changes into a shared repository, with automated builds and tests to catch issues early. Continuous Delivery/Deployment (CD) automates the release process, making deployments reliable and repeatable. CI/CD reduces manual errors, speeds up delivery, and improves collaboration. For example, managing Jenkins pipelines helped me cut deployment time by 50% on a project.

### 2. Explain Infrastructure as Code (IaC) and name a tool you've used.
**Answer:**
IaC manages and provisions infrastructure through code, enabling version control, automation, and consistency. I have used Terraform, which is declarative, cloud-agnostic, and integrates with CI/CD. I scripted Terraform modules to provision AWS resources, ensuring reproducible environments.

### 3. What is Docker, and how does it differ from a virtual machine?
**Answer:**
Docker is a containerization platform that packages applications and dependencies into lightweight, portable containers. Unlike VMs, Docker shares the host OS kernel, making it faster and more efficient. I have used Docker to containerize apps, reducing deployment inconsistencies.

### 4. Describe Ansible and its role in configuration management.
**Answer:**
Ansible is an agentless automation tool for configuration management, orchestration, and deployment. It uses YAML playbooks and is idempotent. I have automated server configs and patching with Ansible, aligning with ITIL change management.

### 5. What is Kubernetes, and what are its main components?
**Answer:**
Kubernetes (K8s) is an open-source container orchestration platform. Key components: Pods, Nodes, Control Plane (API Server, etcd, Scheduler, Controller Manager), and Services. I have set up clusters for auto-scaling and high availability.

### 6. How does Jenkins fit into a CI/CD pipeline? Give an example.
**Answer:**
Jenkins is an automation server for building, testing, and deploying code. It supports plugins for tools like Docker and Ansible. I have configured multi-branch pipelines where a Git push built Docker images and deployed to Kubernetes via Ansible.

### 7. What is Terraform, and how do you handle state management?
**Answer:**
Terraform is an IaC tool for building and versioning infrastructure. State management tracks resource states in a file (terraform.tfstate), which should be stored remotely (e.g., S3, GCS) with locking. I have used remote backends in GCP projects to avoid conflicts.

### 8. Explain the difference between Nexus and Artifactory as artifact repositories.
**Answer:**
Both are artifact repository managers. Nexus supports formats like Maven, npm, Docker and is flexible for proxying. Artifactory offers universal support and advanced metadata/search. I have used Nexus to cache dependencies and speed up CI/CD builds.

### 9. What are SRE practices, and how do they relate to DevOps?
**Answer:**
Site Reliability Engineering (SRE) focuses on reliability using software engineering for operations. Practices include SLIs/SLOs, error budgets, and automation. SRE complements DevOps by emphasizing monitoring, incident response, and blameless post-mortems. I have set up cloud monitoring to maintain 99.9% uptime.

### 10. Describe Scrum and Kanban in agile delivery.
**Answer:**
Scrum uses fixed-length sprints, defined roles, and ceremonies. Kanban is flow-based, visualizing work to limit WIP and optimize throughput. I have used Scrum for structured planning and Kanban for ongoing ops tasks.

### 11. How would you troubleshoot a failed deployment in a CI/CD pipeline?
**Answer:**
Check pipeline logs for errors, verify IaC configs, test in staging, and use tools like kubectl for Kubernetes. Collaborate with developers. I once resolved a Docker image push failure by updating Nexus credentials in the pipeline.

### 12. What cloud platforms have you worked with, and why prefer GCP?
**Answer:**
I have worked with AWS, Azure, and GCP. GCP is preferred for analytics (BigQuery), Kubernetes-native services (GKE), and cost-effective AI/ML tools. I have used GCP for container orchestration and Terraform integration.

### 13. Explain release management and its importance.
**Answer:**
Release management coordinates planning, scheduling, and deployment of software releases, ensuring reliability and compliance. I have used tools like Harness to orchestrate multi-environment deployments and reduce downtime.

### 14. How do you automate scripting for deployments?
**Answer:**
I use Bash, Python, or Groovy for scripting. For example, I automate pre-checks, deployments via Ansible, and post-deploy validations. I have scripted Kubernetes rollouts with Helm for hands-off releases.

### 15. Why is being a team player important in DevOps?
**Answer:**
DevOps relies on collaboration to break silos. I communicate issues early, share knowledge, and work cross-functionally, which leads to faster resolutions and better team outcomes.

---

## Advanced ("Humiliating") Level Questions

These scenario-based questions probe deep expertise and troubleshooting skills. Stay calm, structure answers with problem analysis, step-by-step resolution, and lessons learned.

### 1. Kubernetes pod evictions due to resource pressure (without adding nodes)
**Answer:**
Use `kubectl top` and `describe pods` to find resource hogs. Check Prometheus metrics. Review node affinities. Tune resource requests/limits, implement HPA or ResourceQuotas, and use anti-affinities for better distribution. I reduced evictions by 80% by rightsizing pods.

### 2. Terraform apply fails with cyclic dependency in GCP module
**Answer:**
Cyclic dependencies occur when resources reference each other. Modularize code, separate VPC and GKE modules, use outputs and data sources, and visualize with `terraform graph`. I have used remote state references to ensure acyclic applies.

### 3. Jenkins pipeline hangs on parallel Ansible stage (100+ nodes, SSH timeouts)
**Answer:**
Check Ansible inventory and connection limits. Use `--forks` to limit concurrency. Verify SSH keys and network. Add timeouts/retries in Jenkins. Consider Ansible Tower or async modes. I have tuned pipelines with dynamic inventories for scale.

### 4. SRE: Service breaches SLO due to Istio misconfiguration
**Answer:**
Calculate error budget. Conduct blameless post-mortem. Use traces (Jaeger), circuit breakers, retries, canary deployments, and chaos engineering. I have set SLOs in GCP Monitoring and automated alerts.

### 5. Artifactory vs. Nexus: Hybrid repo setup for monorepo (Docker, Maven, npm, air-gapped)
**Answer:**
Use Nexus for proxying in air-gapped environments. Proxy Artifactory repos in Nexus or vice versa. Handle conflicts with virtual repos and script promotions. I have configured Nexus for monorepos and offline builds.

### 6. Harness pipeline fails on GCP due to GKE quota exhaustion
**Answer:**
Profile usage with Stackdriver. Tune autoscaler configs, implement bin-packing, and add pre-checks for quotas. I have used spot instances in GKE to optimize costs and avoid quota issues.

### 7. Zero-downtime deployments in Kubernetes (blue-green, Ansible, DB changes)
**Answer:**
Deploy to a green environment, test, then switch traffic with Service or Ingress. Use Ansible for config updates. Handle DB changes with migration tools (e.g., Liquibase). Rollback by reverting selectors. I have scripted this in Jenkins for seamless releases.

### 8. Automate ITIL-compliant change requests with DevOps tools
**Answer:**
Integrate Jira/ServiceNow with Jenkins for automated approvals. Use IaC and Git for audit trails. Tools like Auditbeat for logging. I have automated workflows with webhooks for compliance.

### 9. Scrum team velocity drops due to DevOps bottlenecks (apply Kanban)
**Answer:**
Visualize bottlenecks on a Kanban board (Scrumban), limit WIP, and use swimlanes for CI/CD. Retrospectives to identify issues. I have improved velocity by hybridizing Scrum and Kanban.

### 10. Docker build "exec format error" on arm64 in GCP CI/CD
**Answer:**
This is a binary architecture mismatch. Use `docker buildx` for multi-platform builds, QEMU for emulation, and `--platform` flags. I have configured Jenkins agents for cross-builds to ensure compatibility.

### 11. Resilient multi-cloud IaC setup (AWS/GCP failover, Terraform, Ansible)
**Answer:**
Use Terraform workspaces/modules for each cloud, script DNS failover, and use Ansible for post-provision configs. Test with chaos tools. I have built setups with remote exec provisioners for seamless failover.

### 12. Kubernetes pods OOMKilled (JVM apps, GCP integration)
**Answer:**
Use `kubectl debug` or heap dumps. Tune JVM `-Xmx` below container limits, monitor with GCP Profiler, and adjust requests. I have prevented OOMKills by tuning based on metrics.

### 13. Release orchestration with Harness (canary, Prometheus SRE metrics)
**Answer:**
Define canary steps with traffic splits, integrate Prometheus queries for verification, and rollback on failure. I have set up zero-downtime releases using custom metrics.

### 14. Python automation for config drift detection (Ansible, GCP Pub/Sub)
**Answer:**
Use Ansible `--check` mode in a script, parse output, and publish drifts to Pub/Sub. Schedule with cron or Cloud Scheduler. I have built similar compliance integrations.

### 15. Why might a senior DevOps engineer fail in a shift-based role despite strong tech skills?
**Answer:**
Possible reasons include poor communication, lack of adaptability, inability to handle handoffs, or resistance to process. Success in shift-based roles requires strong teamwork, documentation, and flexibility, not just technical expertise.
Answer: Poor soft skills like communication during handoffs, leading to knowledge gaps. Or burnout from shifts without work-life balance. Success requires adaptability; I've thrived by documenting thoroughly and collaborating across time zones.
