# Priority Classes in Kubernetes

In a Kubernetes cluster, different applications may have different levels of importance. Some workloads are critical and must always run, while others can wait if resources are limited.

For example, a cluster may run:

- Core infrastructure components
- Databases or critical business services
- Background processing jobs
- Batch workloads

In such environments, Kubernetes needs a way to ensure that **important workloads get scheduled before less critical ones**. This is where **Priority Classes** are used.

---

# What Are Priority Classes?

Priority Classes allow you to assign a priority value to Pods.

The scheduler uses these values to determine:

- Which Pods should be scheduled first
- Which Pods can be evicted if resources are limited

Pods with **higher priority values are scheduled before Pods with lower priority values**.

If the cluster does not have enough resources to run a high-priority Pod, Kubernetes may terminate lower-priority Pods to free resources.

---

# Priority Class Characteristics

Priority Classes are **non-namespaced objects**.

This means:

- They are created at the cluster level.
- They can be used by Pods in any namespace.

Once defined, they are available across the entire cluster.

---

# Priority Value Range

Priority values are defined using integers.

Typical ranges include:

- Application workloads: roughly **-2,000,000,000 to 1,000,000,000**
- System-level workloads: reserved values close to **2,000,000,000**

Higher numbers indicate higher priority.

Kubernetes reserves the highest range for internal system components so they always receive scheduling priority.

---

# Default System Priority Classes

You can view existing priority classes using:

```bash
kubectl get priorityclass
```

Most clusters include built-in classes such as:

- `system-cluster-critical`
- `system-node-critical`

These are used by important Kubernetes components to ensure they always get priority over regular workloads.

---

# Creating a Priority Class

A new Priority Class is created using a definition file.

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: important-service
value: 1000
description: "Priority for important services"
```

Explanation:

- `value` defines the priority level.
- Higher values indicate higher priority.
- `description` is optional and used for documentation.

Once this object is created, Pods can reference it.

---

# Assigning a Priority Class to a Pod

To use a Priority Class, specify its name in the Pod specification.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-service
spec:
  priorityClassName: important-service
  containers:
  - name: payment-container
    image: nginx
```

When this Pod is created, it inherits the priority defined in the Priority Class.

---

# Default Pod Priority

If a Pod does not specify a Priority Class, Kubernetes assigns a default priority value of **0**.

If you want to change the default priority for all Pods, you can create a Priority Class with the `globalDefault` option.

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: default-priority
value: 100
globalDefault: true
description: "Default priority for all Pods"
```

Important notes:

- Only **one Priority Class** can have `globalDefault: true`.
- This ensures there is only one default priority level.

---

# Pod Scheduling with Priorities

Consider two workloads entering the scheduler:

- `payment-service` with priority **900**
- `batch-processing` with priority **400**

If resources are available, both Pods will be scheduled.

However, if resources are limited:

- The higher priority Pod (`payment-service`) will be scheduled first.
- The lower priority Pod may wait.

---

# Preemption

If a high-priority Pod cannot be scheduled because the cluster is full, Kubernetes may **preempt** lower-priority Pods.

Preemption means:

- Lower-priority Pods are terminated.
- Resources are freed.
- The higher-priority Pod is scheduled.

This behavior ensures critical workloads can always run.

---

# Controlling Preemption Behavior

Preemption behavior is controlled using the **preemptionPolicy** field.

By default:

```
preemptionPolicy: PreemptLowerPriority
```

This allows higher-priority Pods to evict lower-priority ones.

If you want the Pod to wait instead of evicting other workloads, use:

```
preemptionPolicy: Never
```

Example Priority Class:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: reporting-jobs
value: 500
preemptionPolicy: Never
description: "Priority for reporting workloads"
```

In this case:

- The Pod still has a higher priority during scheduling.
- But it will **not terminate other Pods**.

Instead, it waits for resources to become available.

---

# Summary

- Priority Classes define the importance of workloads.
- Higher priority Pods are scheduled before lower priority Pods.
- If resources are limited, Kubernetes may evict lower priority Pods.
- Priority Classes are cluster-wide objects.
- Pods reference them using `priorityClassName`.
- If no priority is defined, Pods receive a default priority of **0**.
- Preemption behavior can be controlled using `preemptionPolicy`.

Priority Classes help ensure that critical applications always receive the resources they need in a busy Kubernetes cluster.
