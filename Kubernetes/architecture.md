**Architecture:**

Suppose I deploy an nginx application with 3 replicas. The request goes to the API server, which stores the configuration in etcd. The scheduler assigns pods to worker nodes. The kubelet starts the containers, and kube-proxy ensures they are accessible over the network. If one pod fails, the controller manager automatically recreates it.

**Visual Summary for Interview**
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

**issues faces in kubernetes:**
If pod fails, I first check using kubectl get pods and kubectl describe pod to see status and events. If CrashLoopBackOff, I check logs using kubectl logs and fix application or configuration.

If ImagePullBackOff, I verify image name and registry access.

If node fails, I check using kubectl get nodes. If node is NotReady, I check kubelet status and system resources like CPU, memory, and disk. Then restart kubelet or fix resource issue.

If pod is Pending, I check scheduler events for insufficient resources.

