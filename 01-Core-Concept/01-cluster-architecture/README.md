# 01 - Cluster Architecture

There are 2 types of ships: **Cargo ships and Worker ships**

Cargo ship will act as the **master node of Kubernetes**, which will manage, maintain, schedule and deploy nodes.

Worker ships are the ships on which our **containers will be running**. Pods will be scheduled here and everything is managed by **kubelet**.

So in simple words:

- Cargo Ship = Brain of Kubernetes  
- Worker Ship = Where actual work happens  

---

# 🏗️ Kubernetes Cluster Architecture Diagram

                        +----------------------+
                        |      Cargo Ship      |
                        |   (Control Plane)    |
                        +----------------------+
                        |  kube-apiserver      |
                        |  etcd                |
                        |  kube-scheduler      |
                        |  Controller Manager  |
                        +----------+-----------+
                                   |
                                   |
                     ---------------------------------
                     |                               |
         +---------------------+         +---------------------+
         |    Worker Ship 1    |         |    Worker Ship 2    |
         +---------------------+         +---------------------+
         |  kubelet            |         |  kubelet            |
         |  kube-proxy         |         |  kube-proxy         |
         |  Container Runtime  |         |  Container Runtime  |
         |  Pods (Containers)  |         |  Pods (Containers)  |
         +---------------------+         +---------------------+
---

## 🚢 Cargo Ship (Master Node)

Cargo ships have few members:

---

### 1. etcd

etcd is a **key-value store database**.

It stores all the important information about the cluster, like:

- Which container should run on which worker node  
- Cluster configuration  
- Node information  
- Pod details  

You can think of etcd as the **memory of Kubernetes**.

> Important: Only the kube-apiserver can directly communicate with etcd.

---

### 2. kube-scheduler

kube-scheduler is responsible for **loading containers on worker nodes**.

It identifies:

- The right ship (worker node)
- Capacity of the node (CPU, Memory)
- Destination of the ship (where the Pod should run)

It does not create containers.  
It only decides **where** the container should run.

---

### 3. Controller

Controller takes care of different areas inside the cluster.

It always checks whether the current state matches the desired state.

Some important controllers:

#### Node Controller
- Takes care of nodes  
- Detects damaged or unresponsive nodes  
- Takes necessary action  

#### Replication Controller
- Takes care of replicas  
- Ensures the desired number of containers are running at all times  
- If one container crashes, it creates a new one automatically  

Controllers continuously monitor the system and fix issues automatically.

---

### 4. kube-apiserver (Orchestrates all container operations)

kube-apiserver manages the whole process of the master node.

It is the **main communication hub** of Kubernetes.

Important points:

- It is the only component that can connect with etcd and get responses  
- It gives instructions to the scheduler about which node should run which container  
- It connects with kubelet on worker nodes to give instructions and receive responses  
- It connects with various controllers, monitors them and makes changes as required  

In simple words:

> Everything in Kubernetes talks through the kube-apiserver.

---

## 📦 Containers in the Cluster

Our application runs inside containers.

Even different components deployed on the master node can also run inside containers.

For example:

- DNS service  
- Networking solution  

All of these can be deployed in the form of containers inside the cluster.

---

## ⚙️ Container Runtime Engine (CRE / CRI)

To run all these containers, we require a **Container Runtime Engine**.

There are many container runtimes available, such as:

- Docker  
- Rocket  
- containerd  
- CRI-O  

The container runtime is responsible for:

- Pulling images  
- Running containers  
- Stopping containers  
- Managing container lifecycle  

Without a container runtime, containers cannot run.

---

# 🚢 Worker Ships (Worker Nodes)

Worker ships are where our actual applications run.

They have few members:

---

### 1. kubelet / Captain

kubelet is an agent that runs on each worker node of the cluster.

It works like the **captain of the worker ship**.

Responsibilities:

- It asks the master to join the cluster  
- Receives information from the kube-apiserver about container deployment  
- Gets instructions from kube-apiserver to deploy or destroy containers  
- Sends status updates back to kube-apiserver  
- Reports whether the node is healthy or if there are any issues  

kube-apiserver periodically fetches the status report from kubelet to monitor the state of nodes and containers.

So basically:

> kubelet makes sure containers are running properly on the worker node.

---

### 2. kube-proxy Service

kube-proxy handles networking between applications running on worker ships.

Suppose:

- Frontend (FE) is running in one container  
- Backend (BE) is running in another container  

If FE wants to communicate with BE, this connection is handled by kube-proxy.

It ensures:

- Containers can communicate with each other  
- Traffic is routed correctly inside the cluster  

Without kube-proxy, applications would not be able to talk to each other properly.

---


# 🔄 Complete Flow in Simple Words

1. User sends request to kube-apiserver  
2. kube-apiserver stores information in etcd  
3. Scheduler selects the correct worker ship  
4. kubelet on worker node receives instructions  
5. Container runtime runs the container  
6. kube-proxy manages communication  
7. Controllers continuously monitor and fix issues  
