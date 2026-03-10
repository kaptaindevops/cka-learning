# Kubernetes Scheduler Framework and Scheduling Process

When a Pod is created in Kubernetes, it is not immediately placed on a node. Instead, it goes through a scheduling process where the Kubernetes scheduler decides the best node to run the Pod.

The scheduler evaluates multiple conditions such as resource availability, node constraints, and Pod priorities before selecting a node.

This section explains how the scheduling process works internally and how Kubernetes allows customization of the scheduler behavior.

---

# Example Scheduling Scenario

Consider a Kubernetes cluster with four worker nodes.

A Pod is created with the following requirement:

- CPU requirement: **10 CPUs**

Each node has a different amount of available CPU resources.

The scheduler must place the Pod on a node that has **at least 10 CPUs available**.

At the same time, there may be other Pods waiting to be scheduled.

---

# Scheduling Queue

When Pods are created, they first enter the **scheduling queue**.

The scheduler processes Pods from this queue and decides where to place them.

Pods in the queue are sorted based on their **priority**.

Higher priority Pods are processed before lower priority ones.

Pod priority is defined using a **PriorityClass**.

Example:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-workload
value: 1000000
```

A Pod using this priority class will be placed earlier in the scheduling queue.

---

# Scheduling Phases

The scheduling process consists of multiple stages.

## 1. Filtering Phase

In this stage, the scheduler removes nodes that cannot run the Pod.

For example, if the Pod requires **10 CPUs**, nodes that have fewer available CPUs will be filtered out.

Other filters may also apply, such as:

- Node selectors
- Node affinity rules
- Taints and tolerations
- Node availability

Only nodes that satisfy all requirements move to the next stage.

---

## 2. Scoring Phase

After filtering, the remaining nodes are evaluated and given a score.

The scheduler calculates scores based on different criteria such as:

- Remaining resources
- Node utilization
- Image locality

The node with the **highest score** is selected for the Pod.

For example:

| Node | Remaining CPU After Scheduling | Score |
|-----|-----|-----|
| Node A | 2 CPUs | Lower score |
| Node B | 6 CPUs | Higher score |

In this case, **Node B** would be selected.

---

## 3. Binding Phase

In the final phase, the scheduler binds the Pod to the selected node.

This updates the Pod specification with the node name and the Pod begins running on that node.

---

# Scheduler Plugins

The Kubernetes scheduler uses a **plugin-based architecture**.

Each phase of the scheduling process is handled by specific plugins.

Examples include:

### PrioritySort Plugin

Used in the scheduling queue.

This plugin sorts Pods based on their priority.

---

### NodeResourcesFit Plugin

Used during filtering and scoring.

It ensures that nodes have enough CPU and memory resources for the Pod.

---

### NodeName Plugin

Checks if the Pod explicitly specifies a node name.

If a node name is defined, only that node is considered.

---

### NodeUnschedulable Plugin

Filters nodes that are marked as unschedulable.

Nodes marked with the **unschedulable flag** will not accept new Pods.

---

### ImageLocality Plugin

Used during scoring.

Nodes that already contain the required container image receive a higher score.

This reduces image download time and improves startup performance.

---

### DefaultBinder Plugin

Used during the binding phase.

This plugin performs the final step of assigning the Pod to a node.

---

# Scheduling Extension Points

The scheduler framework allows plugins to run at different stages called **extension points**.

Common extension points include:

- QueueSort
- PreFilter
- Filter
- PostFilter
- PreScore
- Score
- Reserve
- Permit
- PreBind
- Bind
- PostBind

Each extension point allows plugins to influence scheduling decisions.

For example:

| Extension Point | Purpose |
|---|---|
| QueueSort | Sort Pods in scheduling queue |
| Filter | Remove unsuitable nodes |
| Score | Rank remaining nodes |
| Bind | Attach Pod to node |

This framework allows Kubernetes scheduling to be highly customizable.

---

# Custom Scheduler Plugins

Because Kubernetes is extensible, developers can write their own scheduling plugins.

Custom plugins can be inserted into any extension point.

This allows organizations to implement advanced scheduling logic such as:

- Custom resource policies
- Business logic checks
- External system integrations
- Specialized workload placement

---

# Multiple Schedulers vs Scheduler Profiles

Previously, multiple schedulers were implemented by running **separate scheduler binaries**.

Example:

- default-scheduler
- analytics-scheduler
- custom-scheduler

Each scheduler ran independently.

However, this approach introduced some problems:

- Additional processes to manage
- Possible race conditions between schedulers
- Increased operational complexity

---

# Scheduler Profiles (Introduced in Kubernetes 1.18)

Kubernetes introduced **Scheduler Profiles** to simplify this.

Instead of running multiple scheduler binaries, a single scheduler can contain multiple profiles.

Each profile behaves like a separate scheduler.

Example configuration:

```yaml
profiles:
- schedulerName: analytics-scheduler
- schedulerName: batch-scheduler
```

Each profile can have different plugin configurations.

---

# Customizing Scheduler Profiles

Scheduler behavior can be customized by enabling or disabling plugins.

Example configuration:

```yaml
plugins:
  score:
    disabled:
      - name: TaintToleration
```

This example disables the **TaintToleration plugin** for that scheduler profile.

Custom plugins can also be added.

This allows different scheduling strategies within the same scheduler process.

---

# Summary

The Kubernetes scheduler determines where Pods run in a cluster.

The scheduling process includes several stages:

1. Pods enter the scheduling queue.
2. Nodes that cannot run the Pod are filtered.
3. Remaining nodes are scored.
4. The Pod is bound to the best node.

The scheduler uses a **plugin-based framework**, allowing customization through extension points.

Modern Kubernetes versions support **multiple scheduler profiles**, enabling different scheduling behaviors within a single scheduler instance.

This architecture makes Kubernetes scheduling powerful, flexible, and highly extensible.
