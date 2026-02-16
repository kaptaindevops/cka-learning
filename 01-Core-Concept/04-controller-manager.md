# Kubernetes Controllers

## What is a Controller in Kubernetes?

A Controller is a control loop that continuously monitors the current state of the Kubernetes cluster and compares it with the desired state defined in Kubernetes objects.

If the current state does not match the desired state, the controller takes corrective action to bring the cluster back to the desired state.

This reconciliation mechanism is one of the core design principles of Kubernetes and enables self-healing behavior.

---

## How Controllers Work

Kubernetes follows a declarative model:

1. The user defines the desired state (for example, 3 replicas of a Pod).
2. The controller observes the actual state of the cluster.
3. If there is any difference between actual and desired state, it takes action to reconcile the difference.

This continuous reconciliation loop ensures that the system remains stable and consistent.

---

# Types of Controllers

Kubernetes includes many built-in controllers. Below are some of the most important ones.

---

## 1. Node Controller

The Node Controller is responsible for monitoring the health of worker nodes in the cluster.

### Responsibilities

- Monitors the status of all nodes.
- Detects when a node becomes unreachable.
- Waits for approximately 40 seconds before marking a node as `NotReady`.
- If the node remains unreachable, Pods running on that node are evicted.
- If those Pods are managed by a ReplicaSet, Deployment, or StatefulSet, they are recreated on healthy nodes.

### Example Scenario

If a worker node crashes:

- After the grace period, it is marked as `NotReady`.
- Pods scheduled on that node are evicted.
- ReplicaSet or Deployment controllers create replacement Pods on healthy nodes.

This ensures high availability and fault tolerance.

---

## 2. Replication Controller

The Replication Controller ensures that a specified number of Pod replicas are running at all times.

If:

- A Pod crashes → It creates a new Pod.
- More Pods than required exist → It deletes the extra Pods.

This controller is considered legacy and has largely been replaced by ReplicaSet.

---

## 3. ReplicaSet Controller

ReplicaSet is the modern replacement for ReplicationController and is commonly used by Deployments.

Example specification:

```yaml
spec:
  replicas: 3
```

---
# Kubernetes Controllers and the kube-controller-manager

## The ReplicaSet Controller

The **ReplicaSet** is a primary example of the "reconciliation loop" in action. It ensures that the cluster matches your desired state (exactly 3 Pods running).

* **If replicas drop below 3:** New Pods are automatically created.
* **If replicas exceed 3:** Extra Pods are terminated.
* **Deployments:** Use ReplicaSets internally to manage rolling updates and scaling seamlessly.

---

## The kube-controller-manager

In Kubernetes, controllers are not independent entities floating around; they are grouped into a single control plane component called the **kube-controller-manager**. This "manager of managers" runs multiple controller processes in a single binary to reduce complexity.

### Key Built-in Controllers:

* **Node Controller:** Manages node health and availability.
* **ReplicaSet Controller:** Maintains the pod count.
* **Deployment Controller:** Handles rolling updates.
* **StatefulSet Controller:** Manages stateful applications.
* **Job Controller:** Runs ephemeral tasks to completion.
* **Endpoint / Namespace / Service Account Controllers:** Manage internal resources and security.

---

## How to View Controllers in a Cluster

Controllers are not exposed as separate services. Because they run inside the `kube-controller-manager`, you must inspect that specific component.

### 1. In kubeadm-based Clusters

The controller manager usually runs as a **static Pod** in the `kube-system` namespace.

**Find the Pod:**

```yaml
kubectl get pods -n kube-system | grep controller

```

**Inspect the configuration:**

```yaml
kubectl describe pod kube-controller-manager-master -n kube-system

```

### 2. Static Pod Manifest Location

If using `kubeadm`, the source of truth for how the controller manager is configured is the manifest file located at:
` /etc/kubernetes/manifests/kube-controller-manager.yaml`

### 3. Checking the Running Process

If you have SSH access to the control plane node, you can see the actual process and the flags (which controllers are enabled/disabled) by running:

```yaml
ps -aux | grep kube-controller-manager

```

---

## Summary Table

| Concept | Description |
| --- | --- |
| **Control Loop** | The continuous cycle of comparing **actual state** vs. **desired state**. |
| **ReplicaSet** | Ensures the specified number of pod replicas are running at all times. |
| **kube-controller-manager** | The control plane process that bundles all built-in controllers. |
| **Static Pod** | How the controller manager is typically deployed (managed by the Kubelet). |
