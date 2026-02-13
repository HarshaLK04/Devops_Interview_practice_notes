# How Kubernetes Works - Complete Explanation for Interviews

Here's a comprehensive explanation you can use when an interviewer asks "How does Kubernetes work?"

---

## High-Level Answer (30 seconds)

*"Kubernetes is a container orchestration platform that automates the deployment, scaling, and management of containerized applications. It works on a master-worker architecture where the control plane manages the cluster state, and worker nodes run the actual application containers. When you submit a deployment, Kubernetes continuously monitors and maintains your desired state - if a container crashes, it automatically restarts it. It provides features like service discovery, load balancing, automated rollouts, and self-healing."*

---

## Detailed Architecture & Workflow

### 1. **Kubernetes Architecture**

```
┌─────────────────────────────────────────────────────┐
│              CONTROL PLANE (Master)                 │
├─────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │  API Server  │  │  Scheduler   │  │Controller │ │
│  │              │  │              │  │  Manager  │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              etcd (Cluster State)           │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼───────┐ ┌──────▼──────┐ ┌──────▼──────┐
│  WORKER NODE  │ │ WORKER NODE │ │ WORKER NODE │
├───────────────┤ ├─────────────┤ ├─────────────┤
│   Kubelet     │ │   Kubelet   │ │   Kubelet   │
│   Kube-proxy  │ │   Kube-proxy│ │   Kube-proxy│
│               │ │             │ │             │
│ ┌───────────┐ │ │ ┌─────────┐ │ │ ┌─────────┐ │
│ │ Pod (App) │ │ │ │   Pod   │ │ │ │   Pod   │ │
│ └───────────┘ │ │ └─────────┘ │ │ └─────────┘ │
│ ┌───────────┐ │ │ ┌─────────┐ │ │ ┌─────────┐ │
│ │    Pod    │ │ │ │   Pod   │ │ │ │   Pod   │ │
│ └───────────┘ │ │ └─────────┘ │ │ └─────────┘ │
└───────────────┘ └─────────────┘ └─────────────┘
```

---

## Complete Deployment Workflow (Step-by-Step)

### **Scenario: You run `kubectl apply -f deployment.yaml`**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
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
        image: nginx:1.24
        ports:
        - containerPort: 80
```

---

### **Step 1: kubectl → API Server**

```
Developer
    │
    │ kubectl apply -f deployment.yaml
    ▼
API Server (validates request)
    │
    │ Authentication: Is user authenticated?
    │ Authorization: Does user have permissions?
    │ Admission Control: Does request meet policies?
    ▼
Request Accepted
```

**What happens:**
- `kubectl` sends HTTP request to API Server
- API Server authenticates you (using kubeconfig)
- Checks if you have permissions (RBAC)
- Validates the YAML syntax and schema
- Runs admission controllers (webhooks, policies)

---

### **Step 2: API Server → etcd (Store State)**

```
API Server
    │
    │ Store deployment spec in etcd
    ▼
etcd (Cluster Database)
    │
    │ Deployment "webapp" created
    │ Desired State: 3 replicas
    │ Status: Not yet created
    ▼
State Saved
```

**What happens:**
- API Server writes deployment object to **etcd**
- etcd is a distributed key-value store (cluster's brain)
- Stores **desired state**: "I want 3 pods running nginx"
- All cluster state lives here (deployments, pods, services, etc.)

---

### **Step 3: Deployment Controller Detects Change**

```
Deployment Controller (watches API Server)
    │
    │ "New deployment detected!"
    │ Desired: 3 replicas
    │ Current: 0 replicas
    │ Action needed: Create ReplicaSet
    ▼
Creates ReplicaSet Object
    │
    │ POST /api/v1/replicasets
    ▼
API Server → etcd
    │
    │ ReplicaSet "webapp-abc123" created
    │ Desired: 3 pods
    ▼
Stored in etcd
```

**What happens:**
- **Deployment Controller** runs in Control Plane
- Constantly watches API Server for deployment changes
- Sees new deployment needs 3 replicas
- Creates a **ReplicaSet** object to manage pods
- ReplicaSet spec stored in etcd

---

### **Step 4: ReplicaSet Controller Creates Pods**

```
ReplicaSet Controller (watches API Server)
    │
    │ "New ReplicaSet detected!"
    │ Desired: 3 pods
    │ Current: 0 pods
    │ Action: Create 3 pod objects
    ▼
