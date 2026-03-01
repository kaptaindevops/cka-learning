# Kubernetes Scheduling – Understanding How Pods Are Placed

In this section, we will explore scheduling in more depth and understand how different configurations influence where and how Pods are placed in the cluster.

The goal here is to clearly understand how scheduling decisions are made and how we can control them when needed.

---

# Topics Covered

In this section, we will cover:

1. Manual Scheduling  
2. DaemonSets  
3. Labels and Node Selection  
4. Resource Requests and Limits  
5. Multiple Schedulers  
6. Inspecting Scheduler Events  

Each of these directly affects how workloads are distributed across Nodes.

---

# 1. Manual Scheduling

Normally, the Kubernetes Scheduler automatically selects a suitable Node for a Pod.

However, Kubernetes allows you to manually assign a Pod to a specific Node by defining the `nodeName` field inside the Pod specification.

In this approach:

- The Scheduler is bypassed.
- The Pod is directly assigned to the specified Node.
- No scheduling decision is made by the default scheduler.

This is useful in scenarios such as:

- Testing specific Nodes
- Debugging scheduling behavior
- Running workloads tied to specific hardware

Manual scheduling gives you direct control, but it should be used carefully in production environments.

---

# 2. DaemonSets

A DaemonSet ensures that a particular Pod runs on every Node (or selected Nodes) in the cluster.

For example, consider a security scanning agent that must run on every Node to monitor system activity.

With a DaemonSet:

- A copy of the Pod is automatically created on every Node.
- When a new Node joins the cluster, the Pod is automatically scheduled on it.
- When a Node is removed, the corresponding Pod is also removed.

DaemonSets are commonly used for:

- Logging agents
- Monitoring tools
- Security agents
- Storage drivers

They ensure consistent coverage across the cluster.

---

# 3. Labels and Node Selection

Nodes can be labeled to describe their characteristics.

For example:

- hardware=high-memory
- zone=zone-a
- workload=batch

When creating a Pod, you can use `nodeSelector` to ensure the Pod is scheduled only on Nodes that match specific labels.

Example scenario:

If certain Nodes are optimized for compute-intensive workloads, you can label them accordingly and configure Pods to run only on those Nodes.

This allows better workload placement and resource utilization.

Labels are a powerful way to influence scheduling behavior without manually assigning Nodes.

---

# 4. Resource Requests and Limits

Resource requests and limits directly impact scheduling decisions.

When defining a Pod, you can specify:

- CPU request
- Memory request
- CPU limit
- Memory limit

The Scheduler looks at the resource requests (not limits) to determine if a Node has enough available capacity.

If no Node has sufficient resources:

- The Pod remains in the Pending state.
- It waits until resources become available.

Requests determine placement.  
Limits control how much the container can consume at runtime.

Proper resource configuration ensures efficient cluster utilization and prevents overloading Nodes.

---

# 5. Multiple Schedulers

Kubernetes supports running more than one scheduler.

This is useful when:

- Custom scheduling logic is required.
- Different teams need different scheduling strategies.
- Specialized workloads need unique placement rules.

You can deploy a custom scheduler and configure certain Pods to use it instead of the default scheduler.

This provides flexibility for advanced use cases, especially in large-scale or enterprise environments.

---

# 6. Viewing Scheduler Events

When a Pod is not scheduled as expected, you need to investigate why.

To check scheduling details:

```
kubectl describe pod <pod-name>
```

This command shows:

- Node selection information
- Scheduling failures
- Resource constraint messages

To view cluster-wide events:

```
kubectl get events
```

This helps in identifying scheduling issues such as insufficient CPU, memory, or label mismatches.

---

# Summary

The Kubernetes Scheduler is responsible for assigning Pods to appropriate Nodes.

You can influence scheduling using:

- Manual node assignment
- DaemonSets
- Labels and node selectors
- Resource requests and limits
- Custom schedulers

Understanding scheduling behavior is critical for:

- Optimizing performance
- Efficient resource usage
- Troubleshooting Pending Pods
- Designing scalable and production-ready systems

A solid understanding of scheduling ensures better cluster management and stronger Kubernetes fundamentals.
