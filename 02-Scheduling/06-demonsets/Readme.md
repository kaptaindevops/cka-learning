# DaemonSets in Kubernetes

In many Kubernetes workloads, you may want to run multiple copies of an application across different worker nodes. Tools like ReplicaSets help achieve this by maintaining a specified number of Pod replicas.

However, some use cases require **exactly one Pod running on every node in the cluster**. This is where **DaemonSets** come in.

A DaemonSet ensures that a single copy of a Pod runs on each node. If a new node joins the cluster, the DaemonSet automatically schedules the Pod on that node. If a node is removed, the corresponding Pod is also removed automatically.

This behavior guarantees that the required Pod always runs on every node.

---

# How DaemonSets Work

A DaemonSet automatically manages Pod placement across nodes.

Its behavior can be summarized as follows:

- One Pod instance runs on each node.
- When a new node is added to the cluster, a Pod is automatically created on that node.
- When a node is removed, the Pod running on that node is deleted.

This ensures consistent deployment of certain workloads across the entire cluster.

---

# Common Use Cases for DaemonSets

DaemonSets are commonly used for system-level or node-level services that must run on every node.

### Log Collection

Many clusters run log collection agents on each node to gather application and system logs.

For example, a logging agent can be deployed as a DaemonSet so that every node automatically runs the log collector.

---

### Monitoring Agents

Monitoring tools often require an agent running on every node to collect metrics such as CPU usage, memory consumption, and network activity.

Deploying the monitoring agent as a DaemonSet ensures that each node is continuously monitored.

---

### Kubernetes Networking Components

Some networking solutions require a node-level component to manage networking behavior.

For example, networking plugins often deploy a Pod on every node to handle networking rules and traffic management.

---

### Node-Level System Services

Some components must run on every node to support cluster operations.

For example, system utilities responsible for node-level maintenance or security scanning are often deployed using DaemonSets.

---

# DaemonSet Definition Structure

Creating a DaemonSet is similar to creating a ReplicaSet or Deployment.

A typical DaemonSet definition includes:

- `apiVersion`
- `kind`
- `metadata`
- `spec`

Example structure:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-monitor
spec:
  selector:
    matchLabels:
      app: monitor-agent
  template:
    metadata:
      labels:
        app: monitor-agent
    spec:
      containers:
      - name: monitor-agent
        image: nginx
```

Important points:

- The `selector` identifies which Pods belong to the DaemonSet.
- The `template` defines the Pod specification.
- Labels in the selector must match the labels in the Pod template.

---

# Creating a DaemonSet

To create the DaemonSet, run:

```bash
kubectl apply -f daemonset.yaml
```

To view the DaemonSets in the cluster:

```bash
kubectl get daemonsets
```

To see more details about a DaemonSet:

```bash
kubectl describe daemonset node-monitor
```

These commands help verify that the DaemonSet is running on all nodes.

---

# How Pods Are Scheduled in a DaemonSet

In earlier Kubernetes versions, DaemonSets used a different scheduling mechanism.

Each Pod was assigned to a node by explicitly setting the `nodeName` field in the Pod specification.

However, starting from newer Kubernetes versions, DaemonSets rely on the **default scheduler** along with **node affinity rules**.

This allows the scheduler to handle placement while ensuring that each node receives exactly one Pod instance.

---

# Summary

- A DaemonSet ensures that a Pod runs on every node in the cluster.
- When nodes are added or removed, Pods are automatically created or deleted.
- DaemonSets are commonly used for logging agents, monitoring tools, networking components, and node-level services.
- Their configuration is similar to ReplicaSets, with a Pod template and selector.
- Modern Kubernetes versions use the default scheduler with node affinity to place DaemonSet Pods.

DaemonSets are an important mechanism for running node-specific workloads consistently across a Kubernetes cluster.