Creates 3 Pod Objects
    │
    │ POST /api/v1/pods (webapp-abc123-xyz1)
    │ POST /api/v1/pods (webapp-abc123-xyz2)
    │ POST /api/v1/pods (webapp-abc123-xyz3)
    ▼
API Server → etcd
    │
    │ 3 Pods created (status: Pending)
    │ No node assigned yet
    ▼
Stored in etcd
```

**What happens:**
- **ReplicaSet Controller** sees new ReplicaSet
- Creates 3 individual Pod objects
- Pods are in "Pending" state (not running yet)
- No node assigned (that's Scheduler's job)

---

### **Step 5: Scheduler Assigns Pods to Nodes**

```
Scheduler (watches for unscheduled pods)
    │
    │ "Found 3 pending pods!"
    │ Need to assign to nodes
    ▼
Scheduling Algorithm
    │
    ├─ Filter Nodes
    │  │ ✅ Node1: Has enough CPU/Memory
    │  │ ✅ Node2: Has enough CPU/Memory
    │  │ ❌ Node3: Insufficient memory
    │
    ├─ Score Nodes
    │  │ Node1: Score 85 (least loaded)
    │  │ Node2: Score 72 (more loaded)
    │
    ├─ Binding Decision
    │  │ Pod1 → Node1
    │  │ Pod2 → Node2
    │  │ Pod3 → Node1
    ▼
Update pods with node assignments
    │
    │ PATCH /api/v1/pods/webapp-abc123-xyz1
    │   spec.nodeName: node1
    ▼
API Server → etcd
    │
    │ Pods updated with node assignments
    ▼
Stored
```

**What happens:**
- **Scheduler** watches for pods without node assignment
- Runs two-phase algorithm:
  1. **Filtering**: Eliminates nodes that can't run pod
     - Insufficient resources (CPU/memory)
     - Node selectors don't match
     - Taints/tolerations conflicts
  2. **Scoring**: Ranks suitable nodes
     - Resource balance
     - Pod spreading
     - Affinity rules
- Assigns each pod to best node
- Updates pod objects with `nodeName` field

---

### **Step 6: Kubelet Runs Containers**

```
Kubelet on Node1 (watches API Server)
    │
    │ "I have 2 pods assigned to me!"
    │ Pod1: webapp-abc123-xyz1
    │ Pod3: webapp-abc123-xyz3
    ▼
For each pod:
    │
    ├─ Pull Image
    │  │ docker pull nginx:1.24
    │  │ (or containerd/cri-o pull)
    │  ▼
    │  Image downloaded
    │
    ├─ Create Container
    │  │ docker run nginx:1.24
    │  │ Apply resource limits
    │  │ Set up networking
    │  ▼
    │  Container started
    │
    ├─ Health Checks
    │  │ Run liveness probe
    │  │ Run readiness probe
    │  ▼
    │  Container healthy
    │
    ├─ Report Status
    │  │ PATCH /api/v1/pods/webapp-abc123-xyz1
    │  │   status.phase: Running
    │  │   status.containerStatuses[0].ready: true
    │  ▼
    │  API Server → etcd
    │
    ▼
Pod Running Successfully!
```

**What happens:**
- **Kubelet** (agent on each node) watches API Server
- Sees pods assigned to its node
- For each pod:
  1. **Pulls container image** from registry
  2. **Creates container** using container runtime (Docker/containerd)
  3. **Sets up networking** (assigns IP, DNS)
  4. **Applies resource limits** (CPU/memory)
  5. **Runs health checks** (liveness/readiness probes)
  6. **Reports status** back to API Server
- Pod transitions: Pending → ContainerCreating → Running

---

### **Step 7: Service Discovery & Networking**

```
Service Object (if you created one)
    │
    │ apiVersion: v1
    │ kind: Service
    │ metadata:
    │   name: webapp-service
    │ spec:
    │   selector:
    │     app: webapp
    │   ports:
    │   - port: 80
    │     targetPort: 80
    ▼
