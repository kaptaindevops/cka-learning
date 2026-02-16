# Kube-API Server

## Introduction

The **Kube-API Server** is the primary service of Kubernetes and acts as the main entry point to the cluster. Whenever you perform an action using `kubectl` or an API request, the request first hits this component.

**The Kube-API Server:**

* **Authenticates** the user.
* **Validates** the request.
* **Retrieves or updates** data in the **etcd** cluster.

In simple terms, it is the central communication hub of Kubernetes.

---

## Example: Deploying a Pod

Suppose a user wants to deploy a Pod using the following command:

```bash
kubectl create pod nginx --image=nginx

```

### Step-by-Step Flow

1. **Request:** The command triggers an API request to the Kube-API Server.
2. **Processing:** The Kube-API Server authenticates the user and validates the request.


### Scheduler Interaction

1. The **Scheduler** continuously monitors the Kube-API Server for newly created Pods with no assigned node.
2. The Scheduler identifies the right node for the Pod and sends this decision back to the Kube-API Server.
3. The Kube-API Server:
* Updates the information in **etcd**.
* Passes the updated information to the **kubelet** on the selected node.



### Kubelet Interaction

1. The **kubelet** on the worker node receives instructions from the Kube-API Server.
2. It instructs the **container runtime engine** to deploy the Pod.
3. Once the Pod is running, the kubelet sends the status back to the Kube-API Server.
4. The Kube-API Server updates the Pod status in the **etcd** server.

---

## Overall Communication Pattern

All components communicate exclusively through the Kube-API Server:

| Source | Direction | Destination |
| --- | --- | --- |
| **User** | → | Kube-API Server → etcd |
| **Scheduler** | → | Kube-API Server → etcd |
| **Kubelet** | → | Kube-API Server → etcd |

---

## Installing Kube-API Server (The Hard Way)

If setting up Kubernetes manually, the binary is available on the official Kubernetes release page. It runs with many parameters, including:

* **Kubelet Certificates:** Secures communication with worker nodes.
* **Etcd Certificates:** Secures communication with the data store.
* **Networking/Authorization:** Parameters for cluster security and traffic.

---

## How to View Kube-API Server Options

The method depends on your installation type:

### 1. If setup using `kubeadm`

`kubeadm` deploys the Kube-API Server as a **Static Pod**.

* **View the Pod:**
```bash
kubectl get pods -n kube-system

```


* **See detailed configuration:**
```bash
kubectl describe pod kube-apiserver-<node-name> -n kube-system

```



### 2. If setup manually (Systemd)

If the cluster was set up without `kubeadm`, inspect the service file:

* **Path:** `/etc/systemd/system/kube-apiserver.service`

### 3. Check the Running Process

To see the process and its active parameters:

```bash
ps -aux | grep kube-apiserver

```

---

## Quick Summary

* **Entry Point:** The main gateway for all cluster requests.
* **Gatekeeper:** Handles authentication and validation.
* **Etcd Access:** The *only* component that communicates directly with etcd.
* **Coordination:** Orchestrates interactions between the Scheduler and kubelet.
* **Deployment:** Runs as a Pod (kubeadm) or a systemd service (manual).

---
