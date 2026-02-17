# Kube-Scheduler

## Introduction

Kube-Scheduler is a control plane component in Kubernetes.

Its main responsibility is to decide which Pod runs on which Node.

Important:

- It does NOT create Pods.
- It does NOT delete Pods.
- Pod creation is handled by the kubelet.
- The Scheduler only assigns a Node to a Pod.

---

## What Does the Scheduler Do?

When a Pod is created:

1. The Pod is stored in etcd via the Kube-API Server.
2. At this stage, the Pod has no Node assigned.
3. The Scheduler continuously watches for such unassigned Pods.
4. It selects the most suitable Node.
5. It informs the Kube-API Server about the decision.
6. The API Server updates etcd.
7. The kubelet on the selected Node creates the Pod.

---

## Why is Scheduler Required?

Scheduler ensures that Pods are placed on the most appropriate Node based on:

- CPU requirements
- Memory requirements
- Resource availability
- Constraints and policies

### Example Scenario

Suppose we want to deploy a Pod that requires:

- 10 CPU cores

And we have 4 Nodes with the following CPU capacity:

- Node 1 → 2 CPU
- Node 2 → 4 CPU
- Node 3 → 12 CPU
- Node 4 → 18 CPU

The Scheduler evaluates:

- Resource availability
- Scheduling policies
- Priority functions

Then it decides the best suitable Node (for example, Node 3 or Node 4 depending on scoring and resource usage).

---

## Scheduling Process

The Scheduler follows two main steps:

### 1. Filtering Phase

Removes Nodes that do not meet the Pod’s requirements.

Example:
- Nodes with insufficient CPU are filtered out.

### 2. Scoring Phase (Priority Function)

Among the remaining Nodes, the Scheduler:

- Assigns scores
- Selects the Node with the highest score

---

## Custom Scheduler

Kubernetes allows us to:

- Write our own custom Scheduler
- Run multiple Schedulers in a cluster

This is useful for advanced scheduling requirements.

---

## Installing Kube-Scheduler (Kubernetes the Hard Way)

If setting up Kubernetes manually:

- Download the kube-scheduler binary from the official Kubernetes release page.
- Configure it with required parameters.
- Run it as a system service.

---

## How to View Kube-Scheduler Configuration

The method depends on how the cluster was set up.

---

### 1. If Cluster is Set Up Using kubeadm

The Scheduler runs as a static Pod.

You can inspect it using:

```bash
kubectl get pods -n kube-system
```


To view detailed configuration:
```bash
kubectl describe pod kube-scheduler-<node-name> -n kube-system
```

You can also check the manifest file:
```bash
/etc/kubernetes/manifests/kube-scheduler.yaml
```

### 2. If Cluster is Set Up Manually (Without kubeadm)

If not using kubeadm, you can inspect the service file:
```bash
/etc/systemd/system/kube-scheduler.service
```

This file contains all startup parameters.

## Quick Summary:

1. Kube-Scheduler decides which Pod runs on which Node.
2. It does NOT create or delete Pods.
3. It selects Nodes based on resource requirements and policies.
4. It uses filtering and scoring mechanisms.
5. It can be customized.
6. In kubeadm setup, it runs as a static Pod.
7. In manual setup, it runs as a system service.
