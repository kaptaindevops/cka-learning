# Kube-Proxy in Kubernetes

## Pod-to-Pod Communication in Kubernetes

In a Kubernetes cluster, every Pod can communicate with every other Pod.  
This is possible because of the **Pod networking model**, where each Pod gets its own IP address and all Pods are part of a flat network.

---

## Example Scenario

Suppose:

- A **Web Application** is running in one Pod.
- A **Database (DB)** is running in another Pod.

Now the Web App needs to communicate with the DB Pod.

How is this achieved?

---

## Role of Service

To allow stable communication:

1. We create a **Service** for the DB Pod.
2. The Service exposes the DB application.
3. The Web App accesses the DB using:
   - Service IP
   - Service DNS name

When a Pod sends traffic to the Service IP or name:
- The traffic is forwarded to one of the backend Pods (in this case, the DB Pod).

---

## Does a Service Join the Pod Network?

No.

A Service is **not an actual object like a Pod**.

- It does not have a network interface.
- It does not run a process.
- It is a virtual component stored in Kubernetes memory (API Server / etcd).

The Service is simply an abstraction that defines:
- A virtual IP (ClusterIP)
- A set of backend Pods (via selectors)

---

## How Is the Service Accessible Across All Nodes?

The Service must be reachable from:

- Any Pod
- On any Node
- Across the entire cluster

This is where **kube-proxy** comes in.

---

## What Is Kube-Proxy?

`kube-proxy` is a process that runs on **each node** in the cluster.

### Responsibilities:

- Watches for new Services created in the cluster.
- Detects changes to Services and Endpoints.
- Creates networking rules on the node.
- Forwards traffic from Service IP to backend Pods.

---

## How Does Kube-Proxy Forward Traffic?

### Using iptables Rules

When a Service is created:

- kube-proxy creates **iptables rules** on each node.
- These rules:
  - Capture traffic sent to the Service IP.
  - Redirect it to one of the backend Pods.
  - Perform load balancing between Pods.

So when the Web App sends traffic to the Service IP:
- iptables rules forward it to the DB Pod.

---

## Installation of Kube-Proxy

### Manual Installation

1. Download the kube-proxy binary from the official Kubernetes release page.
2. Extract the binary.
3. Run it as a service on each node.

---

### Using kubeadm

If you set up the cluster using `kubeadm`:

- kube-proxy is automatically deployed.
- It runs as a Pod on each node (usually as a DaemonSet).
- No manual installation is required.

---

## Summary

- Every Pod in Kubernetes can communicate with other Pods.
- Services provide a stable way to expose backend Pods.
- Services are virtual objects (not real network entities).
- kube-proxy runs on each node.
- It creates iptables rules to forward traffic to backend Pods.
- kubeadm automatically deploys kube-proxy.

---
