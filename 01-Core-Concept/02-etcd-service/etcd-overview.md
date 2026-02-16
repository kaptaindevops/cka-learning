# etcd Role in Kubernetes

## What is etcd?
**etcd** is a distributed, reliable key-value datastore used by Kubernetes to store all cluster-related information. It is the **database and single source of truth** for the Kubernetes cluster.

### It stores information such as:
- **Nodes** & **Pods**
- **Deployments** & **ReplicaSets**
- **ConfigMaps** & **Secrets**
- **Service Accounts**
- **Roles** & **Role Bindings**
- Other cluster configurations

Whenever we run commands like the following, the data is retrieved directly from the **etcd server**:

```bash
kubectl get pods
kubectl get nodes
kubectl get deployments

```

---

## Why etcd is Important

Every change in the cluster is first updated in etcd. Only once the change is successfully updated in etcd is the operation considered complete.

**Examples of changes:**

* Adding a new node.
* Deploying a new pod.
* Creating a ReplicaSet.
* Updating a deployment or scaling replicas.

> **Note:** If the data is not stored in etcd, Kubernetes does not consider the change successful.

---

## How etcd is Deployed

The way etcd is deployed depends on how the Kubernetes cluster is set up.

### 1. Manual Cluster Setup (From Scratch)

If you set up the cluster manually (e.g., Kubernetes The Hard Way):

* **Process:** Download etcd binaries  Configure as a service on the master node  Start manually.
* **Important Parameter:** `--advertise-client-urls https://<endpoint>:2379`
* This defines the address on which etcd listens for client requests.
* This same URL must be configured in the **kube-apiserver** so that it can connect to etcd.



### 2. kubeadm Setup

If you set up the cluster using `kubeadm`:

* etcd is deployed automatically.
* It runs as a **Static Pod** in the `kube-system` namespace.
* You can verify the etcd Pod using:
```bash
kubectl get pods -n kube-system

```



---

## etcd in High Availability (HA) Environment

In an HA setup, there are multiple master (control plane) nodes.

* Each master node may run its own etcd instance.
* These instances form an **etcd cluster**.
* All etcd servers must "know" about each other to communicate and replicate data.

**Important Parameter:**

* `--initial-cluster`: This option is used to define all the etcd instances that are part of the cluster.

**This ensures:**

1. Data is replicated between etcd nodes.
2. The cluster remains available even if one etcd node fails.

---

## Quick Summary

* **Role:** The database of Kubernetes; stores the complete state.
* **Retrieval:** All `kubectl get` commands pull data from etcd.
* **Consistency:** Changes are only "complete" once stored in etcd.
* **Manual Setup:** Installed using binaries as a system service.
* **Kubeadm Setup:** Runs as a Static Pod.
* **HA Setup:** Multiple instances form a cluster using the `--initial-cluster` flag.
