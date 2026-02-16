---

# 🚀 CKA Quick Revision – Cluster Architecture

## Control Plane (Master Node)

### kube-apiserver
- Central communication hub
- Only component that talks to etcd
- Handles all API requests

### etcd
- Key-value database
- Stores complete cluster state
- Single source of truth

### kube-scheduler
- Assigns Pods to worker nodes
- Checks CPU, memory, constraints
- Does NOT create Pods

### Controller Manager
- Maintains desired state
- Node Controller → monitors nodes
- Replication Controller → maintains replica count

---

## Worker Node Components

### kubelet
- Agent running on every worker node
- Registers node to cluster
- Creates and manages containers
- Reports status to API server

### kube-proxy
- Handles networking
- Enables Pod-to-Pod communication
- Implements Services

### Container Runtime
- Runs containers
- Examples: Docker, containerd, CRI-O

---

## Important Exam Points

- API Server = Heart of cluster
- etcd = Brain memory
- Scheduler = Assigns Pods
- Controllers = Maintain desired state
- kubelet = Runs containers on nodes
- kube-proxy = Networking inside cluster
- Everything talks through kube-apiserver

---

## 30-Second Revision Flow

1. User sends request to API Server
2. API Server stores state in etcd
3. Scheduler selects node
4. kubelet starts container
5. kube-proxy handles networking
6. Controllers ensure everything stays healthy