Kube-proxy (on each node)
    │
    │ Watches Service objects
    │ Creates iptables/IPVS rules
    │
    ├─ Rule 1: webapp-service:80
    │  │ → LoadBalance to:
    │  │   - Pod1 IP: 10.244.1.5:80
    │  │   - Pod2 IP: 10.244.2.7:80
    │  │   - Pod3 IP: 10.244.1.6:80
    │
    ├─ DNS Entry
    │  │ webapp-service.default.svc.cluster.local
    │  │ → Resolves to Service ClusterIP
    │
    ▼
Traffic flows to pods
```

**What happens:**
- **kube-proxy** (runs on each node) watches Services
- Creates **iptables/IPVS rules** for load balancing
- **DNS** entry created automatically
- Any pod can access service by name:
  ```bash
  curl http://webapp-service
  ```
- Traffic distributed across all healthy pods

---

### **Step 8: Continuous Reconciliation (Self-Healing)**

```
┌─────────────────────────────────────┐
│     Continuous Monitoring Loop      │
└─────────────────────────────────────┘
           │
           ▼
    ┌──────────────┐
    │  Controller  │ ←─────────────┐
    │   Manager    │               │
    └──────┬───────┘               │
           │                       │
           ├─ Check: Desired = 3 pods
           │         Current = 3 pods
           │         ✅ OK          │
           │                       │
    [Pod crashes]                 │
           │                       │
           ├─ Check: Desired = 3 pods
           │         Current = 2 pods
           │         ❌ Need action!
           │                       │
           ├─ Create new pod      │
           │                       │
           ├─ Scheduler assigns    │
           │                       │
           ├─ Kubelet starts       │
           │                       │
           └───────────────────────┘
                  Reconciled!
```

**What happens:**
- Controllers run in **infinite loops** (reconciliation loops)
- Constantly comparing **desired state** vs **current state**
- If mismatch detected:
  - Pod crashed? → Create new pod
  - Too few replicas? → Create more
  - Too many? → Delete excess
  - Wrong image version? → Rolling update
- This is **self-healing** in action

---

## Interview Talking Points

### **1. Control Plane Components**

```
┌─────────────────────────────────────┐
│        CONTROL PLANE                │
├─────────────────────────────────────┤
│ • API Server: Front-end, REST API   │
│ • etcd: Database, stores state      │
│ • Scheduler: Assigns pods to nodes  │
│ • Controller Manager: Reconciliation│
│ • Cloud Controller: Cloud integration│
└─────────────────────────────────────┘
```

**Explain:**
- **API Server**: "The heart of Kubernetes - all communication goes through it"
- **etcd**: "Distributed database that stores the entire cluster state"
- **Scheduler**: "Decides which node should run each pod based on resources"
- **Controller Manager**: "Runs controllers that maintain desired state - deployment controller, replicaset controller, node controller, etc."

---

### **2. Worker Node Components**

```
┌─────────────────────────────────────┐
│         WORKER NODE                 │
├─────────────────────────────────────┤
│ • Kubelet: Node agent, runs pods    │
│ • Kube-proxy: Network proxy         │
│ • Container Runtime: Docker/containerd│
└─────────────────────────────────────┘
```

**Explain:**
- **Kubelet**: "Agent on each node that talks to API Server, pulls images, starts containers"
- **Kube-proxy**: "Handles networking, creates iptables rules for services"
- **Container Runtime**: "Actually runs the containers - Docker, containerd, CRI-O"

---

### **3. Key Kubernetes Objects**

```
Deployment
    ├── ReplicaSet
    │   ├── Pod
    │   ├── Pod
    │   └── Pod
    └── (manages versions)

Service
    └── (routes traffic to pods)

ConfigMap
    └── (configuration data)

Secret
    └── (sensitive data)

PersistentVolume
    └── (storage)
```

**Explain:**
- **Pod**: "Smallest deployable unit - one or more containers"
- **Deployment**: "Manages rollouts, updates, scaling"
- **ReplicaSet**: "Ensures desired number of pod replicas"
- **Service**: "Stable network endpoint for pods"

---

### **4. Real Production Example** (from your experience)

*"At Birlasoft, when we deployed microservices to Kubernetes:*

1. *Developers push code to GitHub*
2. *CI/CD pipeline (Jenkins) builds Docker image*
3. *Image pushed to JFrog Artifactory*
4. *Jenkins updates deployment YAML with new image tag*
5. *kubectl apply triggers rolling update*
6. *Kubernetes gradually replaces old pods with new ones*
7. *If health checks fail, automatic rollback occurs*
8. *Dynatrace and Prometheus monitor the rollout*

*This gave us zero-downtime deployments and the ability to roll back instantly if issues occurred."*

---

### **5. Self-Healing Example**

```
Time: 10:00 AM
State: 3 pods running ✅

