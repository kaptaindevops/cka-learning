# Controllers in Kubernetes – Replication Controller & ReplicaSet

Controllers are the brain behind Kubernetes.

They continuously monitor the state of Kubernetes objects and take action to ensure that the actual state matches the desired state.

In this document, we will focus on:

- Replication Controller (RC)
- ReplicaSet (RS)

---

# Why Do We Need Replication?

Imagine you have a single Pod running your application.

If that Pod crashes:

- Your application becomes unavailable.
- Users lose access.

To avoid this, we run multiple instances of the same Pod.

This ensures:

- High availability
- Load distribution
- Automatic recovery

This is where Replication Controllers and ReplicaSets help.

---

# Replication Controller (RC)

## What It Does

A Replication Controller ensures that a specified number of Pods are always running.

For example:

- If replicas = 3 → It ensures 3 Pods are running at all times.
- If one Pod crashes → It automatically creates a new one.
- Even if replicas = 1 → It ensures one Pod is always running.

So replication is useful even for a single Pod.

---

## Scaling with Replication Controller

When user traffic increases:

- Increase the number of replicas.
- New Pods are created.
- Load is distributed across Pods.

If the current node has no capacity:

- New Pods can run on other nodes.
- The replication controller spans across the cluster.

---

## Structure of Replication Controller YAML

A typical RC definition includes:

- apiVersion: v1
- kind: ReplicationController
- metadata
- spec

Inside `spec`, we define:

- replicas → Number of Pods required
- template → Pod definition used to create replicas

The template section contains:

- Pod metadata
- Pod spec
- Container definition

So essentially:

Replication Controller (Parent)  
→ Pod Template (Child)

---

## Creating a Replication Controller

```bash
kubectl create -f rc-definition.yaml
```

To view:

```bash
kubectl get replicationcontroller
```

To view Pods created by RC:

```bash
kubectl get pods
```

Pods created by RC usually start with the RC name.

---

# ReplicaSet (RS)

ReplicaSet is the newer and recommended version of Replication Controller.

It serves the same purpose:

- Maintain desired number of Pods
- Ensure high availability
- Handle scaling

---

## Key Differences from Replication Controller

### 1. API Version

- Replication Controller → `v1`
- ReplicaSet → `apps/v1`

If you use the wrong API version, Kubernetes will throw an error.

---

### 2. Selector (Major Difference)

ReplicaSet requires a **selector** field.

Replication Controller does not strictly require it.

The selector defines:

- Which Pods the ReplicaSet should monitor.

Example:

```yaml
selector:
  matchLabels:
    app: myapp
```

ReplicaSet matches Pods based on labels.

---

# Why Are Labels and Selectors Important?

In a cluster, there may be hundreds of Pods.

ReplicaSet must know:

- Which Pods belong to it
- Which Pods it should monitor

Labels are added to Pods:

```yaml
labels:
  app: frontend
```

ReplicaSet selector matches those labels.

This way:

- It monitors only specific Pods.
- If one fails → It creates a replacement.

---

# Template in ReplicaSet – Is It Required?

Yes.

Even if Pods already exist and match the selector:

- ReplicaSet will not create new Pods immediately.
- But if a Pod fails in the future,
- It uses the template to create a new Pod.

So template is always required.

---

# Scaling a ReplicaSet

Assume replicas = 3 and you want 6.

There are multiple ways:

---

## Method 1: Update YAML File

1. Change replicas to 6 in the file.
2. Run:

```bash
kubectl replace -f rs-definition.yaml
```

---

## Method 2: Use Scale Command

```bash
kubectl scale rs myapp-rs --replicas=6
```

Important:

If you scale using the command line,
The YAML file will still show old replica count.

---

# Common Commands

Create object:

```bash
kubectl create -f file.yaml
```

List ReplicaSets:

```bash
kubectl get replicaset
```

List Pods:

```bash
kubectl get pods
```

Delete ReplicaSet:

```bash
kubectl delete replicaset <name>
```

Replace/Update:

```bash
kubectl replace -f file.yaml
```

Scale:

```bash
kubectl scale rs <name> --replicas=<number>
```

---

# Summary

- Controllers ensure desired state matches actual state.
- Replication Controller maintains a fixed number of Pods.
- ReplicaSet is the modern replacement for Replication Controller.
- ReplicaSet requires a selector.
- Labels and selectors are critical for Pod management.
- Scaling can be done by editing YAML or using kubectl scale.
- ReplicaSet ensures high availability and automatic recovery.

In real-world Kubernetes environments, ReplicaSets are commonly used through Deployments rather than directly.
