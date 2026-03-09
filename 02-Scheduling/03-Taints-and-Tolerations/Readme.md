# Taints and Tolerations in Kubernetes

For many beginners, the concept of **taints and tolerations** can be difficult to understand at first.  
To simplify the idea, let us start with a simple analogy.

---

# Understanding the Concept with a Simple Example

Imagine a parking area that has multiple parking zones.

One particular zone is reserved only for **electric vehicles**.

To enforce this rule, the parking authority places a restriction sign that says:

> Only electric vehicles allowed.

Regular vehicles cannot park there because they do not meet the requirement.

However, electric vehicles are allowed because they satisfy the rule.

In this example:

- The restriction sign represents a **taint**.
- The vehicle's capability to park there represents a **toleration**.

Only vehicles that satisfy the requirement can enter that zone.

---

# Mapping the Example to Kubernetes

In Kubernetes:

- Parking zone → **Node**
- Vehicles → **Pods**
- Restriction sign → **Taint**
- Ability to park there → **Toleration**

Taints are applied to **Nodes**.  
Tolerations are defined in **Pods**.

They control which Pods are allowed to run on certain Nodes.

---

# Purpose of Taints and Tolerations

Taints and tolerations are used to:

- Restrict Pods from running on specific Nodes
- Reserve Nodes for special workloads
- Control workload distribution

Important point:

They are **not used for security**.  
They are only used for **scheduling control**.

---

# Basic Scheduling Without Taints

Imagine a cluster with three worker nodes:

- node-a
- node-b
- node-c

You create several Pods:

- service-pod
- analytics-pod
- cache-pod
- batch-pod

Without any restrictions, the Kubernetes Scheduler distributes Pods across all nodes to balance the load.

---

# Adding a Taint to a Node

Now assume that **node-a** should be reserved for a special data processing workload.

To prevent other Pods from running there, we apply a taint to node-a.

Example command:

```
kubectl taint nodes node-a workload=data-processing:NoSchedule
```

This taint means:

- Nodes with this taint will reject Pods unless those Pods explicitly tolerate it.

Since most Pods do not have tolerations by default, they will not be scheduled on node-a.

---

# Allowing Specific Pods Using Tolerations

Now suppose the **batch-pod** is part of the data processing workload and should be allowed to run on node-a.

We add a toleration to its Pod definition.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: batch-pod
spec:
  tolerations:
  - key: "workload"
    operator: "Equal"
    value: "data-processing"
    effect: "NoSchedule"
  containers:
  - name: batch-container
    image: nginx
```

Now the scheduler sees:

- node-a has a taint
- batch-pod has a matching toleration

So batch-pod can run on node-a.

Other Pods without tolerations will be scheduled on node-b or node-c.

---

# Taint Effects

A taint includes a **key**, **value**, and **effect**.

Example:

```
workload=data-processing:NoSchedule
```

The effect determines how Kubernetes behaves.

---

## 1. NoSchedule

Pods that do not tolerate the taint will **not be scheduled** on that node.

Example scenario:

A GPU node is reserved for machine learning workloads.

Only Pods with proper tolerations can run there.

---

## 2. PreferNoSchedule

The scheduler will **try to avoid** placing Pods on the node.

However, it is not strictly enforced.

If no other nodes are available, Pods may still be scheduled there.

---

## 3. NoExecute

This effect does two things:

1. Prevents new Pods from being scheduled.
2. Removes existing Pods that do not tolerate the taint.

Example:

If a node becomes unhealthy, a NoExecute taint can force incompatible Pods to be evicted.

Pods with matching tolerations can continue running.

---

# Important Behavior to Remember

Taints and tolerations **do not force a Pod onto a specific node**.

They only **prevent Pods from running on certain nodes**.

For example:

If node-a allows only a specific workload, that workload can still run on other nodes unless additional restrictions are applied.

To force Pods onto certain nodes, you should use **node affinity** or **node selectors**.

---

# Default Taint on Control Plane Nodes

In most Kubernetes clusters, the control plane node (previously called the master node) has a default taint.

This prevents regular application Pods from being scheduled there.

This ensures that control plane components remain stable and are not affected by user workloads.

You can view this taint using:

```
kubectl describe node <control-plane-node-name>
```

Look for the **Taints** section in the output.

Although this taint can be removed, it is considered best practice to keep application workloads away from control plane nodes.

---

# Summary

- Taints are applied to Nodes.
- Tolerations are applied to Pods.
- Taints prevent unwanted Pods from being scheduled on specific nodes.
- Pods must have matching tolerations to run on tainted nodes.
- Three taint effects exist: NoSchedule, PreferNoSchedule, and NoExecute.
- Taints restrict scheduling but do not force Pods onto nodes.
- Node affinity is used when you want Pods to run on specific nodes.

Understanding taints and tolerations is important for controlling workload placement and maintaining clean cluster architecture.
