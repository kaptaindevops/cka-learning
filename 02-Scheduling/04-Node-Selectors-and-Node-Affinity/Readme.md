# Node Selectors and Node Affinity in Kubernetes

In many Kubernetes environments, not all nodes have the same hardware capacity. Some nodes may have higher CPU, memory, or specialized hardware such as GPUs.

In such cases, certain workloads should run only on specific nodes that can support them.

Let us understand how Kubernetes allows us to control where Pods are scheduled.

---

# Example Scenario

Consider a Kubernetes cluster with three worker nodes:

- `node-a`
- `node-b`
- `node-c`

Assume that:

- `node-a` and `node-b` are standard nodes with moderate resources.
- `node-c` is a high-performance node with larger CPU and memory capacity.

Now imagine that you have a **machine learning workload** that requires significant compute resources.

You would prefer this workload to run on `node-c`, since the other nodes may not have enough capacity.

However, in the default configuration, the Kubernetes scheduler can place Pods on **any node** in the cluster.

This means the machine learning Pod might get scheduled on `node-a` or `node-b`, which is not ideal.

To control this behavior, Kubernetes provides mechanisms such as:

- Node Selectors
- Node Affinity

---

# Using Node Selectors

The simplest way to control Pod placement is using **node selectors**.

A node selector allows you to schedule a Pod on nodes that match specific labels.

---

# Step 1: Label the Node

First, you must label the node.

Example:

```
kubectl label nodes node-c workload=ml
```

This assigns a label to `node-c` indicating that it is suitable for machine learning workloads.

---

# Step 2: Use Node Selector in Pod Definition

Now update the Pod specification to include the node selector.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: ml-training-pod
spec:
  containers:
    - name: trainer
      image: nginx
  nodeSelector:
    workload: ml
```

When the Pod is created, the scheduler will only place it on nodes that have the label:

```
workload=ml
```

In this case, the Pod will be scheduled on `node-c`.

---

# Limitation of Node Selectors

Node selectors are simple and easy to use, but they have limitations.

They only support **exact matches**.

For example, you cannot express conditions like:

- Run on nodes with `gpu` **or** `high-memory`
- Avoid nodes labeled `testing`
- Run on nodes where a label exists

To support these advanced scheduling rules, Kubernetes introduced **node affinity**.

---

# Node Affinity

Node affinity provides a more flexible way to control Pod placement.

It allows you to define advanced rules for selecting nodes.

The basic structure looks like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: ml-training-pod
spec:
  containers:
    - name: trainer
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - ml
```

This configuration ensures that the Pod runs on nodes where the label:

```
workload=ml
```

exists.

---

# Using Multiple Values

Suppose your Pod can run on nodes labeled either `ml` or `gpu`.

You can specify multiple values:

```
values:
- ml
- gpu
```

This means the Pod can run on nodes labeled either of those values.

---

# Using the NotIn Operator

If you want to avoid certain nodes, you can use the `NotIn` operator.

Example:

```
operator: NotIn
values:
- testing
```

This tells the scheduler not to place the Pod on nodes labeled as testing.

---

# Using the Exists Operator

You can also schedule Pods on nodes where a label simply exists.

Example:

```
operator: Exists
```

This means the scheduler only checks whether the label is present, without evaluating its value.

---

# Node Affinity Types

Node affinity rules include two important behaviors that determine how strictly the scheduler follows them.

---

## 1. Required During Scheduling

```
requiredDuringSchedulingIgnoredDuringExecution
```

This means:

- The Pod **must** be scheduled on nodes that satisfy the rule.
- If no matching node exists, the Pod will remain in the **Pending** state.

This is used when the placement requirement is critical.

---

## 2. Preferred During Scheduling

```
preferredDuringSchedulingIgnoredDuringExecution
```

This means:

- The scheduler **tries** to place the Pod on matching nodes.
- If none are available, the Pod can still run on other nodes.

This approach is useful when the preference is important but not mandatory.

---

# During Execution Behavior

Both current node affinity types include the phrase:

```
IgnoredDuringExecution
```

This means that once a Pod is scheduled and running, changes to node labels will **not affect the running Pod**.

For example:

- A Pod is scheduled on `node-c`.
- Later, an administrator removes the label `workload=ml`.

The Pod will continue running on that node.

It will not be automatically moved or evicted.

---

# Summary

- Kubernetes schedules Pods on nodes automatically by default.
- Node selectors allow simple label-based scheduling.
- Nodes must be labeled before using node selectors.
- Node affinity provides advanced scheduling rules.
- Operators such as `In`, `NotIn`, and `Exists` enable flexible placement.
- Required affinity enforces strict placement.
- Preferred affinity allows flexible scheduling.
- Changes to node labels do not affect running Pods when using the current affinity types.

Node selectors and node affinity are essential tools for controlling workload placement in Kubernetes clusters.