Time: 10:15 AM
Event: Node1 crashes 💥
Current: 1 pod running ❌

Kubernetes Actions:
├─ Node Controller detects node down
├─ Marks node as NotReady
├─ Deployment Controller sees pods missing
├─ Creates new pods
├─ Scheduler assigns to healthy nodes
└─ Kubelet starts containers

Time: 10:17 AM
State: 3 pods running ✅
```

**Explain:**
*"If a node crashes, Kubernetes automatically detects it, marks pods as failed, and reschedules them on healthy nodes. This happens without any manual intervention - it's part of Kubernetes' self-healing capability."*

---

### **6. Scaling Example**

```bash
# Scale up
kubectl scale deployment webapp --replicas=10

# What happens:
API Server → etcd (update desired replicas to 10)
Deployment Controller → sees 3 current, 10 desired
ReplicaSet Controller → creates 7 new pods
Scheduler → assigns to nodes
Kubelet → starts containers

# Time: ~30 seconds for 7 new pods
```

---

## Common Interview Questions & Answers

### **Q: What happens when you create a deployment?**

**A:** 
*"When you run kubectl apply -f deployment.yaml:*
1. *kubectl sends request to API Server*
2. *API Server authenticates, authorizes, validates, and stores in etcd*
3. *Deployment Controller sees new deployment and creates a ReplicaSet*
4. *ReplicaSet Controller creates Pod objects*
5. *Scheduler assigns pods to nodes based on resources*
6. *Kubelet on each node pulls images and starts containers*
7. *kube-proxy sets up networking rules*
8. *Controllers continuously monitor and maintain desired state"*

---

### **Q: How does Kubernetes handle pod failures?**

**A:**
*"Kubernetes uses a reconciliation loop:*
1. *ReplicaSet Controller constantly checks: desired replicas vs current replicas*
2. *If pod crashes, current < desired*
3. *Controller immediately creates new pod to replace it*
4. *Scheduler assigns to a healthy node*
5. *Kubelet starts the container*
6. *Typically takes 10-30 seconds from crash to new pod running*

*I've seen this in production at Birlasoft - when pods crashed due to OOM errors, Kubernetes automatically restarted them while we investigated the root cause."*

---

### **Q: What's the difference between Deployment and ReplicaSet?**

**A:**
*"ReplicaSet just ensures N pod replicas are running. Deployment is higher-level:*
- *Manages ReplicaSets*
- *Handles rolling updates (creates new ReplicaSet, gradually shifts traffic)*
- *Maintains revision history for rollbacks*
- *Provides declarative update strategy*

*You typically never create ReplicaSets directly - you create Deployments, which create and manage ReplicaSets for you."*

---

### **Q: How does service discovery work?**

**A:**
*"Kubernetes has built-in DNS:*
1. *Every Service gets a DNS name: `service-name.namespace.svc.cluster.local`*
2. *Pods can reference services by name: `http://webapp-service`*
3. *kube-proxy creates iptables rules that load-balance traffic*
4. *When you call the service, traffic is distributed across all healthy pods*
5. *If pods are added/removed, kube-proxy automatically updates rules*

*This means applications don't need hardcoded IPs - they just use service names, and Kubernetes handles the routing."*

---

## Visual Summary for Interview

```
USER
 │
 │ kubectl apply -f deployment.yaml
 ▼
API SERVER ──────────────► etcd (stores state)
 │
 ├──► Deployment Controller ──► Creates ReplicaSet
 │
 ├──► ReplicaSet Controller ──► Creates Pods
 │
 ├──► Scheduler ──────────────► Assigns Pods to Nodes
 │
 ▼
NODES
 ├─ Kubelet ─────────────────► Pulls image, starts container
 └─ Kube-proxy ──────────────► Sets up networking

CONTINUOUS:
 └─ Controllers monitor ──────► Self-healing, scaling
```

---

This explanation covers everything you need for an interview. Start with the high-level answer, then go deeper based on their questions!