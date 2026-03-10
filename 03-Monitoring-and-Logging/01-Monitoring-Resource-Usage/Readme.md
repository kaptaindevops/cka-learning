# Monitoring Resource Usage in Kubernetes

Monitoring is an essential part of managing a Kubernetes cluster. It helps administrators understand how resources are being used and whether the cluster is operating efficiently. When monitoring Kubernetes, we typically want visibility into both **node-level metrics** and **pod-level metrics**.

## Metric Types

### Node-Level Metrics

At the node level, common metrics to monitor include:

* Total number of nodes in the cluster
* Health status of each node
* CPU and Memory usage
* Network and Disk utilization

### Pod-Level Metrics

At the application level, we monitor Pods to track:

* Total number of Pods running
* CPU consumption per Pod
* Memory consumption per Pod

---

## Monitoring Solutions

Kubernetes does not include a complete built-in long-term monitoring platform. Instead, it relies on external tools to collect, store, and analyze metrics.

| Solution | Use Case |
| --- | --- |
| **Metrics Server** | Real-time resource usage & Autoscaling (HPA) |
| **Prometheus** | Industry standard for time-series data & alerting |
| **Elastic Stack** | Log aggregation and analysis |
| **Datadog / Dynatrace** | Managed SaaS APM and infrastructure monitoring |

> **Note:** Earlier Kubernetes environments used **Heapster**, but this project has been deprecated and replaced by the **Metrics Server**.

---

## The Metrics Server

The **Metrics Server** is the primary "source of truth" for resource metrics within a cluster.

### Key Characteristics:

* **One per cluster:** Only one instance runs per cluster.
* **In-Memory Storage:** It does **not** maintain historical data.
* **Purpose:** Designed for real-time monitoring and enabling the **Horizontal Pod Autoscaler (HPA)**.

### How Metrics Are Collected

The Metrics Server aggregates data from every node via the **Kubelet**. Inside every Kubelet is a component called **cAdvisor** (Container Advisor), which is responsible for reaching into the container runtime to pull stats.

1. **cAdvisor:** Collects CPU, memory, network, and disk stats from containers.
2. **Kubelet:** Exposes these statistics through its API.
3. **Metrics Server:** Polls the Kubelet API on each node to aggregate the data.

---

## Installation and Usage

### Installing Metrics Server

**On Minikube:**

```bash
minikube addons enable metrics-server

```

**On standard Kubernetes clusters:**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

```

### Viewing Resource Metrics

Once installed, you can use the `kubectl top` command to view resource consumption in real-time.

#### 1. Node Metrics

```bash
kubectl top node

```

**Example Output:**

```text
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node-1       166m         8%     1024Mi          25%
node-2       120m         6%     980Mi           22%

```

#### 2. Pod Metrics

```bash
kubectl top pod

```

**Example Output:**

```text
NAME         CPU(cores)   MEMORY(bytes)
nginx-pod    5m           20Mi
api-server   25m          120Mi

```

---

## Summary
Monitoring is critical for cluster health. While **Metrics Server** provides the real-time data necessary for commands like `kubectl top` and for the **Horizontal Pod Autoscaler**, it lacks historical depth. For long-term trends and advanced dashboarding, administrators typically pair the Metrics Server with a tool like **Prometheus**.
