# Resource Requests and Limits in Kubernetes

In a Kubernetes cluster, every node has a limited amount of **CPU and memory**.  
When Pods run on a node, they consume these resources. The Kubernetes scheduler is responsible for deciding where a Pod should run based on the resources available on each node.

Understanding **resource requests and limits** is important to ensure that applications run reliably without starving other workloads.

---

# Example Scenario

Consider a cluster with three nodes:

- `worker-1`
- `worker-2`
- `worker-3`

Each node has a different amount of CPU and memory available.

Now suppose we deploy a Pod that runs a **video encoding service**.  
This service requires a certain amount of CPU and memory to function properly.

When the Pod is created, the scheduler evaluates:

- The resources requested by the Pod
- The resources available on each node

If `worker-2` has enough resources available, the scheduler places the Pod on that node.

However, if none of the nodes have sufficient resources, the Pod will remain in a **Pending** state.

You can verify the reason using:

```
kubectl describe pod <pod-name>
```

The events section may show a message such as **Insufficient CPU** or **Insufficient memory**.

---

# Resource Requests

Resource requests define the **minimum amount of resources a container needs** to run.

The scheduler uses these values to decide where the Pod should be placed.

Example Pod definition with resource requests:

```
apiVersion: v1
kind: Pod
metadata:
  name: video-encoder
spec:
  containers:
  - name: encoder
    image: nginx
    resources:
      requests:
        cpu: "1"
        memory: "512Mi"
```

This means the container requires:

- 1 CPU
- 512 MiB memory

The scheduler will only place this Pod on a node that has at least these resources available.

Once scheduled, the Pod is **guaranteed** these resources.

---

# CPU Units Explained

CPU values can be expressed in different ways.

Examples:

```
cpu: "1"
cpu: "500m"
cpu: "250m"
```

Explanation:

- `1` CPU = one virtual CPU core
- `500m` = 0.5 CPU
- `1000m` = 1 CPU

The `m` stands for **millicores**.

In cloud environments:

- 1 CPU usually equals one vCPU
- It may also represent a core or hyperthread depending on the platform.

---

# Memory Units Explained

Memory can also be defined using different units.

Examples:

```
memory: "256Mi"
memory: "1Gi"
```

Common units:

- Ki (Kibibyte)
- Mi (Mebibyte)
- Gi (Gibibyte)

For example:

- `1Gi` = 1024 Mi
- `512Mi` = 512 Mebibytes

---

# Resource Limits

While requests guarantee minimum resources, **limits define the maximum resources a container can consume**.

Example:

```
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
```

In this case:

- The container is guaranteed 0.5 CPU and 256Mi memory
- It can use up to 1 CPU and 512Mi memory if available

---

# What Happens When Limits Are Exceeded?

The behavior differs for CPU and memory.

### CPU Limits

If a container tries to exceed its CPU limit:

- The system **throttles CPU usage**
- The container cannot use more CPU than the limit

### Memory Limits

Memory behaves differently.

If a container uses more memory than its limit:

- Kubernetes terminates the container
- The event is recorded as **OOMKilled (Out Of Memory)**

This happens because memory cannot be throttled like CPU.

---

# Default Behavior Without Requests or Limits

If a Pod is created without specifying requests or limits:

- The container can consume as many resources as available on the node
- This may cause other workloads to suffer

Example problem:

One container might consume all available CPU or memory, leaving nothing for other applications.

---

# Common Resource Configuration Patterns

Let us consider a node running two application Pods.

### Case 1: No Requests and No Limits

- Any Pod can consume all resources
- Other Pods may starve

This setup is risky and not recommended.

---

### Case 2: Limits Without Requests

If only limits are defined:

Kubernetes automatically sets **requests equal to limits**.

Example:

```
limits:
  cpu: "2"
```

Requests will also become `2`.

This guarantees resources but may reduce scheduling flexibility.

---

### Case 3: Requests and Limits Defined

Example:

```
requests:
  cpu: "500m"
limits:
  cpu: "2"
```

This configuration guarantees 0.5 CPU but allows the container to use up to 2 CPUs if available.

This is a common setup for many workloads.

---

### Case 4: Requests Without Limits

Example:

```
requests:
  cpu: "500m"
```

This guarantees minimum resources but allows the container to use extra CPU if available.

Many production systems prefer this configuration because it allows better resource utilization.

---

# Memory Behavior Compared to CPU

CPU can be throttled when limits are exceeded.

Memory cannot be throttled.

If a container exceeds its memory limit:

- Kubernetes terminates it
- Memory is freed
- The container may restart

Because of this difference, memory limits should be configured carefully.

---

# LimitRange: Setting Default Values

Clusters often enforce default resource policies using **LimitRange**.

A LimitRange defines default requests and limits for Pods created within a namespace.

Example:

```
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    max:
      cpu: "1"
    min:
      cpu: "100m"
    type: Container
```

This ensures:

- Containers receive default resource values
- Maximum and minimum values are enforced

LimitRanges apply only to **new Pods created after the rule is applied**.

---

# ResourceQuota: Controlling Namespace Usage

Sometimes you may want to limit the total resources consumed within a namespace.

This can be done using **ResourceQuota**.

Example:

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "6"
    requests.memory: 8Gi
    limits.cpu: "12"
    limits.memory: 16Gi
```

This ensures that all Pods within the namespace collectively cannot exceed these limits.

ResourceQuota helps control resource usage across teams and applications.

---

# Summary

- Kubernetes schedules Pods based on available node resources.
- **Resource requests** define the minimum resources required.
- **Resource limits** define the maximum resources a container can use.
- CPU exceeding limits is throttled.
- Memory exceeding limits results in container termination.
- Without requests or limits, Pods may consume all node resources.
- **LimitRange** sets default values for containers in a namespace.
- **ResourceQuota** limits total resource usage within a namespace.

Proper resource management ensures stable workloads and efficient cluster utilization.
```
