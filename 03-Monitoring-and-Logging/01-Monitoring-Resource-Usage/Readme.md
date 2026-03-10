# Monitoring Resource Usage in Kubernetes

Monitoring is an important part of managing a Kubernetes cluster. It helps administrators understand how cluster resources are being used and whether the system is performing as expected.

When monitoring Kubernetes, we typically focus on two main levels:

1. **Node-level metrics**
2. **Pod-level metrics**

---

# Node-Level Metrics

Node-level monitoring provides information about the overall health and performance of the cluster nodes.

Some common metrics include:

- Number of nodes in the cluster
- Node health status
- CPU utilization
- Memory usage
- Network usage
- Disk usage

These metrics help determine whether nodes have enough resources to run workloads efficiently.

---

# Pod-Level Metrics

Pod-level monitoring focuses on the applications running inside the cluster.

Important metrics include:

- Total number of Pods running
- CPU usage of each Pod
- Memory usage of each Pod

This information helps identify resource-heavy workloads and troubleshoot performance issues.

---

# Monitoring Solutions in Kubernetes

Kubernetes does not include a complete built-in monitoring system by default. Instead, it supports several external monitoring tools.

Some commonly used monitoring solutions include:

### Metrics Server
A lightweight monitoring solution used for basic resource metrics.

### Prometheus
A powerful open-source monitoring and alerting system commonly used with Kubernetes.

### Elastic Stack
Used for log aggregation and monitoring.

### Commercial Monitoring Tools
Some organizations use proprietary monitoring platforms such as:

- Datadog
- Dynatrace

Earlier Kubernetes setups used a monitoring tool called **Heapster**, but it has been deprecated and replaced by **Metrics Server**.

---

# Metrics Server

The **Metrics Server** is a lightweight monitoring component designed specifically for Kubernetes.

Key characteristics:

- Only **one Metrics Server runs per cluster**
- Collects metrics from nodes and Pods
- Aggregates metrics and stores them **in memory**
- Does **not store historical data**

Because it stores metrics only in memory, it is mainly used for **real-time monitoring**, not long-term analytics.

For historical monitoring and advanced analysis, tools like **Prometheus** are typically used.

---

# How Metrics Are Collected

Metrics are collected from each node using the **Kubelet**.

The Kubelet is responsible for:

- Managing Pods on a node
- Communicating with the API Server
- Reporting node information

Inside the Kubelet is a component called **cAdvisor (Container Advisor)**.

cAdvisor performs the following tasks:

- Collects resource usage statistics from containers
- Tracks CPU, memory, and network usage
- Exposes these metrics through the Kubelet API

The **Metrics Server** retrieves this data from the Kubelet and aggregates it for cluster monitoring.

---

# Installing Metrics Server

If you are using **Minikube**, you can enable the Metrics Server using:

```bash
minikube addons enable metrics-server
